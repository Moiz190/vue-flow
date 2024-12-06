<template>
  <BaseEdge :id="id" :path="path[0]" />
  <EdgeLabelRenderer v-if="edgeType !=='router'">
    <div
      :style="{
        pointerEvents: 'all',
        position: 'absolute',
        transform: `translate(-50%, -50%) translate(${path[1]}px,${path[2]}px)`,
      }"
      class="nodrag nopan"
    >
      <button class="edgebutton" @click="handleRemoveEdge(id)">×</button>
    </div>
  </EdgeLabelRenderer>
</template>
<script setup lang="ts">
import {
  BaseEdge,
  EdgeLabelRenderer,
  getSmoothStepPath,
  Position,
} from "@vue-flow/core";
import { computed } from "vue";
export interface IProps {
  id: string;
  edgeType?: "router" | "node";
  sourceX: number;
  sourceY: number;
  targetX: number;
  targetY: number;
  sourcePosition: Position;
  targetPosition: Position;
}

const props = defineProps<IProps>();
const emit = defineEmits<{ (event: "remove-edge", id: string): void }>();
const path = computed(() => getSmoothStepPath(props));
const handleRemoveEdge = (selectedEdgeId: string) => {
  emit("remove-edge", selectedEdgeId);
};
</script>
<style scoped>
.edgebutton {
  width: 1.3rem;
  border-radius: 50%;
  cursor: pointer;
}
.edgebutton:hover {
  transform: scale(1.1);
  transition: all ease 0.5s;
  box-shadow: 0 0 0 2px #10b98180, 0 0 0 4px #10b981;
}
</style>
