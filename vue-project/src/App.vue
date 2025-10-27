<template>
  <div class="main-container">
    <div class="app-card">
      <h1 class="main-title">
        Sistema de Registro de Clientes
      </h1>

      <div class="form-section">
        <h2 class="form-title">
          <svg xmlns="http://www.w3.org/2000/svg" class="title-icon" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
            <path stroke-linecap="round" stroke-linejoin="round" d="M12 9v3m0 0v3m0-3h3m-3 0H9m12 0a9 9 0 11-18 0 9 9 0 0118 0z" />
          </svg>
          Añadir Nuevo Cliente
        </h2>
        
        <form @submit.prevent="agregarCliente" class="form-grid">
          <div class="form-group full-width">
            <label for="input-nombre" class="form-label">Nombre Completo:</label>
            <input
              id="input-nombre"
              v-model="nuevoCliente.nombre"
              type="text"
              required
              placeholder="Ej: Ana Gómez"
              class="form-input"
            />
          </div>
          
          <div class="split-group">
            
            <div class="form-group">
              <label for="input-telefono" class="form-label">Teléfono:</label>
              <input
                id="input-telefono"
                v-model="nuevoCliente.telefono"
                type="tel"
                required
                placeholder="Ej: 809-555-1234"
                class="form-input"
              />
            </div>
            
            <div class="form-group">
              <label for="input-direccion" class="form-label">Dirección:</label>
              <input
                id="input-direccion"
                v-model="nuevoCliente.direccion"
                type="text"
                required
                placeholder="Ej: Calle 1, No. 5, Sector Centro"
                class="form-input"
              />
            </div>
          </div>
          
          <button
            type="submit"
            class="submit-button"
          >
            <span class="button-content">
              <svg xmlns="http://www.w3.org/2000/svg" class="button-icon" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M10 3a1 1 0 011 1v5h5a1 1 0 110 2h-5v5a1 1 0 11-2 0v-5H4a1 1 0 110-2h5V4a1 1 0 011-1z" clip-rule="evenodd" />
              </svg>
              Guardar y Agregar Cliente
            </span>
          </button>
        </form>
        </div>
      <ClienteList :clientes="clientes" />
      
      <div v-if="showModal" class="modal-backdrop" @click="showModal = false">
        <div class="modal-content" @click.stop>
          <div :class="modalVariant === 'success' ? 'text-green' : 'text-red'" class="modal-icon">
            {{ modalVariant === 'success' ? '🎉' : '⚠️' }}
          </div>
          <h3 class="modal-title-text">{{ modalTitle }}</h3>
          <p class="modal-message">{{ modalMessage }}</p>
          <button 
            @click="showModal = false" 
            :class="modalVariant === 'success' ? 'modal-button-success' : 'modal-button-error'" 
            class="modal-button"
          >
            Entendido
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
// CAMBIO: Importamos el nuevo componente de lista (debe ser creado)
import ClienteList from './components/ClienteList.vue'; 

// CAMBIO: Definición de la interfaz Cliente, basada en la consulta SQL.
interface Cliente {
  id_cliente: number;
  nombre: string;
  telefono: string;
  direccion: string;
}

// CAMBIO: Actualizamos la URL de la API para el endpoint de clientes.
const API_URL = 'http://localhost:2000/api/clientes';

// CAMBIO: Inicializamos los datos del nuevo cliente con los campos correspondientes.
const nuevoCliente = ref({ nombre: '', telefono: '', direccion: '' }); 

// CAMBIO: Usamos 'clientes' en lugar de 'estudiantes' y el tipo Cliente.
const clientes = ref<Cliente[]>([]); 
const showModal = ref(false);
const modalTitle = ref('');
const modalMessage = ref('');
const modalVariant = ref('success');

const mostrarModal = (title: string, message: string, variant = 'success') => {
  modalTitle.value = title;
  modalMessage.value = message;
  modalVariant.value = variant;
  showModal.value = true;
};

// CAMBIO: Renombramos y actualizamos la función de carga.
const cargarClientes = async () => {
  try {
    const res = await fetch(API_URL);
    if (!res.ok) {
      throw new Error(`Error HTTP: ${res.status}`);
    }
    // CAMBIO: Asignamos a 'clientes'.
    clientes.value = await res.json(); 
  } catch (error: any) {
    console.error('Error al cargar clientes:', error);
    clientes.value = [];
    if (!clientes.value.length) {
      mostrarModal('❌ Error de Conexión', `No se pudo conectar con el servidor. Asegúrate de que el backend (http://localhost:2000) esté corriendo y use el endpoint '/api/clientes'.`, 'error');
    }
  }
};

// CAMBIO: Renombramos y actualizamos la función de agregar.
const agregarCliente = async () => {
  try {
    // Los datos se envían tal cual están en nuevoCliente.value, no hay necesidad de parsear un número como la edad.
    const dataToSend = nuevoCliente.value; 

    // Opcional: Podrías añadir validaciones específicas para teléfono o dirección aquí si fuera necesario.
    if (!dataToSend.nombre.trim() || !dataToSend.telefono.trim() || !dataToSend.direccion.trim()) {
      throw new Error('Todos los campos son obligatorios.');
    }
    
    const response = await fetch(API_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(dataToSend),
    });

    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.message || `Error del servidor: ${response.status}`);
    }

    // CAMBIO: Reseteamos los campos del nuevo cliente.
    nuevoCliente.value = { nombre: '', telefono: '', direccion: '' }; 
    mostrarModal('✅ Éxito', 'El cliente fue agregado correctamente.', 'success');
    await cargarClientes(); // Recargar la lista de clientes
  } catch (error: any) {
    console.error('Error al agregar cliente:', error);
    // CAMBIO: Mensaje adaptado.
    mostrarModal('❌ Error al Registrar', error.message || 'Ocurrió un error inesperado al intentar registrar el cliente.', 'error'); 
  }
};

