<template>
  <div class="treeview" role="tree">
    <ul>
      <TreeNode
        v-for="(node, index) in treeDataReactive"
        :key="node.id"
        :node="node"
        :selectedNodes="selectedNodeIds"
        :allowMultipleSelection="allowMultipleSelection"
        :enableReordering="enableReordering"
        @toggle="toggleNode"
        @select="selectNode"
        @dragstart="dragStart"
        @drop="dropNode"
        @dragover.prevent
        aria-selected="selectedNodeIds.has(node.id)"
      />
    </ul>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import TreeNode from './TreeNode.vue';

// Props del componente
const props = defineProps({
  treeData: {
    type: Array,
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

// Emit para comunicar con el componente padre
const emit = defineEmits(['select']);

// Hacer la data reactiva
const treeDataReactive = ref([...props.treeData]);
const selectedNodeIds = ref(new Set()); // Set para guardar los IDs de nodos seleccionados
let draggedNode = null;
let draggedParent = null;

// Alterna la expansión del nodo
const toggleNode = (node) => {
  node.expanded = !node.expanded;
};

// Selecciona o deselecciona el nodo usando su id único
const selectNode = (node) => {
  if (props.allowMultipleSelection) {
    if (selectedNodeIds.value.has(node.id)) {
      selectedNodeIds.value.delete(node.id); // Deseleccionar el nodo
    } else {
      selectedNodeIds.value.add(node.id); // Seleccionar el nodo
    }
  } else {
    selectedNodeIds.value.clear(); // Limpiar selección previa si no es multiselección
    selectedNodeIds.value.add(node.id); // Seleccionar solo el nodo actual
  }

  // Emitir los IDs de los nodos seleccionados al componente padre
  emit('select', Array.from(selectedNodeIds.value));
};

// Buscar el padre del nodo
const findParent = (tree, targetNode) => {
  for (const node of tree) {
    if (node.children && node.children.includes(targetNode)) {
      return node;
    }
    if (node.children) {
      const parent = findParent(node.children, targetNode);
      if (parent) {
        return parent;
      }
    }
  }
  return null;
};

// Manejar el drag
const dragStart = (node, event) => {
  draggedNode = node;
  draggedParent = findParent(treeDataReactive.value, node);
};

// Manejar el drop
const dropNode = (targetNode, event) => {
  if (draggedNode && draggedNode !== targetNode) {
    if (draggedParent) {
      removeNode(draggedParent.children, draggedNode);
    } else {
      removeNode(treeDataReactive.value, draggedNode);
    }

    if (!targetNode.children) {
      targetNode.children = [];
    }
    targetNode.children.push(draggedNode);

    treeDataReactive.value = [...treeDataReactive.value];
  }

  draggedNode = null;
  draggedParent = null;
};

// Función para eliminar un nodo
const removeNode = (tree, nodeToRemove) => {
  for (let i = 0; i < tree.length; i++) {
    if (tree[i] === nodeToRemove) {
      tree.splice(i, 1);
      return;
    }
    if (tree[i].children) {
      removeNode(tree[i].children, nodeToRemove);
    }
  }
};
</script>

<style scoped>
.treeview ul {
  list-style-type: none;
  padding-left: 1rem;
}

.treeview li {
  cursor: pointer;
}
</style>
