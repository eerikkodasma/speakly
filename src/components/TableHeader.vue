<template>
  <th
    class="px-4 py-2 text-left text-sm font-semibold text-gray-700"
    :class="[
      hasSorting && 'cursor-pointer',
      header === TableHeaderTitle.RANK && 'w-[100px]',
    ]"
    :role="hasSorting ? 'button' : undefined"
    :tabindex="hasSorting ? 0 : undefined"
    @click="handleSort"
    @keyup.enter="handleSort"
  >
    <span class="inline-flex items-center">
      {{ header }}
      <template v-if="hasSorting">
        <span v-if="sortAsc != null" class="ml-1">
          {{ sortAsc ? "▲" : "▼" }}
        </span>
        <span v-else class="ml-1 text-gray-400">⇅</span>
      </template>
    </span>
  </th>
</template>

<script setup lang="ts">
import { PropType } from "vue";
import { TableHeaderTitle } from "../types/Table";

const props = defineProps({
  header: {
    type: String,
    required: true,
  },
  sortAsc: {
    type: [Boolean, null] as PropType<boolean | null>,
    default: null,
  },
  hasSorting: {
    type: Boolean,
    default: false,
  },
});

function handleSort() {
  if (props.hasSorting) {
    emits("asc", props.header);
  }
}

const emits = defineEmits(["asc"]);
</script>

<style scoped></style>
