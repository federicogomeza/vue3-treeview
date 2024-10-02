<script setup lang="ts">
import TreeView from './components/TreeView.vue';

const treeData = [
  {
    id: 1,
    label: 'Nodo 1',
    expanded: false,
    children: [
      {
        id: 2,
        label: 'Nodo 1.1',
        expanded: false,
        children: [
          {
            id: 3,
            label: 'Nodo 1.1.1',
            expanded: false,
            children: [
              {
                id: 4,
                label: 'Nodo 1.1.1.1',
                expanded: false,
                children: [
                  {
                    id: 5,
                    label: 'Hoja 1.1.1.1.1',
                    expanded: false,
                    children: [],
                  },
                ],
              },
            ],
          },
        ],
      },
    ],
  },
  {
    id: 6,
    label: 'Nodo 2',
    expanded: false,
    children: [
      { id: 7, label: 'Nodo 2.1', expanded: false, children: [] },
      { id: 8, label: 'Nodo 2.2', expanded: false, children: [] },
    ],
  },
];

const handleSelectionChange = (selectedNodes) => {
  console.log('Nodos seleccionados:', selectedNodes);
};

// Manejador del evento "view" para visualizar la información del nodo
const handleViewNode = (node) => {
  console.log('Visualizando información del nodo:', node);
};
</script>

<template>
  <TreeView
    :treeData="treeData"
    :allowMultipleSelection="true"
    :enableReordering="false"
    @select="handleSelectionChange"
    @view="handleViewNode"
  />
</template>
