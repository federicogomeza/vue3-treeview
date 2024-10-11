<template>
  <div class="treeview">
    <!-- Título opcional y controles superiores -->
    <div class="treeview__header">
      <h2 v-if="title" class="treeview__title">{{ title }}</h2>
      <div v-if="allowMultipleSelectionToggle" class="treeview__toggle-multiselect">
        <label>
          Selección múltiple
          <input type="checkbox" v-model="multipleSelectionEnabled" />
        </label>
      </div>
    </div>
    <div v-if="enableSearch" class="treeview__search">
        <input
          type="text"
          v-model="searchTerm"
          placeholder="Buscar"
          class="treeview__search-input"
          @input="handleSearchInput"
        />
      </div>




      <div v-if="showSelectionOptions" class="treeview__selection-controls">
        <label class="treeview__select-all">
          <input type="checkbox" @change="toggleSelectAll" /> Seleccionar todo
        </label>
        <button class="treeview__clear-button" @click="clearSelection">Borrar selección</button>
      </div>


    <!-- Listado de nodos -->
    <ul>
      <TreeNode
        v-for="(node, index) in visibleTreeData"
        :key="node.id"
        :node="node"
        :isVisible="node.isVisible"
        :expanded="node.expanded"
        :selectedNodes="selectedNodes"
        :allowMultipleSelection="multipleSelectionEnabled"
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
  title: {
    type: String,
    default: '',
  },
  allowMultipleSelection: {
    type: Boolean,
    default: false,
  },
  allowMultipleSelectionToggle: {
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
  showSelectionOptions: {
    type: Boolean,
    default: false,
  },
});

const emit = defineEmits(['select', 'view']);
const treeDataReactive = ref(JSON.parse(JSON.stringify(props.treeData)));
const selectedNodes = ref(new Set());
const searchTerm = ref("");
const multipleSelectionEnabled = ref(props.allowMultipleSelection);

const handleSelect = (node) => {
  if (multipleSelectionEnabled.value) {
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

const toggleSelectAll = (event) => {
  if (event.target.checked) {
    selectedNodes.value = new Set(treeDataReactive.value.flatMap(flattenTree));
  } else {
    selectedNodes.value.clear();
  }
  emit('select', Array.from(selectedNodes.value));
};

const clearSelection = () => {
  selectedNodes.value.clear();
  emit('select', Array.from(selectedNodes.value));
};

const flattenTree = (node) => {
  return [node, ...(node.children ? node.children.flatMap(flattenTree) : [])];
};

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

const visibleTreeData = computed(() => {
  if (!searchTerm.value) {
    return treeDataReactive.value;
  }
  return filterAndExpandNodes(treeDataReactive.value, searchTerm.value);
});

watch(searchTerm, () => {
  treeDataReactive.value = JSON.parse(JSON.stringify(props.treeData));
  filterAndExpandNodes(treeDataReactive.value, searchTerm.value);
  nextTick();
});
</script>

<style scoped>
.treeview__header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.treeview__title {
  font-size: 20px;
  color: #004b87;
  margin: 0;
}

.treeview__toggle-multiselect label {
  font-size: 14px;
  color: #004b87;
}

.treeview__controls {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.treeview__search {
  flex-grow: 1;
}

.treeview__search-input {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.treeview__selection-controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.treeview__select-all {
  display: flex;
  align-items: center;
  font-size: 14px;
  color: black;
}

.treeview__clear-button {
  background-color: #f5f5f5;
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 0.5rem;
  cursor: pointer;
  font-size: 14px;
  color: #333;
}

.treeview__clear-button:hover {
  background-color: #e0e0e0;
}
</style>
