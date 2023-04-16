<script setup lang="ts">
import {computed, onMounted, onUnmounted, reactive, Ref, ref} from 'vue'

interface Queue {
  id: number,
  flowRate: number,
  length: number,
}

// 常量
const bufferSize = 20
const numQueues = 4
const maxFlowRate = 4
const flowRateStep = 0.01

const alpha = ref(1)

const running = ref(false)
const queueInfos = reactive<Queue[]>([])
for (let i = 0; i < numQueues; i++) {
  queueInfos.push({id: i, flowRate: 0, length: 0})
}

const totalSize = computed(() => {
  let s = 0
  for (let i of queueInfos) {
    s += i.length
  }
  return s
})

const threshold = computed(() => {
  return alpha.value * (bufferSize - totalSize.value);
})

const thresholdPosition = computed(() => {
  let n = threshold.value / bufferSize
  if (n > 1) {
    n = 1
  }
  return {'--position': n * 100 + '%'}
})

let intervalId = 0;
onMounted(() => {
  intervalId = setInterval(() => {
    if (!running.value) {
      return
    }
    for (let i = 0; i < queueInfos.length; i++) {
      let n = queueInfos[i].length
      let v = queueInfos[i].flowRate - 1
      let k = 0
      if (n <= threshold.value) {
        k = n + v / 10
      } else {
        k = n - 1 / 10
      }
      if (k < 0) {
        k = 0
      }
      queueInfos[i].length = k
    }
  }, 100)
})

onUnmounted(() => {
  clearInterval(intervalId)
})

</script>

<template>
<div class="px-8 py-4 bg-white rounded-xl shadow-lg flex flex-col items-center justify-center space-y-4">
  <p >Dynamic Threshold Simulator</p>
  <div class="space-x-4">
    <input type="range" min="1" max="6" step="0.1" v-model.number="alpha">
    <span>α = {{ alpha.toFixed(1) }}</span>
  </div>
  <div v-for="i in queueInfos" class="space-x-4">
    <progress class="with-threshold" :style="thresholdPosition" :value="bufferSize - i.length" :max="bufferSize" />
    <input type="range" min="0" :max="maxFlowRate" :step="flowRateStep" v-model.number="i.flowRate">
    <span>Rate = {{ i.flowRate.toFixed(2) }}</span>
  </div>
  <progress :value="bufferSize - totalSize" :max="bufferSize" />
  <div>
    <button class="p-2 bg-blue-300 rounded-xl shadow-lg w-20" @click="running = !running">{{ running ? 'Pause' : 'Resume' }}</button>
  </div>
</div>
</template>

<style scoped>

progress {
  position: relative;
}

.with-threshold::before {
  content: '';
  position: absolute;
  top: 0;
  bottom: 0;
  right: var(--position);
  border-left: 2px solid red;
}

progress::-webkit-progress-bar {
  background-color: greenyellow;
}

progress::-webkit-progress-value {
  background-color: grey;
}

</style>
