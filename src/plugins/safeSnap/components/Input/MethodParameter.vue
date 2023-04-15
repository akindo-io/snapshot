<script>
import { isParameterValue } from '../../utils/validator';
import { isArrayParameter } from '../../index';
import SafeSnapInputAddress from './Address.vue';
import SafeSnapInputArrayType from './ArrayType.vue';

export default {
  components: { SafeSnapInputAddress, SafeSnapInputArrayType },
  props: ['modelValue', 'disabled', 'parameter', 'modelInjectVote'],
  emits: ['update:modelValue', 'isValid'],
  data() {
    const placeholder = this.parameter.name
      ? `${this.parameter.name} (${this.parameter.type})`
      : this.parameter.type;

    let value;
    if (this.parameter.type === 'bool') value = false;

    return {
      placeholder,
      value,
      dirty: false,
      injectVote: false
    };
  },
  computed: {
    isValid() {
      return isParameterValue(this.parameter.baseType, this.value);
    }
  },
  watch: {
    modelValue(value) {
      this.input = value;
    }
  },
  mounted() {
    if (this.modelValue) this.value = this.modelValue;
    if(this.modelInjectVote) this.injectVote = this.modelInjectVote;
  },
  created() {
    if (this.modelValue) this.input = this.modelValue;
    if (this.modelInjectVote) this.injectVote = this.modelInjectVote;
  },
  methods: {
    handleInput(value) {
      this.value = value;
      this.dirty = true;
      this.$emit('update:modelValue', value);
      this.$emit('isValid', this.isValid);
    },
    handleInvectVote(value) {
      this.value = value;
      this.$emit('update:modelInjectVote', value);
      this.$emit('isValid', this.isValid);
    },
    isArrayType() {
      return isArrayParameter(this.parameter.baseType);
    }
  }
};
</script>

<template>
  <div class="mb-3">
  <UiSelect
    v-if="parameter.type === 'bool'"
    :disabled="disabled"
    :model-value="value"
    @update:modelValue="handleInput($event)"
  >
    <template #label>{{ placeholder }}</template>
    <option :value="true">true</option>
    <option :value="false">false</option>
  </UiSelect>

  <!-- ADDRESS -->
  <SafeSnapInputAddress
    v-else-if="parameter.type === 'address'"
    :disabled="disabled"
    :input-props="{ required: true }"
    :label="placeholder"
    :model-value="value"
    @update:modelValue="handleInput($event)"
  />
  <!-- Array of X type -->
  <SafeSnapInputArrayType
    v-else-if="isArrayType()"
    :disabled="disabled"
    :model-value="value"
    :model-inject-vote="injectVote"
    :parameter="parameter"
    @update:modelValue="handleInput($event)"
    @update:modelInjectVote="handleInvectVote($event)"
  />
  <!-- Text input -->
  <UiInput
    v-else
    :disabled="disabled"
    :error="dirty && !isValid && `Invalid ${parameter.type}`"
    :model-value="value"
    @update:modelValue="handleInput($event)"
  >
    <template #label>{{ placeholder }}</template>
  </UiInput>
  </div>
</template>
