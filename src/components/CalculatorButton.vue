CalculatorButton Component with Options API

<script>
export default {
  name: "CalculatorButton",
  props: {
    name: {
      type: String,
    },
    handleButton: {
      type: Function,
    },
  },
  methods: {
    handleClick() {
      this.handleButton(this.name);
    },
  },
};
</script>

<template>
  <button @click="handleClick">
    <slot></slot>
  </button>
</template>

<style scoped></style>
