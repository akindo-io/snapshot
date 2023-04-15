<script>
import {
  isAddress,
  isBoolean,
  isByte,
  isInt,
  isStringArray,
  isUint
} from '../../utils/validator';

const getPlaceholder = (name, type) => {
  if (isAddress(type)) {
    return 'E.g.: ["0xACa9...DA6E","0x1dF6...006e"]';
  }

  if (isBoolean(type)) {
    return 'E.g.: [true, false, false, true]';
  }

  if (isUint(type)) {
    return 'E.g.: [1000, 212, 320000022, 23]';
  }

  if (isInt(type)) {
    return 'E.g.: [1000, -212, 1232, -1]';
  }

  if (isByte(type)) {
    return 'E.g.: ["0xc00000000000000000000000000000000000", "0xc00000000000000000000000000000000001"]';
  }

  return 'E.g.: ["first value", "second value", "third value"]';
};

const getInjectableVotes = (type) => {
  return isUint(type);
}

function getLabel(parameter) {
  let type = parameter.type;
  if (parameter.baseType === 'tuple') {
    const components = parameter.components.map(param => param.type).join(', ');
    type = `${type}(${components})`;
  }
  if (parameter.name) return `${parameter.name} (${type})`;
  return type;
}

export default {
  props: ['parameter', 'disabled', 'modelValue', 'modelInjectVote'],
  emits: ['update:modelValue', 'update:modelInjectVote'],
  data() {
    const label = getLabel(this.parameter);
    const placeholder = getPlaceholder(
      this.parameter.name,
      this.parameter.type
    );
    const injectableVotes = getInjectableVotes(this.parameter.type);
    return {
      input: '',
      dirty: false,
      placeholder,
      label,
      injectableVotes,
      injectVote: false
    };
  },
  mounted() {
    if (this.modelValue) this.input = this.modelValue;
    if(this.modelInjectVote) this.injectVote = this.modelInjectVote;
  },
  computed: {
    isValid() {
      return isStringArray(this.input);
    }
  },
  methods: {
    handleInput(value) {
      this.input = value;
      this.$emit('update:modelValue', this.input);
    },
    handleInjectVote(value) {
      this.injectVote = value;
      this.$emit('update:modelInjectVote', this.injectVote);
      if (this.injectVote) {
        this.$emit('update:modelValue', '[]');
      }
    },
  }
};
</script>

<template>
  <UiInput
    :disabled="disabled"
    :error="dirty && !isValid && `Invalid ${type}`"
    :model-value="input"
    :placeholder="placeholder"
    @update:modelValue="handleInput($event)"
  >
  </UiInput>
  <InputSwitch
      :is-disabled="disabled"
      v-model="injectVote"
      v-if="injectableVotes"
      :text-right="'Inject voting results'"
      @update:modelValue="handleInjectVote($event)"
  />
</template>