// CAMBIO: Usamos cargarClientes al montar.
onMounted(cargarClientes); 
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap');

/* ESTILOS BASE */
/* PALETA DE COLORES */
/* Primario Oscuro: #6A040F (Rosewood) */
/* Primario Claro: #9A0D1B (Carmin) */
/* Fondo de Sección: #550816 (Chocolate cosmos) */
/* Fondo Principal: #FAEAEB (Rosa muy claro) */


/* ESTILOS BASE */
.main-container {
  min-height: 100vh;
  /* Fondo principal muy claro para contraste */
  background-color: #FAEAEB; /* Rosa muy claro */
  display: flex;
  align-items: center; /* Centrar el formulario verticalmente */
  justify-content: center;
  padding: 2rem 1rem;
  font-family: 'Inter', sans-serif;
}

/* **ELIMINACIÓN de .app-card:** Ahora el formulario es el contenedor principal y se centra */

/* NUEVO CONTENEDOR DE FORMULARIO: Usa la estética del .form-section anterior */
.form-container {
  max-width: 32rem; /* Más compacto, max-w-lg */
  width: 100%;
  padding: 3rem 2rem; /* Más padding vertical */
  /* Fondo de sección Rosewood/Chocolate Cosmos para un look rico y compacto */
  background-color: #550816; /* Chocolate cosmos */
  border-radius: 0.5rem; /* Bordes menos redondeados (rounded-lg) */
  /* Sombra más marcada para definir el contenedor */
  box-shadow: 0 25px 50px -12px rgba(106, 4, 15, 0.4);
  border: 2px solid #6A040F; /* Borde más definido */
  color: white; /* Asegurar que el texto sea legible */
}

.main-title {
  font-size: 2rem; /* Un poco más pequeño */
  font-weight: 700;
  /* Color Rosewood */
  color: #9A0D1B; /* Carmin para el título principal */
  text-align: center;
  margin-bottom: 3rem; /* Más espacio */
}

/* --- ESTILOS DEL FORMULARIO --- */
/* Se eliminan .form-section y .form-title para simplificar la estructura */

.form-title-h2 { /* Nuevo título para secciones internas si las hubiera */
    font-size: 1.25rem; /* text-xl */
    font-weight: 600;
    color: #9A0D1B; /* Carmin */
    margin-bottom: 1.5rem;
    padding-bottom: 0.5rem;
    border-bottom: 1px solid rgba(154, 13, 27, 0.3); /* Línea divisoria sutil */
    display: flex;
    align-items: center;
    gap: 0.5rem;
}

.title-icon {
  height: 1.25rem;
  width: 1.25rem;
  color: #9A0D1B;
}

.form-grid {
    display: flex;
    flex-direction: column;
    gap: 1.25rem; /* Más compacto */
}

/* GRUPO DE FORMULARIO CON ESTILO MÁS MINIMALISTA */
.form-group {
    position: relative; /* Necesario para el estilo de input flotante */
    margin-bottom: 0;
}

.form-label {
  display: block;
  font-size: 0.75rem; /* Texto de etiqueta más pequeño */
  font-weight: 500;
  color: #c4c4c4; /* Gris claro para las etiquetas */
  margin-bottom: 0.1rem; /* Mantenemos la etiqueta cerca del input */
  position: absolute; /* Para el efecto flotante */
  top: 0.3rem;
  left: 1rem;
  transition: all 0.2s ease;
  pointer-events: none;
}

.form-input {
  width: 100%;
  padding: 1rem 1rem 0.5rem; /* Más padding en la parte superior para el texto flotante */
  background-color: #43070f; /* Fondo del input un poco más claro que el contenedor */
  border: 1px solid #6A040F; /* Borde Rosewood */
  border-radius: 0.25rem; /* Bordes muy sutiles */
  box-shadow: none; /* Quitamos la sombra de la entrada */
  transition: all 0.2s ease;
  outline: none;
  color: #ffffff; /* Texto blanco en el input */
}

.form-input:hover {
    border-color: #9A0D1B; /* Carmin en hover */
    box-shadow: none;
}

.form-input:focus {
  border-color: #9A0D1B; /* Carmin en focus */
  box-shadow: 0 0 0 1px #9A0D1B;
  background-color: #43070f;
}

/* Estilos para mover la etiqueta cuando el input está enfocado o tiene contenido */
.form-input:focus + .form-label,
.form-input:not(:placeholder-shown) + .form-label {
    top: 0.1rem;
    font-size: 0.65rem;
    color: #9A0D1B; /* Carmin para la etiqueta activa */
}


.submit-button {
  width: 100%;
  /* Gradiente entre Rosewood y Carmin */
  background-image: linear-gradient(to right, #6A040F, #9A0D1B);
  color: white;
  font-weight: 700;
  padding: 1rem 1rem; /* Un poco más alto */
  border-radius: 0.5rem; /* Bordes sutiles */
  /* Sombra Rosewood */
  box-shadow: 0 5px 15px -5px rgba(106, 4, 15, 0.6);
  transition: all 0.3s ease;
  margin-top: 2rem; /* Más margen superior */
  border: none;
  cursor: pointer;
}

.submit-button:hover {
  /* Gradiente más oscuro/profundo en hover */
  background-image: linear-gradient(to right, #550816, #6A040F);
  transform: translateY(-2px); /* Un pequeño levantamiento para el efecto */
  box-shadow: 0 8px 20px -5px rgba(106, 4, 15, 0.8);
}

.submit-button:focus {
    outline: none;
    box-shadow: 0 0 0 3px rgba(154, 13, 27, 0.7);
}
</style>