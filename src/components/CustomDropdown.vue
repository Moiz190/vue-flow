<template>
  <div
    class="dropdown"
    id="dropdownMenu"
    :style="{
      top: `${top}px`,
      left: `${left}px`,
      transform: isMenuVisible ? 'scale(1)' : 'scale(0)',
      transformOrigin: isMenuVisible ? 'top left' : '',
      transition: isMenuVisible ? 'transform 200ms ease-in-out' : 'none',
    }"
  >
    <div
      v-for="option in dropdownOptions"
      :key="option.label"
      class="dropdownItem"
      @click="handleAction(option.action)"
    >
      {{ option.label }}
    </div>
  </div>
</template>
<script setup lang="ts">
import { dropdownOption } from "../interfaces";
interface IProps {
  left: number;
  top: number;
  isMenuVisible: boolean;
}
const emit = defineEmits<{
  (event: "selected-action", payload: string): void;
}>();
withDefaults(defineProps<IProps>(), {
  left: 0,
  top: 0,
  isMenuVisible: false,
});
const dropdownOptions = [
  { label: "Add new", action: dropdownOption.addNode },
  { label: "Filter", action: dropdownOption.addFilter },
  { label: "Add Router", action: dropdownOption.addRouter },
  { label: "Loop", action: dropdownOption.addLoop },
  { label: "End", action: dropdownOption.end },
  { label: "Delay", action: dropdownOption.delay },
  { label: "Copy Step", action: dropdownOption.copyStep },
];
const handleAction = (clickedAction: string) => {
  emit("selected-action", clickedAction);
};
</script>
<style scoped>
#dropdownMenu {
  position: fixed;
  z-index: 100000;
  width: 10rem;
  background: #d7d7d7;
  border-radius: 5px;
}
.dropdown .dropdownItem {
  padding: 0.7rem;
  font-size: 0.9rem;
  cursor: pointer;
  border-radius: inherit;
}
.dropdown .dropdownItem:hover {
  background: #cecece;
}
</style>
