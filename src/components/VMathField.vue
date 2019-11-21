<template>
<v-text-field
  v-bind="attributes"
  v-model="current"
  @keydown="keydown"
  @focus="focus"
  @blur="blur"
  @click:clear="clear"
  :error="error"
  :error-messages="message"
  :class="textColor"
  :success="!error && mode === 'display'"
  ref="vtf"
  >
  <slot v-for="(_, name) in $slots" :name="name" :slot="name" />
  <template v-for="(_, name) in $scopedSlots" :slot="name" slot-scope="slotData">
    <slot :name="name" v-bind="slotData" />
  </template>
</v-text-field>
</template>

<script>
const math = require('mathjs');

math.createUnit('voxel', {
  definition: '7.5 cm', aliases: [ 'voxel', 'voxels' ]
});

export default {
  name: 'v-math-field',
  props: {
    displayPrecision: {
      type: Number,
      default: 2
    },
    enter: {
      type: String,
      default: 'render'
    },
    notation: {
      type: String,
      default: 'fixed'
    },
    numeric: {
      type: Boolean,
      default: true
    },
    precision: {
      type: Number,
      default: 4
    },
    rules: {
      type: Array,
      default: () => { return []; }
    },
    type: {
      type: String,
      default: 'number'
    },
    units: {
      type: String,
      default: 'meters'
    },
    uuid: { type: String },
    value: {
      type: [ Number, String ],
      default: 0
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
    if (this.uuid && localStorage.getItem(this.uuid)) {
      this.raw = localStorage.getItem(this.uuid);
    } else {
      this.raw = this.value.toString();
    }

    this.attributes = {};
    for (const attr in this.$attrs) {
      if (attr !== 'type' && attr !== 'rules') {
        this.attributes[attr] = this.$attrs[attr];
      }
    }

    this.evaluate(true);
    this.current = this.pretty;
    console.log(this.$attrs);
  },
  computed: { textColor () {
    return this.mode === 'display' ? 'blue--text' : 'black--text';
  } },
  methods: {
    round (number) {
      const factor = Math.pow(10, this.precision);
      return Math.round(number * factor) / factor;
    },
    save (value) {
      this.raw = value;
      if (this.uuid) {
        localStorage.setItem(this.uuid, this.raw.toString());
      }
      return this.raw;
    },
    focus (...args) {
      this.mode = 'edit';
      this.current = this.raw;
      console.log('focused');
      this.$emit('focus', ...args);
    },
    blur (...args) {
      if (this.error === false) {
        if (this.mode === 'edit') {
          this.save(this.current);

          this.evaluate();
          if (!this.error) {
            this.mode = 'display';
            this.current = this.pretty;
            console.log('blurred');
          }
        }
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
      } else if (event.keyCode === 13) {
        if (this.mode === 'display') {
          event.preventDefault();
          event.stopPropagation();
        } else {
          this.save(this.current);
          this.evaluate();

          if (this.enter === 'blur' && !this.error) {
            this.$refs.vtf.blur();
          } else if (this.enter === 'render' && !this.error) {
            this.mode = 'display';
            this.update();

            event.preventDefault();
            event.stopPropagation();
          } else {
            this.update();
          }
        }
      } else if (event.keyCode === 27) {
        console.log('escape');
        if (this.mode === 'edit') {
          this.current = this.raw;
        } else {
          this.current = this.pretty;
        }
      }
    },
    clear () {
      this.current = '';
      this.save(this.current);
      this.pretty = '';
      if (this.numeric || this.type === 'number') {
        this.$emit('input', 0);
      } else {
        this.$emit('input', '');
      }
    },
    evaluate (force = false) {
      let value = this.raw;

      if (value.length) {
        if (value === this.pretty && !force) {
          return;
        }

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
    update (recompute = false) {
      if (recompute) {
        this.evaluate(true);
      }

      if (this.mode === 'display') {
        this.current = this.pretty;
      } else {
        this.current = this.raw;
      }
    }
  },
  watch: {
    precision () {
      this.update(true);
    },
    displayPrecision () {
      this.update(true);
    },
    numeric () {
      this.update(true);
    },
    units () {
      this.update(true);
    }
  }
};
</script>
