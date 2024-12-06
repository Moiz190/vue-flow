<template>
  <div ref="dropdownRef">
    <custom-dropdown
      :isMenuVisible="isMenuVisible"
      :top="dropdownPosition.top"
      :left="dropdownPosition.left"
      @selected-action="handleSelectedDropdownAction"
    />
  </div>
  <VueFlow
    :nodes="nodes"
    :edges="edges"
    :class="{ dark }"
    class="flow-diagram-container"
    :default-viewport="{ zoom: 1.5 }"
    :min-zoom="0.2"
    :max-zoom="1.2"
  >
    <template #edge-custom="customEdgeProps">
      <custom-edge
        edge-type="node"
        :id="customEdgeProps.id"
        :source-x="customEdgeProps.sourceX"
        :source-y="customEdgeProps.sourceY"
        :target-x="customEdgeProps.targetX"
        :target-y="customEdgeProps.targetY"
        :source-position="customEdgeProps.sourcePosition"
        :target-position="customEdgeProps.targetPosition"
        @remove-edge="handleRemoveEdge"
      />
    </template>
    <template #edge-router="routerEdgeProps">
      <custom-edge
        edge-type="router"
        :id="routerEdgeProps.id"
        :source-x="routerEdgeProps.sourceX"
        :source-y="routerEdgeProps.sourceY"
        :target-x="routerEdgeProps.targetX"
        :target-y="routerEdgeProps.targetY"
        :source-position="routerEdgeProps.sourcePosition"
        :target-position="routerEdgeProps.targetPosition"
        @remove-edge="handleRemoveEdge"
      />
    </template>
    <template #node-custom="customNodeProps">
      <custom-node
        type="node"
        :node-info="customNodeProps as NodeProps<CustomData, CustomEvents, CustomNodeTypes>"
        @remove-node="handleRemoveNode"
        @add-node="handleAddNode"
      />
    </template>
    <template #node-router="customRouteProps">
      <custom-node
        type="router"
        :node-info="customRouteProps as NodeProps<CustomData, CustomEvents, CustomNodeTypes>"
        @remove-router="handleRemoveRouter"
        @add-node="handleAddNode"
      />
    </template>
    <Background pattern-color="#aaa" :gap="16" />
  </VueFlow>
</template>
<script setup lang="ts">
import { ref, onUnmounted, onMounted } from "vue";
import {
  Edge,
  NodeProps,
  Node,
  Position,
  VueFlow,
  useVueFlow,
} from "@vue-flow/core";
import { Background } from "@vue-flow/background";
import CustomDropdown from "./CustomDropdown.vue";
import CustomNode from "./CustomNode.vue";
import CustomEdge from "./CustomEdge.vue";
import { dropdownOption } from "../interfaces";

export interface CustomData {
  header?: string;
  body?: string;
  title?: string;
}
export interface CustomEvents {
  [onCustomEvent: string]: (event: MouseEvent) => void;
}
export type CustomNodeTypes = "custom" | "router";
type CustomNode = Node<CustomData, CustomEvents, CustomNodeTypes>;
const { onEdgeUpdate, updateEdge, removeEdges } = useVueFlow();
const nodes = ref<CustomNode[]>([
  {
    id: "0",
    type: "custom",
    label: "Node 1",
    position: { x: 10, y: 10 },
  },
]);
const isMenuVisible = ref(false);
const selectedNodeId = ref<null | string>(null);
const edges = ref<Edge[]>([]);
const dark = ref(false);
const nodeCounter = ref(0);
const dropdownPosition = ref({
  top: 0,
  left: 0,
});
const dropdownRef = ref<null | HTMLElement>(null);

