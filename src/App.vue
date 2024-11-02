<script>
import './assets/main.css';
import data from './data/builder-order.json';
import { nextTick } from 'vue';

export default {
  data() {
    return {
      buildOrder: data.buildOrder,
      running: false,
      time: 1,
    };
  },
  methods: {
    startStop() {
      clearInterval(this.interval);
      this.running = !this.running;

      if (!this.running) {
        return;
      }

      this.interval = setInterval(() => {
        this.buildOrder.map(async (item, index) => {
          if (this.time === item.time) {
            const tmp = this.buildOrder;
            if (index !== 0) {
              tmp[index - 1].active = false;
            }
            tmp[index].active = true;
            this.buildOrder = tmp;
            await nextTick();

            if (index !== 0) {
              this.$refs.buildStepRefs[index - 1].scrollIntoView();
            } else {
              this.$refs.buildStepRefs[index].scrollIntoView();
            }
          }

          const lastItem = this.buildOrder.slice(-1)[0];
          if (this.time === lastItem.time) {
            clearInterval(this.interval);
            this.running = false;
            this.time = 1;
            return;
          }
        });

        this.time = this.time + 1;
      }, 1000);
    },
    reset() {
      clearInterval(this.interval);
      this.time = 1;
      window.scrollTo(0, 0);
      this.buildOrder.map((item, index) => {
        item.active && (this.buildOrder[index].active = false);
      });
    },
  },
};
</script>

<template>
  <div class="max-w-md mx-auto">
    <div
      v-for="(buildStep, index) in buildOrder"
      :key="index"
      class="grid grid-cols-2 items-center m-2 p-3 bg-purple-500 font-bold"
      :class="{ 'border-2 border-white text-4xl py-12': buildStep.active }"
      ref="buildStepRefs"
    >
      <div>{{ buildStep.at }}</div>
      <ul>
        <li v-for="(instruction, index) in buildStep.instructions" :key="index">
          {{ instruction }}
        </li>
      </ul>
    </div>
    <div class="sticky bottom-0 flex bg-white m-2">
      <button
        class="m-1 p-3 bg-purple-500 w-full h-24 border-2 border-white"
        @click="startStop"
      >
        {{ running ? 'Stop' : 'Start' }}
      </button>
      <button
        class="m-1 p-3 bg-purple-500 w-full h-24 border-2 border-white"
        @click="reset"
      >
        Reset
      </button>
    </div>
  </div>
</template>
