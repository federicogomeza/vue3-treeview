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
  {
    id: 9,
    label: 'Nodo 3',
    expanded: false,
    children: [
      {
        id: 10,
        label: 'Nodo 3.1',
        expanded: false,
        children: [
          {
            id: 11,
            label: 'Nodo 3.1.1',
            expanded: false,
            children: [
              {
                id: 12,
                label: 'Nodo 3.1.1.1',
                expanded: false,
                children: [
                  {
                    id: 13,
                    label: 'Hoja 3.1.1.1.1',
                    expanded: false,
                    children: [],
                  },
                ],
              },
            ],
          },
        ],
      },
      {
        id: 14,
        label: 'Hoja 3.2',
        expanded: false,
        children: [],
      },
    ],
  },
];

const handleSelectionChange = (selectedNodes: any) => {
  console.log('Nodos seleccionados:', selectedNodes);
};

// Manejador del evento "view" para visualizar la información del nodo
const handleViewNode = (node: any) => {
  console.log('Visualizando información del nodo:', node);
};
</script>

<template>
  <TreeView
    :treeData="treeData"
    :allowMultipleSelection="true"
    enableSearch
    enableSelectAll
    :enableReordering="false"
    @select="handleSelectionChange"
    @view="handleViewNode"
  />
</template>