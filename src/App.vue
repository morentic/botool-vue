<script>
import './assets/main.css';
import data from './data/builder-order.json';
import { nextTick } from 'vue';

export default {
  data() {
    return {
      buildOrder: data.buildOrder,
      active: false,
      running: false,
      time: 1,
    };
  },
  created() {
    scrollTo(0, 0);
  },
  methods: {
    startStop() {
      clearInterval(this.interval);
      this.running = !this.running;

      if (!this.running) {
        return;
      }

      this.interval = setInterval(() => {
        console.log(this.time);

        this.buildOrder.map(async (item, index) => {
          if (this.time === item.time) {
            const tmp = this.buildOrder;
            if (index !== 0) {
              tmp[index - 1].active = false;
            }
            tmp[index].active = true;
            this.buildOrder = tmp;
            await nextTick();
            this.$refs.buildStepRefs[index].scrollIntoView();
          }

          const lastItem = this.buildOrder.slice(-1)[0];
          if (this.time === lastItem.time) {
            this.running = false;
            this.time = 1;
            return;
          }
        });

        this.time = this.time + 1;
      }, 1000);
    },
  },
};
</script>

<template>
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
  <button
    class="sticky bottom-0 p-3 bg-purple-500 w-full h-24 border-2 border-white"
    @click="startStop"
  >
    {{ running ? 'Stop' : 'Start' }}
  </button>
</template>
