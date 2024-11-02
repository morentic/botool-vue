<script lang="ts">
import './assets/main.css';
import data from './data/builder-order.json';
import BuildOrder from './components/BuildOrder.vue';

export default {
  setup() {
    return {
      buildOrder: data.buildOrder,
    };
  },
  components: {
    BuildOrder,
  },
};
</script>

<template>
  <BuildOrder :initial-build-order="buildOrder" />
</template>
