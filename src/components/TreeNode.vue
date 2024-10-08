<template>
  <li :class="{ selected: isSelected }" role="treeitem" aria-expanded="expanded">
    <div
      class="node-container"
      :class="{
        'is-hovered': isHovered,
        'is-pressed': isPressed,
        'is-focused': isFocused
      }"
      @mouseover="setHovered(true)"
      @mouseleave="setHovered(false)"
      @mousedown="setPressed(true)"
      @mouseup="setPressed(false)"
      @focus="setFocused(true)"
      @blur="setFocused(false)"
    >
      <!-- Flecha para expandir si el nodo tiene hijos -->
      <span
        v-if="hasChildren"
        @click="toggle"
        class="arrow"
        :class="{ 'is-expanded': expanded }"
      >
        {{ expanded ? '▼' : '▶' }}
      </span>

      <!-- Checkbox para selección múltiple -->
      <input
        v-if="allowMultipleSelection"
        type="checkbox"
        :checked="isSelected"
        @click.stop="selectNode"
        class="custom-checkbox"
      />

      <!-- Texto del nodo que maneja el nuevo evento click -->
      <span
        @click.stop="viewNode"
        class="node-label"
        :class="{ 'is-leaf': !hasChildren }"
      >
        {{ node.label }}
      </span>
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
        @select="$emit('select', child)"
        @view="$emit('view', $event)"
      />
    </ul>
  </li>
</template>

<script setup>
import { ref, computed } from 'vue';

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

const isHovered = ref(false);
const isPressed = ref(false);
const isFocused = ref(false);

const setHovered = (value) => (isHovered.value = value);
const setPressed = (value) => (isPressed.value = value);
const setFocused = (value) => (isFocused.value = value);

const hasChildren = computed(() => props.node.children && props.node.children.length > 0);
const expanded = ref(props.node.expanded || false);

const toggle = () => {
  expanded.value = !expanded.value;
  emit('toggle', props.node);
};

// Verifica si el nodo actual está seleccionado
const isSelected = computed(() => {
  return props.selectedNodes?.id === props.node.id;
});

// Lógica para seleccionar un nodo al hacer clic en el label
const selectNode = () => {
  emit('select', props.node);
};

// Lógica para ver el nodo al hacer clic en el label
const viewNode = () => {
  emit('view', props.node); // Emitir el evento "view" al hacer clic en el label
};
</script>

<style scoped>
/* Nodo contenedor */
.node-container {
  display: flex;
  align-items: center;
  padding: 5px 10px;
  border-left: 4px solid transparent;
  transition: background-color 0.3s, border-left-color 0.3s;
  border-radius: 5px;
  cursor: pointer;
}

/* Nodo seleccionado */
.node-container.selected {
  background-color: #e6f7ff; /* Fondo azul claro */
  border-left-color: #004080; /* Borde izquierdo azul oscuro */
  padding: 5px 10px;
  border-left-width: 6px; /* Borde izquierdo más grueso */
  border-radius: 5px;
}

/* Estados: hover, press, focus */
.node-container.is-hovered {
  background-color: #f5f8fa; /* Hover: Fondo gris claro */
}

.node-container.is-pressed {
  background-color: #e0e7ed; /* Press: Fondo gris oscuro */
}

.node-container.is-focused {
  outline: 2px solid #0056b3; /* Focus: Borde azul */
}

/* Flecha desplegable */
.arrow {
  margin-right: 8px;
  cursor: pointer;
  color: #333;
  transition: color 0.3s;
}

.arrow.is-expanded {
  color: #007bff; /* Nodo desplegado: Flecha azul */
}

/* Checkbox personalizado */
.custom-checkbox {
  margin-right: 12px;
  width: 18px;
  height: 18px;
  accent-color: #007bff;
  border-radius: 5px;
  background-color: white;
  border: 1px solid #007bff;
}

/* Estilos del label en Nodo y Hoja */
.node-label {
  color: #007bff;
  font-weight: bold;
  transition: color 0.3s;
}

.node-label:hover {
  text-decoration: underline;
}

.node-container.selected .node-label {
  color: #0056b3;
}

.treeview ul {
  list-style-type: none;
  padding-left: 1.5rem;
  margin: 0;
}

.treeview li {
  cursor: pointer;
}
</style>
