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

      <!-- Texto del nodo -->
      <span>{{ node.label }}</span>
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

const emit = defineEmits(['toggle', 'select']);

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
  emit('select', props.node); // Emitir el nodo actual para selección
};

// Maneja la selección del nodo hijo
const handleSelect = (childNode) => {
  emit('select', childNode); // Emitir la selección hacia el padre con el nodo hijo
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
  cursor: default; /* No hace nada cuando se hace clic */
}
</style>
