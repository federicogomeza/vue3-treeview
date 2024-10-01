<template>
  <div class="treeview" role="tree">
    <ul>
      <TreeNode
        v-for="(node, index) in treeDataReactive"
        :key="node.id"
        :node="node"
        :selectedNodes="selectedNodes"
        :allowMultipleSelection="allowMultipleSelection"
        :enableReordering="enableReordering"
        @toggle="toggleNode"
        @select="selectNode"
        @dragstart="dragStart"
        @drop="dropNode"
        @dragover.prevent
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
const selectedNodes = ref(new Set()); // Set para guardar los objetos de nodos seleccionados
let draggedNode = null;
let draggedParent = null;

// Alterna la expansión del nodo
const toggleNode = (node) => {
  node.expanded = !node.expanded;
};

// Selecciona o deselecciona el nodo usando el objeto completo
const selectNode = (node) => {
  if (props.allowMultipleSelection) {
    // Si el nodo ya está seleccionado, lo eliminamos del Set
    if (selectedNodes.value.has(node)) {
      selectedNodes.value.delete(node);
    } else {
      selectedNodes.value.add(node);
    }
  } else {
    // Si no se permite la selección múltiple, limpiamos el Set
    selectedNodes.value.clear();
    selectedNodes.value.add(node);
  }

  // Emitir los objetos completos de los nodos seleccionados
  emit('select', Array.from(selectedNodes.value));
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
    console.log(`Moviendo ${draggedNode.label} a ${targetNode.label}`);

    // 1. Eliminar el nodo de la posición original
    if (draggedParent) {
      removeNode(draggedParent.children, draggedNode);
    } else {
      removeNode(treeDataReactive.value, draggedNode); // Si es un nodo raíz
    }

    // 2. Añadir el nodo al nuevo destino
    if (!targetNode.children) {
      targetNode.children = [];
    }
    targetNode.children.push(draggedNode);

    // 3. Actualizar el estado del árbol para reflejar el cambio
    treeDataReactive.value = [...treeDataReactive.value];

    console.log('Nodo movido:', draggedNode.label);
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
