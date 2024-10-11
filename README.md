# TreeView Component

El componente `TreeView` permite mostrar una estructura jerárquica de datos en forma de árbol con opciones de selección múltiple, búsqueda y expansión de nodos. También incluye opciones para seleccionar todos los nodos y borrar la selección.

## Ejemplos de Uso

### Ejemplo Básico
```vue
<template>
  <TreeView :treeData="treeData" />
</template>

<script setup>
const treeData = [
  { id: 1, label: 'Nodo 1', children: [{ id: 2, label: 'Nodo 1.1' }] },
  { id: 3, label: 'Nodo 2' }
];
</script>
```

### Con Título, Búsqueda y Selección Múltiple
```vue
<template>
  <TreeView
    title="Estructura"
    :treeData="treeData"
    :allowMultipleSelection="true"
    enableSearch
    @select="handleSelect"
    @view="handleView"
  />
</template>

<script setup>
const treeData = [
  {
    id: 1,
    label: 'Nodo 1',
    expanded: true,
    children: [
      { id: 2, label: 'Nodo 1.1', children: [] }
    ]
  },
  { id: 3, label: 'Nodo 2', children: [] }
];

const handleSelect = (selectedNodes) => {
  console.log('Seleccionados:', selectedNodes);
};

const handleView = (node) => {
  console.log('Visualizando:', node);
};
</script>
```

### Con Selección Múltiple Dinámica y Controles de Selección
```vue
<template>
  <TreeView
    title="Estructura"
    :treeData="treeData"
    :allowMultipleSelectionToggle="true"
    :showSelectionOptions="true"
    enableSearch
    @select="handleSelect"
  />
</template>

<script setup>
const treeData = [
  {
    id: 1,
    label: 'Nodo 1',
    expanded: true,
    children: [
      { id: 2, label: 'Nodo 1.1', children: [] }
    ]
  },
  { id: 3, label: 'Nodo 2', children: [] }
];

const handleSelect = (selectedNodes) => {
  console.log('Nodos seleccionados:', selectedNodes);
};
</script>
```





## Notas Adicionales

- **Búsqueda**: El árbol se actualiza automáticamente cuando se introduce texto en el campo de búsqueda.
- **Selección múltiple**: Cuando se habilita, permite seleccionar más de un nodo a la vez. Con la opción `allowMultipleSelectionToggle`, el usuario puede habilitar/deshabilitar esta funcionalidad dinámicamente.


## Props

| Prop                       | Tipo     | Requerido | Valor por Defecto | Descripción |
|----------------------------|----------|-----------|--------------------|-------------|
| `treeData`                 | Array    | Sí        | -                  | La estructura de datos del árbol. Debe contener objetos con propiedades `id`, `label`, `expanded` (opcional) y `children` (opcional). |
| `title`                    | String   | No        | -                  | Texto opcional que se muestra como título del componente. |
| `allowMultipleSelection`   | Boolean  | No        | `false`            | Habilita la selección múltiple de nodos. |
| `allowMultipleSelectionToggle` | Boolean | No     | `false`            | Muestra un checkbox que permite habilitar/deshabilitar la selección múltiple dinámicamente. |
| `enableSearch`             | Boolean  | No        | `false`            | Muestra un campo de búsqueda para filtrar nodos dentro del árbol. |
| `showSelectionOptions`     | Boolean  | No        | `false`            | Muestra opciones de "Seleccionar todo" y "Borrar selección" cuando la selección múltiple está habilitada. |

## Eventos

| Evento      | Descripción                                       | Argumentos |
|-------------|---------------------------------------------------|------------|
| `select`    | Se dispara cuando se selecciona un nodo.          | Array de nodos seleccionados |
| `view`      | Se dispara cuando se hace clic en el label de un nodo. | Nodo visualizado |
| `expand`    | Se dispara cuando se expande o colapsa un nodo.   | Nodo expandido o colapsado |
