<template>
  <div class="treeview">
    <div v-if="enableSearch" class="search-container">
      <input
        type="text"
        v-model="searchTerm"
        placeholder="Buscar"
        class="search-input"
        @input="handleSearchInput"
      />
    </div>
    <ul>
      <TreeNode
        v-for="(node, index) in visibleTreeData"
        :key="node.id"
        :node="node"
        :isVisible="node.isVisible"
        :expanded="node.expanded"
        :selectedNodes="selectedNodes"
        :allowMultipleSelection="allowMultipleSelection"
        :enableReordering="enableReordering"
        @select="handleSelect"
        @view="handleView"
        @expand="handleExpand"
      />
    </ul>
  </div>
</template>

<script setup>
import { ref, computed, watch, nextTick } from 'vue';
import TreeNode from './TreeNode.vue';

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
  enableSearch: {
    type: Boolean,
    default: false,
  },
});

const emit = defineEmits(['select', 'view']);
const treeDataReactive = ref(JSON.parse(JSON.stringify(props.treeData)));
const selectedNodes = ref(new Set());
const searchTerm = ref("");

const handleSelect = (node) => {
  if (props.allowMultipleSelection) {
    if (selectedNodes.value.has(node)) {
      selectedNodes.value.delete(node);
    } else {
      selectedNodes.value.add(node);
    }
  } else {
    selectedNodes.value.clear();
    selectedNodes.value.add(node);
  }
  emit('select', Array.from(selectedNodes.value));
};


const handleView = (node) => {
  emit('view', node);
};


// Filtra los nodos y ajusta visibilidad y expansión
const filterAndExpandNodes = (nodes, term) => {
  return nodes.map(node => {
    const matches = node.label.toLowerCase().includes(term.toLowerCase());
    let filteredChildren = [];

    if (node.children) {
      filteredChildren = filterAndExpandNodes(node.children, term);
    }

    node.isVisible = matches || filteredChildren.some(child => child.isVisible);
    node.expanded = node.isVisible && node.children && filteredChildren.length > 0;

    return { ...node, children: filteredChildren };
  });
};

// Computed para obtener nodos visibles
const visibleTreeData = computed(() => {
  if (!searchTerm.value) {
    return treeDataReactive.value;
  }
  return filterAndExpandNodes(treeDataReactive.value, searchTerm.value);
});

// Observador para actualizar el árbol cuando cambia el término de búsqueda
watch(searchTerm, () => {
  treeDataReactive.value = JSON.parse(JSON.stringify(props.treeData)); // Reseteo de data reactiva
  filterAndExpandNodes(treeDataReactive.value, searchTerm.value);
  nextTick();
});
</script>

<style scoped>
.treeview ul {
  list-style-type: none;
  padding-left: 1rem;
}

.search-container {
  margin-bottom: 10px;
}

.search-input {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 4px;
}
</style>