onEdgeUpdate(({ edge, connection }) => {
  if (edge.source === connection.source && edge.target === connection.target) {
    const selectedEdgeId = `e${edge.target}-${edge.source}`;
    const selectedEdgeIndex = edges.value.findIndex(
      (selectedEdge) => selectedEdge.id === selectedEdgeId
    );
    if (selectedEdgeIndex !== -1) {
      updateEdge(edge, connection);
      edges.value[selectedEdgeIndex].targetHandle = connection.targetHandle;
      edges.value[selectedEdgeIndex].sourceHandle = connection.sourceHandle;
    }
  } else {
    const existingSelectedEdgeId = `e${edge.target}-${edge.source}`;
    const newSelectedEdgeId = `e${connection.target}-${connection.source}`;
    const existingSelectedEdgeIdIndex = edges.value.findIndex(
      (edge) => edge.id === existingSelectedEdgeId
    );
    edges.value[existingSelectedEdgeIdIndex] = {
      ...edges.value[existingSelectedEdgeIdIndex],
      id: newSelectedEdgeId,
      source: connection.source,
      target: connection.target,
      sourceHandle: connection.sourceHandle,
      targetHandle: connection.targetHandle,
    };
    updateEdge(edge, connection);
  }
});
const handleRemoveEdge = (id: string) => {
  const selectedEdgeId = edges.value.findIndex((edge) => edge.id === id);
  edges.value.splice(selectedEdgeId, 1);
  removeEdges(id);
};
const handleClickOutside = (event: Event) => {
  if (!isClickInsideDropdown(event.target as unknown as Node)) {
    isMenuVisible.value = false;
  }
};
const isClickInsideDropdown = (target: Node) => {
  return dropdownRef.value && dropdownRef.value.contains(target.data);
};
const handleContextMenu = (event: MouseEvent) => {
  const target = event.target as HTMLElement;
  const isClickedOnNode = target.closest(".custom-node");
  const isClickedOnVueFlow = target.closest(".flow-diagram-container");
  if (isClickedOnNode) {
    const nodeId = isClickedOnNode.getAttribute("id");
    selectedNodeId.value = nodeId;
    isMenuVisible.value = false;
    event.preventDefault();
    dropdownPosition.value.top = event.clientY;
    dropdownPosition.value.left = event.clientX;
    setTimeout(() => {
      isMenuVisible.value = true;
    }, 50);
  } else if (!isClickedOnVueFlow) {
    isMenuVisible.value = false;
  } else {
    isMenuVisible.value = false;
  }
};
onMounted(() => {
  window.addEventListener("contextmenu", handleContextMenu);
  document.addEventListener("click", handleClickOutside);
});
onUnmounted(() => {
  window.removeEventListener("contextmenu", handleContextMenu);
  document.removeEventListener("click", handleClickOutside);
});
const handleAddNode = (selectedNode: string) => {
  nodeCounter.value++;
  const newNode = {
    id: nodeCounter.value.toString(),
    label: `Node ${nodeCounter.value}`,
    position: {
      x:
        nodes.value.length !== 0
          ? nodes.value[nodes.value.length - 1].position.x + 200
          : 10,
      y:
        nodes.value.length !== 0
          ? nodes.value[nodes.value.length - 1].position.y
          : 10,
    },
    class: "light",
  };
  const selectedNodeIndex = nodes.value.findIndex(
    (node) => node.id === selectedNode
  );
  if (nodes.value[selectedNodeIndex + 1]) {
    let nextNodeIndex = selectedNodeIndex + 1;
    const nextNode = nodes.value[nextNodeIndex];
    nodes.value.splice(nextNodeIndex, 0, {
      ...newNode,
      position: {
        x: nodes.value[selectedNodeIndex].position.x,
        y: nodes.value[selectedNodeIndex].position.y,
      },
      targetPosition: Position.Left,
      sourcePosition: Position.Right,
      type: "custom",
    });
    edges.value.push({
      id: `e${nodeCounter.value}-${selectedNode}`,
      source: `${String(selectedNode)}`,
      target: `${nodeCounter.value}`,
      type: "custom",
      updatable: true,
      sourceHandle: `${selectedNode}__handle-right`,
      targetHandle: `${nodeCounter.value}__handle-left`,
    });
    const edgeId = `e${nextNode.id}-${selectedNode}`;
    const selectedEdgeIdIndex = edges.value.findIndex(
      (edge) => edge.id === edgeId
    );
    if (selectedEdgeIdIndex !== -1) {
      edges.value[selectedEdgeIdIndex] = {
        ...edges.value[selectedEdgeIdIndex],
        id: `e${edges.value[selectedEdgeIdIndex].target}-${nodeCounter.value}`,
        source: `${nodeCounter.value}`,
        sourceHandle: `${nodeCounter.value}__${
          edges.value[selectedEdgeIdIndex].sourceHandle?.split("__")[1]
        }`,
      };
    }
    for (let i = nextNodeIndex; i < nodes.value.length; i++) {
      nodes.value[i].position.x += i * 2 + 200;
    }
  } else {
    if (nodes.value.length === 0) {
      nodes.value.push({
        ...newNode,
        sourcePosition: Position.Right,
        type: "custom",
      });
    } else {
      nodes.value.push({
        ...newNode,
        targetPosition: Position.Left,
        sourcePosition: Position.Right,
        type: "custom",
      });
      const newNodeIndex = nodes.value.findIndex(
        (node) => node.id === `${nodeCounter.value}`
      );
      if (newNodeIndex !== -1) {
        edges.value.push({
          id: `e${nodeCounter.value}-${nodes.value[newNodeIndex - 1].id}`,
          source: `${nodes.value[newNodeIndex - 1].id}`,
          target: `${nodeCounter.value}`,
          type: "custom",
          updatable: true,
          sourceHandle: `${nodes.value[newNodeIndex - 1].id}__handle-right`,
          targetHandle: `${nodeCounter.value}__handle-left`,
        });
      }
    }
  }
};
const handleAddRouter = (selectedNode: string) => {
  console.log(selectedNode, "router");
  const routerId = ++nodeCounter.value;
  const router = {
    label: "Router",
    id: routerId.toString(),
    position: {
      x:
        nodes.value.length !== 0
          ? nodes.value[nodes.value.length - 1].position.x + 200
          : 10,
      y:
        nodes.value.length !== 0
          ? nodes.value[nodes.value.length - 1].position.y
          : 10,
    },
    class: "light",
  };
  nodes.value.push({
    ...router,
    sourcePosition: Position.Right,
    type: "router",
  });
  const secondSmallerId = secondSmallerNumberThanCounter(
    nodes.value.map((node) => node.id),
    routerId.toString()
  );
  edges.value.push({
    id: `e${nodeCounter.value}-${secondSmallerId}`,
    source: `${String(secondSmallerId)}`,
    target: `${nodeCounter.value}`,
    type: "router",
    animated: true,
    updatable: true,
    sourceHandle: `${secondSmallerId}__handle-right`,
    targetHandle: `${nodeCounter.value}__handle-left`,
  });
  nodeCounter.value++;
  nodes.value.push({
    label: "node",
    position: {
      x:
        nodes.value.length !== 0
          ? nodes.value[nodes.value.length - 1].position.x + 200
          : 10,
      y:
        nodes.value.length !== 0
          ? nodes.value[nodes.value.length - 1].position.y
          : 10,
    },
    class: "light",
    id: nodeCounter.value.toString(),
    targetPosition: Position.Left,
    sourcePosition: Position.Right,
    type: "custom",
  });
  edges.value.push({
    id: `e${nodeCounter.value}-${routerId}`,
    source: `${String(routerId)}`,
    target: `${nodeCounter.value}`,
    updatable: true,
    type: "smoothstep",
    sourceHandle: `${routerId}__handle-right`,
    targetHandle: `${nodeCounter.value}__handle-left`,
  });

  nodeCounter.value++;
  nodes.value.push({
    label: "node",
    position: {
      x:
        nodes.value.length !== 0
          ? nodes.value[nodes.value.length - 2].position.x
          : 10,
      y:
        nodes.value.length !== 0
          ? nodes.value[nodes.value.length - 2].position.y + 200
          : 10,
    },
    class: "light",
    id: nodeCounter.value.toString(),
    targetPosition: Position.Top,
    sourcePosition: Position.Bottom,
    type: "custom",
  });
  edges.value.push({
    id: `e${nodeCounter.value}-${routerId}`,
    source: `${String(routerId)}`,
    target: `${nodeCounter.value}`,
    updatable: true,
    type: "smoothstep",
    sourceHandle: `${routerId}__handle-bottom`,
    targetHandle: `${nodeCounter.value}__handle-top`,
  });
};

