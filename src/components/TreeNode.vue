<template>
  <li v-if="isVisible" :class="{ 'tree-line': hasChildren }">
    <div
      :class="[
        'tree-node__container', 
        { 
          'tree-node__container--selected': isSelected, 
          'tree-node__container--hovered': isHovered, 
          'tree-node__container--focused': isFocused,
          'tree-node__container--pressed': isPressed,
          'tree-node__container--expanded': localExpanded 
        }
      ]"
      @mouseenter="isHovered = true"
      @mouseleave="isHovered = false"
      @mousedown="isPressed = true"
      @mouseup="isPressed = false"
      @focus="isFocused = true"
      @blur="isFocused = false"
      tabindex="0"
    >
      <span v-if="hasChildren" class="tree-node__expand-icon" @click.stop="expandNode">
        {{ localExpanded ? '▼' : '▶' }}
      </span>

      <div v-if="allowMultipleSelection" class="tree-node__checkbox-container" @click.stop="selectNode">
        <input
          type="checkbox"
          class="tree-node__checkbox"
          :checked="isSelected"
          aria-label="Select Node"
        />
      </div>

      <div class="tree-node__label-container" @click.stop="handleLabelClick">
        <span class="tree-node__label">{{ node.label }}</span>
      </div>
    </div>

    <ul v-if="localExpanded" class="tree-node__list">
      <TreeNode
        v-for="(child, index) in node.children"
        :key="child.id"
        :node="child"
        :isVisible="child.isVisible !== false"
        :expanded="child.expanded || false"
        :selectedNodes="selectedNodes"
        :allowMultipleSelection="allowMultipleSelection"
        :enableReordering="enableReordering"
        @view="$emit('view', $event)"
        @select="$emit('select', $event)"
        @expand="$emit('expand', $event)"
      />
    </ul>
  </li>
</template>

<script setup>
import { ref, computed, watch } from 'vue';

const props = defineProps({
  node: {
    type: Object,
    required: true,
  },
  selectedNodes: {
    type: Object,
    required: true,
  },
  allowMultipleSelection: {
    type: Boolean,
    default: false,
  },
  enableReordering: {
    type: Boolean,
    default: false,
  },
  expanded: {
    type: Boolean,
    default: false,
  },
  isVisible: {
    type: Boolean,
    default: true,
  },
});

const emit = defineEmits(['expand', 'select', 'view']);

const localExpanded = ref(props.expanded);
const isHovered = ref(false);
const isPressed = ref(false);
const isFocused = ref(false);

const hasChildren = computed(() => {
  return props.node.children && props.node.children.length > 0;
});

watch(() => props.expanded, (newVal) => {
  localExpanded.value = newVal;
});

const expandNode = () => {
  localExpanded.value = !localExpanded.value;
  emit('expand', props.node);
};

const handleLabelClick = () => {
  emit('view', props.node);
}

const isSelected = computed(() => {
  return props.selectedNodes.has(props.node);
});

const selectNode = () => {
  emit('select', props.node);
};
</script>

<style scoped>
.tree-node__container {
  display: flex;
  align-items: center;
  padding: 0.25rem;
  border-left: 4px solid transparent;
  cursor: pointer;
  position: relative;
}

.tree-node__container--selected {
  background-color: #eaf1f5;
  border-left-color: #004b87;
}

.tree-node__container--pressed {
  background-color: #d0e4f1;
}

.tree-node__container--focused {
  outline: 2px solid #004b87;
}

.tree-node__container--hovered {
  background-color: #f0f0f0;
}

.tree-node__checkbox-container {
  margin-right: 0.5rem;
}

.tree-node__label-container {
  cursor: pointer;
}

.tree-node__label {
  font-size: 14px;
  color: #1d4b87;
}

.tree-node__label:hover {
  color: #1a3e72;
}

.tree-node__checkbox {
  appearance: none;
  width: 18px;
  height: 18px;
  background-color: #eaf1f5;
  border: 2px solid #004b87;
  border-radius: 3px;
  position: relative;
}

.tree-node__checkbox:checked {
  background-color: #0078d4;
  border-color: #0078d4;
}

.tree-node__checkbox:checked::after {
  content: '✔';
  position: absolute;
  top: -2px;
  left: 2px;
  font-size: 12px;
  color: white;
}

.tree-node__expand-icon {
  margin-right: 0.5rem;
  color: #004b87;
}

.tree-node__list {
  padding-left: 1.5rem;
  position: relative;
}

.tree-line::before {
  content: "";
  position: absolute;
  left: -1rem;
  top: 0;
  bottom: 0;
  width: 1px;
  background-color: #ccc;
}
</style>
