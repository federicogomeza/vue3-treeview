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

    <div v-if="enableSelectAll" class="select-controls">
      <label>
        <input type="checkbox" @change="selectAllNodes($event)" />
        Seleccionar todo
      </label>
      <button @click="clearSelection" class="clear-btn">Borrar selección</button>
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
  enableSelectAll: {
    type: Boolean,
    default: false,
  },
});

const emit = defineEmits(['select', 'view']);
const treeDataReactive = ref(JSON.parse(JSON.stringify(props.treeData)));
const selectedNodes = ref(new Set());
const searchTerm = ref("");

// Seleccionar todos los nodos
const selectAllNodes = (event) => {
  selectedNodes.value.clear();
  if (event.target.checked) {
    const selectAll = (nodes) => {
      nodes.forEach(node => {
        selectedNodes.value.add(node);
        if (node.children) {
          selectAll(node.children);
        }
      });
    };
    selectAll(treeDataReactive.value);
  }
  emit('select', Array.from(selectedNodes.value));
};

// Borrar selección
const clearSelection = () => {
  selectedNodes.value.clear();
  emit('select', Array.from(selectedNodes.value));
};

// Control de selección individual
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

// Filtrar y expandir nodos con búsqueda
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

// Computed para nodos visibles
const visibleTreeData = computed(() => {
  if (!searchTerm.value) {
    return treeDataReactive.value;
  }
  return filterAndExpandNodes(treeDataReactive.value, searchTerm.value);
});

// Observador para término de búsqueda
watch(searchTerm, () => {
  treeDataReactive.value = JSON.parse(JSON.stringify(props.treeData));
  filterAndExpandNodes(treeDataReactive.value, searchTerm.value);
  nextTick();
});
</script>

<style scoped>
.treeview ul {
  list-style-type: none;
  padding-left: 1rem;
}

.search-container, .select-controls {
  margin-bottom: 10px;
}

.search-input {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.select-controls label {
  cursor: pointer;
}

.select-controls label input {
  margin-right: 5px;
}

.clear-btn {
  margin-left: 10px;
  background-color: #f5f5f5;
  border: 1px solid #ccc;
  padding: 5px 10px;
  border-radius: 4px;
  cursor: pointer;
}
</style>
