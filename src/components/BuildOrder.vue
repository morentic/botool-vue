<script lang="ts">
import data from './../data/builder-order.json';
import { nextTick, ref, toRefs } from 'vue';

export default {
  setup(props) {
    const buildStepRefs = ref<Array<HTMLDivElement>>([]);
    const { initialBuildOrder } = toRefs(props);

    const buildOrder = ref(initialBuildOrder);

    return { buildStepRefs, buildOrder };
  },
  props: {
    initialBuildOrder: Array<{
      time: number;
      at: string | number;
      instructions: Array<string>;
      active: Boolean;
    }>,
  },
  data() {
    return {
      running: false,
      time: 1,
      interval: 0,
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
        if (!this.buildOrder) {
          return;
        }

        this.buildOrder.map(async (item, index) => {
          if (this.time === item.time) {
            const tmp = this.buildOrder;
            if (!tmp) {
              return;
            }
            if (index !== 0) {
              tmp[index - 1].active = false;
            }

            tmp[index].active = true;
            this.buildOrder = tmp;
            await nextTick();

            if (index !== 0) {
              this.buildStepRefs[index - 1].scrollIntoView();
            } else {
              this.buildStepRefs[index].scrollIntoView();
            }
          }
        });

        const lastItem = this.buildOrder.slice(-1)[0];
        if (this.time === lastItem.time) {
          clearInterval(this.interval);
          this.running = false;
          this.time = 1;
          return;
        }

        this.time = this.time + 1;
      }, 1000);
    },
    reset() {
      clearInterval(this.interval);
      this.time = 1;
      window.scrollTo(0, 0);
      if (!this.buildOrder) {
        return;
      }
      this.buildOrder.map((item, index) => {
        if (!this.buildOrder) {
          return;
        }
        item.active && (this.buildOrder[index].active = false);
      });
    },
  },
};
</script>

<template>
  <div class="max-w-md mx-auto">
    <div
      class="p-3 absolute top-2 right-2 h-12 bg-sky-950 border-white border-2"
    >
      {{ time }}
    </div>
    <div
      v-for="(buildStep, index) in buildOrder"
      :key="index"
      class="grid grid-cols-2 items-center m-2 p-3 bg-purple-500 font-bold"
      :class="{ 'border-2 border-white text-4xl py-12': buildStep.active }"
      ref="buildStepRefs"
    >
      <div>{{ buildStep.at }}</div>
      <ul class="list-disc">
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
