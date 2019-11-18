<template>
<v-app>
  <v-content>
    <v-container fluid>
      <v-row>
        <v-col
          cols="12"
          sm="6"
          >
          <v-text-field
            v-model="displayPrecision"
            label="Display Precision (decimal places)"
            ></v-text-field>
          <v-text-field
            v-model="precision"
            label="Return Value Precision (decimal places)"
            ></v-text-field>
          <v-switch
            v-model="numeric"
            class="ma-1"
            label="Numeric Return Value"
            ></v-switch>
          <v-select
            v-model="enter"
            :items="enterOptions"
            item-text="text"
            item-value="value"
            label="Enter Key Behavior"
            ></v-select>
          <v-text-field
            v-model="label"
            label="Label"
            ></v-text-field>
          <v-text-field
            v-model="hint"
            label="Hint"
            ></v-text-field>
          <v-text-field
            v-model="placeholder"
            label="Placeholder"
            ></v-text-field>
        </v-col>
        <v-col
          cols="12"
          md="6"
          >
          <v-sheet
            elevation="12"
            class="pa-12"
            >
            <v-math-field
              v-model="model"
              :precision="Number(precision)"
              :displayPrecision="Number(displayPrecision)"
              :enter="enter"
              :numeric="numeric"
              :label="label"
              :hint="hint"
              :placeholder="placeholder"
              :shaped="shaped"
              :outlined="outlined"
              :rounded="rounded"
              :solo="solo"
              :single-line="singleLine"
              :filled="filled"
              :clearable="clearable"
              :persistent-hint="persistentHint"
              :loading="loading"
              :flat="flat"
              :counter="counterEn ? counter : false"
              :dense="dense"
              ></v-math-field>
            <div class="mt-12 text-center">
              Return Value: <span style="background-color: #ddd; padding: 4px;">{{ model }}</span>
            </div>
        </v-sheet>
        </v-col>
        <v-col
          cols="12"
          md="6"
          >
          <v-switch
            v-model="shaped"
            class="ma-1"
            label="Shaped (requires Filled, Outlined or Solo)"
            :disabled="!outlined && !filled && !solo"
            ></v-switch>
          <v-switch
            v-model="outlined"
            class="ma-1"
            label="Outlined"
            ></v-switch>
          <v-switch
            v-model="rounded"
            class="ma-1"
            label="Rounded (requires Filled, Outlined or Solo)"
            :disabled="!filled && !outlined && !solo"
            ></v-switch>
          <v-switch
            v-model="solo"
            class="ma-1"
            label="Solo"
            :disabled="filled"
            ></v-switch>
          <v-switch
            v-model="singleLine"
            class="ma-1"
            label="Single-line"
            ></v-switch>
          <v-switch
            v-model="filled"
            class="ma-1"
            label="Filled"
            :disabled="outlined || solo"
            ></v-switch>
          <v-switch
            v-model="clearable"
            class="ma-1"
            label="Clearable"
            ></v-switch>
          <v-switch
            v-model="persistentHint"
            class="ma-1"
            label="Persistent Hint"
            ></v-switch>
          <v-switch
            v-model="loading"
            class="ma-1"
            label="Loading"
            ></v-switch>
          <v-switch
            v-model="flat"
            class="ma-1"
            label="Flat (requires Solo)"
            :disabled="!solo"
            ></v-switch>
          <v-switch
            v-model="dense"
            class="ma-1"
            label="Dense"
            ></v-switch>
          <v-row>
            <v-switch
              v-model="counterEn"
              class="ma-0 mr-2 ml-1"
              label="Counter"
              ></v-switch>
            <v-slider
              v-model="counter"
              :disabled="!counterEn"
              ></v-slider>
          </v-row>
        </v-col>
    </v-row>
  </v-container>
    </v-content>
  </v-app>
</template>

<script>
import VMathField from './components/VMathField';

export default {
  name: 'App',
  components: { VMathField },
  data: () => {
    return {
      model: '(37 in + 5 m + 2 voxels * PI / 2)',
      label: 'Meters',
      hint: 'Enter meters or expression',
      enter: 'render',
      enterOptions: [ {
        text: 'Render while editable',
        value: 'render'
      }, {
        text: 'Tab away from component',
        value: 'blur'
      }, {
        text: 'Internally recompute value',
        value: 'compute'
      } ],
      placeholder: '',
      displayPrecision: 2,
      precision: 8,
      numeric: true,
      shaped: false,
      outlined: false,
      rounded: false,
      solo: false,
      singleLine: false,
      filled: false,
      clearable: false,
      persistentHint: false,
      loading: false,
      flat: false,
      counterEn: false,
      counter: 0,
      dense: false
    };
  }
};
</script>
