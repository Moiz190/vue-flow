<template>
  <div class="custom-node relative" :id="nodeInfo?.id">
    <div class="h-full add-new-node-container p-1">
      <div
        v-if="Object.keys(nodeInfo?.data!).length === 0"
        class="h-full add-new-btn font-bold flex flex-col items-center justify-center"
      >
        <div class="add-new-icon"><span>+</span></div>
        <div>
          <span>Add {{ nodeInfo?.type }} {{ nodeInfo?.id }}</span>
        </div>
      </div>
      <div v-else>
        <div
          v-if="type === 'node'"
          class="h-full flex flex-col items-center justify-between"
        >
          <span class="nodeHeader text-ellipsis font-bold">{{
            nodeInfo?.data.header
          }}</span>
          <span class="nodeBody text-ellipsis font-bold">{{
            nodeInfo?.data.body
          }}</span>
          <span class="nodeTitle text-ellipsis font-bold">{{
            nodeInfo?.data.title
          }}</span>
        </div>
        <div v-else>
          {{ nodeInfo?.type }}
        </div>
      </div>
      <div
        class="remove-node-container absolute"
        @click="handleRemoveNode(nodeInfo?.id!, nodeInfo?.type!)"
      >
        x
      </div>
      <div
        v-if="nodeInfo?.type !== 'router'"
        class="add-right-node add-node-container flex items-center justify-center absolute"
        @click="handleAddNode(nodeInfo?.id!)"
      >
        +
      </div>
    </div>
  </div>
  <Handle :position="Position.Right" type="source" />
  <Handle :position="Position.Bottom" type="source" />
  <Handle type="source" :position="Position.Top" />
  <Handle :position="Position.Bottom" />
  <Handle v-if="nodeInfo?.id !== '0'" type="target" :position="Position.Left" />
</template>
<script setup lang="ts">
import { NodeProps, Position, Handle } from "@vue-flow/core";
import { CustomData, CustomEvents, CustomNodeTypes } from "./FlowDiagram.vue";
const emit = defineEmits<{
  (event: "add-node", nodeId: string): void;
  (event: "remove-node", nodeId: string): void;
  (event: "remove-router", nodeId: string): void;
}>();
interface ICustomNodeProps {
  type: "node" | "router";
  nodeInfo: NodeProps<CustomData, CustomEvents, CustomNodeTypes> | null;
}
withDefaults(defineProps<ICustomNodeProps>(), {
  type: "node",
  nodeInfo: null,
});
const handleAddNode = (selectedNodeId: string) => {
  emit("add-node", selectedNodeId);
};
const handleRemoveNode = (nodeId: string, nodeType: string) => {
  if (nodeType === "custom") {
    emit("remove-node", nodeId);
  } else {
    emit("remove-router", nodeId);
  }
};
</script>

<style scoped>
.custom-node {
  width: 6.5rem;
  height: 6.5rem;
  background-color: #fff;
  border-radius: 10px;
  box-shadow: 0px 1px 7px 1px rgba(0, 0, 0, 0.4);
}
.add-new-node-container .add-new-icon {
  font-size: 1.3rem;
}
.remove-node-container {
  transition: all 0.5s;
  width: 1.2rem;
  height: 2rem;
  top: 0.7rem;
  right: 1.1rem;
  z-index: -1;
  background-color: #bbbbbb;
  color: #fff;
  padding: 0.3rem;
  border-radius: 0 6px 0;
  box-shadow: 0px 1px 7px 1px rgba(0, 0, 0, 0.4);
}
.add-node-container {
  transition: all 0.5s;
  width: 1.3rem;
  bottom: 0.7rem;
  height: 3.5rem;
  z-index: -1;
  color: #fff;
  box-shadow: 0px 1px 7px 1px rgba(0, 0, 0, 0.4);
  background: linear-gradient(to bottom right, #5721d7, #ba0af3);
  padding: 0.3rem;
}
.add-right-node {
  border-radius: 0 14px 14px 0;
  right: 1.6rem;
}
.add-new-btn:hover,
.remove-node-container:hover,
.add-node-container:hover {
  cursor: pointer;
}
.add-new-node-container:hover > .remove-node-container {
  right: -1.1rem;
}
.add-new-node-container:hover > .add-right-node {
  right: -1.3rem;
}
.nodeTitle,
.nodeBody,
.nodeHeader {
  font-size: 0.7rem;
}
</style>
