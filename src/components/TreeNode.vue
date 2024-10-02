<template>
  <li :class="{ selected: isSelected }" role="treeitem" aria-expanded="expanded">
    <div class="node-container">
      <!-- Flecha para expandir si el nodo tiene hijos -->
      <span v-if="hasChildren" @click="toggle">{{ expanded ? '▼' : '▶' }}</span>

      <!-- Checkbox para selección múltiple -->
      <input
        v-if="allowMultipleSelection"
        type="checkbox"
        :checked="isSelected"
        @click.stop="selectNode"
      />

      <!-- Texto del nodo que maneja el nuevo evento click -->
      <span @click.stop="viewNode">{{ node.label }}</span>
    </div>

    <!-- Mostrar hijos si el nodo está expandido -->
    <ul v-if="expanded && hasChildren">
      <TreeNode
        v-for="(child, index) in node.children"
        :key="child.id"
        :node="child"
        :selectedNodes="selectedNodes"
        :allowMultipleSelection="allowMultipleSelection"
        :enableReordering="enableReordering"
        @toggle="$emit('toggle', child)"
        @select="handleSelect"
        @view="$emit('view', $event)" 
      />
    </ul>
  </li>
</template>

<script setup>
import { ref, computed } from 'vue';

// Props para recibir el nodo y configuraciones adicionales
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
});

const emit = defineEmits(['toggle', 'select', 'view']);

// Verifica si el nodo tiene hijos
const hasChildren = computed(() => {
  return props.node.children && props.node.children.length > 0;
});

// Estado reactivo interno para la expansión
const expanded = ref(props.node.expanded || false);

// Alterna la expansión del nodo actual
const toggle = () => {
  expanded.value = !expanded.value;
  emit('toggle', props.node);
};

// Verifica si el nodo está seleccionado
const isSelected = computed(() => {
  return props.selectedNodes.has(props.node.id);
});

// Maneja la selección del nodo actual solo desde el checkbox
const selectNode = () => {
  emit('select', props.node);
};

// Maneja la selección del nodo hijo
const handleSelect = (childNode) => {
  emit('select', childNode);
};

// Maneja el clic en el label (nombre del item)
const viewNode = () => {
  emit('view', props.node); // Emitir el evento "view" con el objeto del nodo actual
};
</script>

<style scoped>
.node-container {
  display: flex;
  align-items: center;
}

.node-container.selected {
  background-color: #e6f7ff;
}

.node-container:hover {
  background-color: #f0f0f0;
}

.node-container span {
  cursor: pointer;
}
</style>
