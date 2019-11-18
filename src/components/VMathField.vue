<template>
<v-text-field
  v-bind="$attrs"
  v-model="current"
  @change="evaluate"
  @keydown="keydown"
  @keypress.enter="pressedEnter"
  @focus="focus"
  @blur="blur"
  :error="error"
  :error-messages="message"
  :class="textColor"
  :success="!error && mode === 'display'"
  ref="vtf"
  />
</template>

<script>
const math = require('mathjs');

math.createUnit('voxel', {
  definition: '7.5 cm', aliases: [ 'voxel', 'voxels' ]
});

export default {
  name: 'v-math-field',
  props: {
    value: {
      type: [ Number, String ],
      default: 0
    },
    units: {
      type: String,
      default: 'meters'
    },
    displayPrecision: {
      type: Number,
      default: 2
    },
    precision: {
      type: Number,
      default: 4
    },
    notation: {
      type: String,
      default: 'fixed'
    },
    numeric: {
      type: Boolean,
      default: true
    },
    enter: {
      type: String,
      default: 'render'
    }
  },
  data: () => {
    return {
      current: '',
      mode: 'display',
      raw: '',
      pretty: '',
      error: false,
      message: ''
    };
  },
  created () {
    this.mode = 'display';
    this.evaluate(this.value.toString(), true);
    this.current = this.pretty;
  },
  computed: { textColor () {
    return this.mode === 'display' ? 'blue--text' : 'black--text';
  } },
  methods: {
    round (number) {
      const factor = Math.pow(10, this.precision);
      return Math.round(number * factor) / factor;
    },
    focus (...args) {
      this.mode = 'edit';
      this.current = this.raw;
      console.log('focused');
      this.$emit('focus', ...args);
    },
    blur (...args) {
      if (this.error === false) {
        this.mode = 'display';
        this.current = this.pretty;
        console.log('blurred');
        this.$emit('blur', ...args);
      }
    },
    keydown (event) {
      if (this.mode === 'display' && this.enter === 'render' &&
          (event.keyCode !== 9 && event.keyCode !== 13)) {
        event.preventDefault();
        event.stopPropagation();
        this.mode = 'edit';
        this.update();
      }
    },
    pressedEnter () {
      console.log('enter pressed');
      if (this.enter === 'blur') {
        if (!this.error) {
          this.raw = this.current;
          this.$refs.vtf.blur();
        }
      } else if (this.enter === 'render' && this.mode !== 'display') {
        if (!this.error) {
          this.raw = this.current;
          this.mode = 'display';
          this.update();

          event.preventDefault();
          event.stopPropagation();
        }
      } else if (this.mode === 'display') {
        event.preventDefault();
        event.stopPropagation();
      }
    },
    evaluate (value, force = false) {
      if (value.length) {
        if (value === this.pretty && !force) {
          return;
        }

        this.raw = value;

        try {
          console.log(`evaluating "${ value }"`);

          value = math.evaluate(value);
          if (typeof value === 'number') {
            value = math.unit(value, this.units);
          } else {
            value = value.to(this.units);
          }

          console.log(typeof value, value);

          this.pretty = math.format(value, {
            notation: this.notation,
            precision: this.displayPrecision
          });

          this.error = false;
          this.message = '';

          if (this.numeric) {
            const number = this.round(value.toNumber(this.units));
            this.$emit('input', number);
          } else {
            this.$emit('input', this.pretty);
          }
        } catch (error) {
          console.log('errored', error);
          this.error = true;
          this.message = error.message;
        }
      }
    },
    update () {
      if (this.raw) {
        this.evaluate(this.raw);
        if (this.mode === 'display') {
          this.current = this.pretty;
        } else {
          this.current = this.raw;
        }
      }
    }
  },
  watch: {
    precision () {
      this.update();
    },
    displayPrecision () {
      this.update();
    },
    numeric () {
      this.update();
    }
  }
};
</script>
