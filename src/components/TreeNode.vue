<template>
  <li v-if="isVisible">
    <div
      :class="[
        'node-container', 
        { 
          selected: isSelected, 
          'is-hovered': isHovered, 
          'is-focused': isFocused,
          'is-pressed': isPressed,
          expanded: localExpanded 
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
      <!-- Indicador de expansión -->
      <span v-if="hasChildren" class="expand-icon" @click.stop="toggleNode">
        {{ localExpanded ? '▼' : '▶' }}
      </span>

      <!-- Checkbox para selección múltiple -->
      <input
        v-if="allowMultipleSelection"
        type="checkbox"
        class="node-checkbox"
        :checked="isSelected"
        @click.stop="selectNode"
      />

      <!-- Etiqueta del nodo -->
      <span class="node-label" @click.stop="viewNode">{{ node.label }}</span>
    </div>

    <!-- Nodos hijos, visibles solo si localExpanded es true -->
    <ul v-if="localExpanded && hasChildren">
      <TreeNode
        v-for="(child, index) in node.children"
        :key="child.id"
        :node="child"
        :isVisible="child.isVisible !== false"
        :expanded="child.expanded || false"
        :selectedNodes="selectedNodes"
        :allowMultipleSelection="allowMultipleSelection"
        :enableReordering="enableReordering"
        @toggle="handleToggle"
        @select="$emit('select', $event)"
        @view="$emit('view', $event)"
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

const emit = defineEmits(['toggle', 'select', 'view']);

// Estados locales
const localExpanded = ref(props.expanded);
const isHovered = ref(false);
const isPressed = ref(false);
const isFocused = ref(false);

// Verifica si el nodo tiene hijos
const hasChildren = computed(() => {
  return props.node.children && props.node.children.length > 0;
});

// Observa cambios en props.expanded para sincronizar localExpanded
watch(() => props.expanded, (newVal) => {
  localExpanded.value = newVal;
});

// Alterna la expansión del nodo actual
const toggleNode = () => {
  localExpanded.value = !localExpanded.value;
  emit('toggle', props.node);
};

// Verifica si el nodo está seleccionado
const isSelected = computed(() => {
  return props.selectedNodes.has(props.node);
});

// Maneja la selección del nodo actual solo desde el checkbox
const selectNode = () => {
  emit('select', props.node);
};

// Maneja el clic en el label (nombre del item)
const viewNode = () => {
  emit('view', props.node);
};

const handleToggle = (node) => {
  emit('toggle', node); // Emite el nodo actual para que el padre decida expandir o contraer
};
</script>

<style scoped>
.node-container {
  display: flex;
  align-items: center;
  padding: 0.25rem;
  border-left: 4px solid transparent;
  cursor: pointer;
}

.node-container.selected {
  background-color: #eaf1f5;
  border-left-color: #004b87;
}

.node-container.is-pressed {
  background-color: #d0e4f1; /* Estado presionado */
}

.node-container.is-focused {
  outline: 2px solid #004b87; /* Estado focus */
}

.node-container:hover {
  background-color: #f0f0f0;
}

.node-label {
  font-size: 14px;
  color: #1d4b87;
}

.node-label:hover {
  color: #1a3e72;
}

.node-checkbox {
  appearance: none;
  width: 18px;
  height: 18px;
  background-color: #eaf1f5;
  border: 2px solid #004b87;
  border-radius: 3px;
  display: inline-block;
  position: relative;
  margin-right: 0.5rem;
}

.node-checkbox:checked {
  background-color: #0078d4;
  border-color: #0078d4;
}

.node-checkbox:checked::after {
  content: '✔';
  position: absolute;
  top: -2px;
  left: 2px;
  font-size: 12px;
  color: white;
}

.expand-icon {
  margin-right: 0.5rem;
  color: #004b87;
}

ul {
  padding-left: 1.5rem;
}
</style>