const handleRemoveNode = (selectedNodeId: string) => {
  const selectedNodeIndex = nodes.value.findIndex(
    (node) => node.id === selectedNodeId
  );
  if (selectedNodeIndex === -1) return;
  nodes.value.splice(selectedNodeIndex, 1);
  const existingNodesIds = nodes.value.map((node) => node.id);
  if (
    edges.value.length === 1 &&
    (edges.value[0].source === selectedNodeId ||
      edges.value[0].target === selectedNodeId)
  ) {
    edges.value.pop();
    return;
  } else {
    edges.value = edges.value
      .filter((edge) => existingNodesIds.includes(edge.source && edge.target))
      .map((edge) => {
        if (edge.source === selectedNodeId) {
          return {
            ...edge,
            id: `e${edge.target}-${nodes.value[selectedNodeIndex - 1].id}`,
            source: nodes.value[selectedNodeIndex - 1].id,
            sourceHandle: `${
              nodes.value[selectedNodeIndex - 1].id
            }__handle-right`,
          };
        }
        if (edge.target === selectedNodeId) {
          return {
            ...edge,
            id: `e${nodes.value[selectedNodeIndex + 1].id}-${edge.source}`,
            target: nodes.value[selectedNodeIndex + 1].id,
            sourceHandle: `${
              nodes.value[selectedNodeIndex + 1].id
            }__handle-right`,
          };
        }
        return edge;
      });
    edges.value = removeDuplicatesEdges(
      edges.value.filter((edge) =>
        existingNodesIds.includes(edge.target && edge.source)
      )
    );
  }
};
const handleRemoveRouter = (selectedNodeId: string) => {
  const nodeIdsToDelete = [
    selectedNodeId,
    +selectedNodeId + 1,
    +selectedNodeId + 2,
  ];
  nodeIdsToDelete.forEach((nodeId) => {
    const selectedNodeIndex = nodes.value.findIndex(
      (node) => node.id === nodeId.toString()
    );
    if (selectedNodeIndex === -1) return;
    nodes.value.splice(selectedNodeIndex, 1);
  });
  edges.value = edges.value.filter(
    (edge) => edge.source !== selectedNodeId && edge.target !== selectedNodeId
  );
};

const removeDuplicatesEdges = (edges: Edge[]) => {
  const uniqueEdges = [];
  const edgeIds = new Set();
  for (const edge of edges) {
    const edgeId = `e${edge.target}-${edge.source}`;
    if (!edgeIds.has(edgeId)) {
      uniqueEdges.push(edge);
      edgeIds.add(edgeId);
    }
  }
  return uniqueEdges;
};
const secondSmallerNumberThanCounter = (arr: string[], counter: string) => {
  const sortedArr = arr.sort(
    (secondItem: string, firstItem: string) =>
      parseInt(secondItem) - parseInt(firstItem)
  );
  const counterIndex = sortedArr.indexOf(counter);
  if (counterIndex === -1) {
    return null;
  }
  return sortedArr[counterIndex - 1];
};
const handleSelectedDropdownAction = (selectedAction: string) => {
  if (selectedAction === dropdownOption.addRouter) {
    handleAddRouter(selectedNodeId.value!);
  }
};
</script>
