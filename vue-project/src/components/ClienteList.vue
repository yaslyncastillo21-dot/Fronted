¡Absolutamente\! Aquí tienes el componente de lista adaptado para el **Sistema de Registro de Clientes**, siguiendo los campos de tu consulta SQL anterior (`id_cliente`, `nombre`, `telefono`, `direccion`).

Este componente está optimizado para funcionar como `ClienteList.vue` y reemplaza al componente original `EstudianteList.vue`.

## `ClienteList.vue` (Componente de Lista de Clientes)

```vue
<template>
  <div class="cliente-list-container mt-10">
    <h2 class="text-2xl font-semibold text-gray-700 mb-4 border-b-2 border-blue-500 pb-2">
      Clientes Registrados
    </h2>
    
    <div v-if="clientes.length > 0" class="overflow-x-auto bg-white rounded-lg shadow">
      <table class="min-w-full divide-y divide-gray-200">
        <thead class="bg-gray-50">
          <tr>
            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
              ID Cliente
            </th>
            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
              Nombre Completo
            </th>
            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
              Teléfono
            </th>
            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
              Dirección
            </th>
          </tr>
        </thead>
        <tbody class="bg-white divide-y divide-gray-200">
          <tr v-for="cliente in clientes" :key="cliente.id_cliente" class="hover:bg-gray-50 transition">
            <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">
              {{ cliente.id_cliente }}
            </td>
            <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-700">
              {{ cliente.nombre }}
            </td>
            <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-700 text-left">
              {{ cliente.telefono }}
            </td>
            <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-700">
              {{ cliente.direccion }}
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <div v-else class="text-center bg-yellow-100 border border-yellow-300 text-yellow-800 px-4 py-3 rounded-lg">
      <p>No hay clientes registrados. ¡Usa el formulario principal para agregar el primero!</p>
    </div>
  </div>
</template>

<script>
// CAMBIO: Renombrado del componente a 'ClienteList'
export default {
  name: 'ClienteList',
  props: {
    // CAMBIO: El prop ahora se llama 'clientes'
    clientes: {
      type: Array,
      required: true,
      default: () => []
    }
  }
};
</script>

<style scoped>
/* Estilos específicos para este componente si son necesarios */
</style>
```