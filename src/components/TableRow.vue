<template>
  <tr>
    <td class="px-4 py-2 text-sm text-gray-800">{{ index }}.</td>
    <td class="px-4 py-2 text-sm text-gray-800">
      {{ rank }}{{ suffix }}
      <span
        v-if="growth != 0"
        class="inline-flex items-center"
        :class="growth > 0 ? 'text-green-500' : 'text-red-500'"
      >
        {{ growth }}
        <span class="ml-1">
          {{ growth > 0 ? "▲" : "▼" }}
        </span>
      </span>
    </td>
    <td class="px-4 py-2 text-sm text-gray-800">{{ userName }}</td>
    <td class="px-4 py-2 text-sm text-gray-800">
      {{ score }}
      <span
        v-if="pointsGained != 0"
        class="inline-flex items-center"
        :class="pointsGained > 0 ? 'text-green-500' : 'text-red-500'"
      >
        <span v-if="pointsGained > 0">+</span>
        {{ pointsGained }}
      </span>
    </td>
    <td class="px-4 py-2 flex gap-2">
      <button
        class="bg-blue-500 text-white px-2 rounded hover:bg-blue-600"
        @click="$emit('changeScore', { id, points: 1 })"
        aria-label="Increase score"
      >
        <div class="w-3">+</div>
      </button>
      <button
        class="bg-red-500 text-white px-2 rounded hover:bg-red-600"
        @click="$emit('changeScore', { id, points: -1 })"
        aria-label="Decrease score"
      >
        <div class="w-3">-</div>
      </button>
    </td>
  </tr>
</template>

<script setup lang="ts">
import { computed } from "vue";
const props = defineProps({
  id: {
    type: Number,
    required: true,
  },
  rank: {
    type: Number,
    required: true,
  },
  previousRank: {
    type: Number,
    required: true,
  },
  userName: {
    type: String,
    required: true,
  },
  score: {
    type: Number,
    required: true,
  },
  pointsGained: {
    type: Number,
    required: true,
  },
  index: {
    type: Number,
    required: true,
  },
});

const growth = computed(() => {
  return props.previousRank - props.rank;
});

const suffix = computed(() => {
  let reducedSmallNumber = props.rank % 10,
    reducedBigNumber = props.rank % 100;
  if (reducedSmallNumber === 1 && reducedBigNumber !== 11) {
    return "st";
  }
  if (reducedSmallNumber === 2 && reducedBigNumber !== 12) {
    return "nd";
  }
  if (reducedSmallNumber === 3 && reducedBigNumber !== 13) {
    return "rd";
  }
  return "th";
});

const emits = defineEmits(["changeScore"]);
</script>

<style scoped></style>
