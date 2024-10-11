<template>
  <li v-if="isVisible" class="tree-node--line" role="treeitem" :aria-expanded="localExpanded" :aria-selected="isSelected" aria-haspopup="hasChildren">
    <div
      :class="[
        'tree-node__container', 
        { 
          'tree-node--selected': isSelected, 
          'tree-node--hovered': isHovered, 
          'tree-node--focused': isFocused,
          'tree-node--pressed': isPressed,
          'tree-node--expanded': localExpanded 
        }
      ]"
      tabindex="0"
      @focus="handleFocus"
      @blur="isFocused = false"
      @keydown="handleKeyDown"
    >
      <!-- Indicador de expansión -->
      <span v-if="hasChildren" class="tree-node__expand-icon" @click.stop="expandNode">
        {{ localExpanded ? '▼' : '▶' }}
      </span>

      <!-- Checkbox, con tabindex -1 para excluir del flujo de navegación -->
      <div v-if="allowMultipleSelection" class="tree-node__checkbox-container" @click.stop="selectNode">
        <input
          type="checkbox"
          class="tree-node__checkbox"
          :checked="isSelected"
          tabindex="-1"  
          aria-label="Select Node"
        />
      </div>

      <!-- Contenedor del Label -->
      <div class="tree-node__label-container" @click.stop="handleLabelClick">
        <span class="tree-node__label">{{ node.label }}</span>
      </div>
    </div>

    <!-- Nodos hijos -->
    <ul v-if="localExpanded" class="tree-node__children" role="group">
      <TreeNode
        v-for="(child, index) in node.children"
        :key="child.id"
        :node="child"
        :isVisible="child.isVisible !== false"
        :expanded="child.expanded || false"
        :selectedNodes="selectedNodes"
        :allowMultipleSelection="allowMultipleSelection"
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

const hasChildren = computed(() => props.node.children && props.node.children.length > 0);

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

const isSelected = computed(() => props.selectedNodes.has(props.node));

const selectNode = () => {
  emit('select', props.node);
};

// Gestión del foco para navegación y accesibilidad
const handleFocus = () => {
  isFocused.value = true;
};

const handleKeyDown = (event) => {
  switch (event.key) {
    case 'ArrowDown':
      moveFocusToNext();
      break;
    case 'ArrowUp':
      moveFocusToPrevious();
      break;
    case 'ArrowRight':
      if (hasChildren.value && !localExpanded.value) expandNode();
      break;
    case 'ArrowLeft':
      if (hasChildren.value && localExpanded.value) expandNode();
      break;
    case 'Enter':
    case ' ':
      if (props.allowMultipleSelection) {
        selectNode();
      } else {
        expandNode();
      }
      event.preventDefault();
      break;
  }
};

const moveFocusToNext = () => {
  const focusableElements = document.querySelectorAll('.tree-node__container');
  const currentIndex = Array.from(focusableElements).indexOf(document.activeElement);
  const nextElement = focusableElements[currentIndex + 1] || focusableElements[0];
  nextElement.focus();
};

const moveFocusToPrevious = () => {
  const focusableElements = document.querySelectorAll('.tree-node__container');
  const currentIndex = Array.from(focusableElements).indexOf(document.activeElement);
  const prevElement = focusableElements[currentIndex - 1] || focusableElements[focusableElements.length - 1];
  prevElement.focus();
};
</script>

<style scoped>
.tree-node__container {
  display: flex;
  align-items: center;
  padding: 8px;
  border-left: 4px solid transparent;
  cursor: pointer;
  position: relative;
}

.tree-node--selected {
  background-color: #eaf1f5;
  border-left-color: #004b87;
}

.tree-node--pressed {
  background-color: #d0e4f1;
}

.tree-node--focused {
  outline: 2px solid #004b87;
}

.tree-node--hovered {
  background-color: #f0f0f0;
}

.tree-node__checkbox-container {
  margin-right: 8px;
}

.tree-node__label-container {
  cursor: pointer;
}

.tree-node__label {
  font-size: 14px;
  color: #1d4b87;
  font-weight: 500;
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
  margin-right: 8px;
  color: #004b87;
}

.tree-node__children {
  padding-left: 1.5rem;
  position: relative;
  list-style-type: none; 
}

.tree-node--line {
  list-style-type: none; 
}

.tree-node--line::before {
  content: "";
  position: absolute;
  left: 19px;
  top: 0;
  bottom: 0;
  width: 1px;
  background-color: #ccc;
}
</style>
