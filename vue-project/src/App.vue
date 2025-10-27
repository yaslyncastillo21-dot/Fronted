<template>
  <div class="main-container">
    <div class="app-card">
      <h1 class="main-title">
        Registra tus Clientes
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
            <div class="input-wrapper">
              <input
                id="input-nombre"
                v-model="nuevoCliente.nombre"
                type="text"
                required
                placeholder=" "
                class="form-input"
              />
              <label for="input-nombre" class="form-label">Nombre Completo:</label>
            </div>
          </div>
          
          <div class="split-group">
            <div class="form-group">
              <div class="input-wrapper">
                <input
                  id="input-telefono"
                  v-model="nuevoCliente.telefono"
                  type="tel"
                  required
                  placeholder=" "
                  class="form-input"
                />
                <label for="input-telefono" class="form-label">Teléfono:</label>
              </div>
            </div>
            
            <div class="form-group">
              <div class="input-wrapper">
                <input
                  id="input-direccion"
                  v-model="nuevoCliente.direccion"
                  type="text"
                  required
                  placeholder=" "
                  class="form-input"
                />
                <label for="input-direccion" class="form-label">Dirección:</label>
              </div>
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
import ClienteList from './components/ClienteList.vue'; 

interface Cliente {
  id_cliente: number;
  nombre: string;
  telefono: string;
  direccion: string;
}

const API_URL = 'http://localhost:2000/api/clientes';

const nuevoCliente = ref({ nombre: '', telefono: '', direccion: '' }); 

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

const cargarClientes = async () => {
  try {
    const res = await fetch(API_URL);
    if (!res.ok) {
      throw new Error(`Error HTTP: ${res.status}`);
    }
    clientes.value = await res.json(); 
  } catch (error: any) {
    console.error('Error al cargar clientes:', error);
    clientes.value = [];
    if (!clientes.value.length) {
      mostrarModal('❌ Error de Conexión', `No se pudo conectar con el servidor. Asegúrate de que el backend (http://localhost:2000) esté corriendo y use el endpoint '/api/clientes'.`, 'error');
    }
  }
};

const agregarCliente = async () => {
  try {
    const dataToSend = nuevoCliente.value; 

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

    nuevoCliente.value = { nombre: '', telefono: '', direccion: '' }; 
    mostrarModal('✅ Éxito', 'El cliente fue agregado correctamente.', 'success');
    await cargarClientes(); 
  } catch (error: any) {
    console.error('Error al agregar cliente:', error);
    mostrarModal('❌ Error al Registrar', error.message || 'Ocurrió un error inesperado al intentar registrar el cliente.', 'error'); 
  }
};

onMounted(cargarClientes); 
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap');

/* PALETA DE COLORES */
/* Primario Oscuro: #6A040F (Rosewood) */
/* Primario Claro: #9A0D1B (Carmin) */
/* Fondo de Sección: #550816 (Chocolate cosmos) */
/* Fondo Principal: #FAEAEB (Rosa muy claro) */
/* Botón/Éxito: #1a73e8 (un azul vibrante, o puedes usar un verde oscuro/rojo más apagado si prefieres) */


/* ESTILOS BASE */
.main-container {
  min-height: 100vh;
  background-color: #FAEAEB; /* Rosa muy claro */
  display: flex;
  align-items: flex-start; /* Alinea arriba para dar espacio al título */
  justify-content: center;
  padding: 3rem 1rem; /* Más padding para el contenedor principal */
  font-family: 'Inter', sans-serif;
}

.app-card {
  background-color: #550816; /* Chocolate cosmos para el fondo principal de la tarjeta */
  border-radius: 1rem; /* Bordes más redondeados */
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2); /* Sombra suave pero visible */
  padding: 2.5rem 2rem; /* Espaciado interno generoso */
  max-width: 600px; /* Ancho máximo para el formulario */
  width: 100%;
  color: #E0E0E0; /* Texto claro general */
  display: flex;
  flex-direction: column;
  gap: 2rem; /* Espacio entre secciones */
  border: 1px solid #6A040F; /* Borde sutil del color primario oscuro */
}

.main-title {
  font-size: 2.5rem; /* Título principal grande */
  font-weight: 800; /* Extra bold */
  color: #33060a; /* Carmín para el título principal */
  text-align: center;
  margin-bottom: 1.5rem; /* Espacio debajo del título */
  line-height: 1.2;
}

/* --- ESTILOS DEL FORMULARIO --- */
.form-section {
  background-color: #6A040F; /* Rosewood para el fondo del formulario, resalta */
  padding: 2rem;
  border-radius: 0.75rem;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.form-title {
  font-size: 1.6rem; /* Título del formulario más grande */
  font-weight: 700;
  color: #33060a; /* Rosa muy claro para el título del formulario */
  display: flex;
  align-items: center;
  gap: 0.8rem;
  margin-bottom: 1.5rem;
  padding-bottom: 0.8rem;
  border-bottom: 1px solid rgba(250, 234, 235, 0.2); /* Línea divisoria más clara */
}

.title-icon {
  height: 1.8rem; /* Icono más grande */
  width: 1.8rem;
  color: #33060a; /* Rosa muy claro para el icono */
}

.form-grid {
  display: flex;
  flex-direction: column;
  gap: 1.5rem; /* Espaciado entre grupos de formulario */
}

.split-group {
  display: flex;
  gap: 1.5rem;
  flex-wrap: wrap; /* Permite que los campos se apilen en pantallas pequeñas */
}

.split-group .form-group {
  flex: 1; /* Distribuye el espacio equitativamente */
  min-width: 150px; /* Evita que los campos se hagan demasiado pequeños */
}

/* GRUPO DE FORMULARIO CON ESTILO FLOTANTE */
.form-group {
  position: relative;
  margin-bottom: 0; /* Ya no necesitamos margen inferior aquí */
}

.input-wrapper {
  position: relative;
  width: 100%;
}

.form-label {
  position: absolute;
  top: 1rem; /* Posición inicial de la etiqueta */
  left: 1rem;
  font-size: 1rem;
  color: #E0E0E0; /* Color inicial de la etiqueta */
  pointer-events: none;
  transition: all 0.2s ease;
  transform-origin: left top;
}

.form-input {
  width: 100%;
  padding: 1.5rem 1rem 0.75rem; /* Más padding para dejar espacio a la etiqueta flotante */
  background-color: #550816; /* Chocolate cosmos para los inputs */
  border: 1px solid #9A0D1B; /* Borde carmín */
  border-radius: 0.5rem;
  color: #FFFFFF; /* Texto blanco */
  font-size: 1rem;
  transition: all 0.2s ease;
  outline: none;
}

.form-input::placeholder {
  color: transparent; /* Ocultamos el placeholder, la etiqueta lo reemplaza */
}

/* Estilos para mover la etiqueta cuando el input está enfocado o tiene contenido */
.form-input:focus + .form-label,
.form-input:not(:placeholder-shown) + .form-label {
  top: 0.5rem; /* Posición más alta cuando está activo */
  font-size: 0.75rem; /* Más pequeño */
  color: #FAEAEB; /* Rosa muy claro cuando está activo */
  transform: translateY(-5px); /* Pequeño ajuste para evitar que tape el borde */
}

.form-input:hover {
  border-color: #FAEAEB; /* Rosa muy claro en hover */
}

.form-input:focus {
  border-color: #FAEAEB; /* Rosa muy claro en focus */
  box-shadow: 0 0 0 2px rgba(250, 234, 235, 0.5); /* Sombra de focus sutil */
  background-color: #43070f; /* Ligeramente más oscuro en focus */
}

.submit-button {
  width: 100%;
  background-image: linear-gradient(to right, #9A0D1B, #6A040F); /* Gradiente de Carmín a Rosewood */
  color: rgb(57, 9, 9);
  font-weight: 700;
  padding: 1.2rem 1.5rem;
  border-radius: 0.75rem;
  box-shadow: 0 8px 20px -8px rgba(154, 13, 27, 0.7); /* Sombra más pronunciada */
  transition: all 0.3s ease;
  margin-top: 1.5rem;
  border: none;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.8rem;
  font-size: 1.1rem;
}

.submit-button:hover {
  background-image: linear-gradient(to right, #6A040F, #9A0D1B); /* Invertir gradiente o hacerlo más oscuro */
  transform: translateY(-3px);
  box-shadow: 0 12px 25px -10px rgba(154, 13, 27, 0.9);
}

.submit-button:focus {
  outline: none;
  box-shadow: 0 0 0 4px rgba(250, 234, 235, 0.6); /* Anillo de focus más notorio */
}

.button-icon {
  height: 1.5rem;
  width: 1.5rem;
}

/* --- ESTILOS DEL MODAL (Ajuste de colores) --- */
.modal-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.6); /* Fondo semi-transparente oscuro */
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background-color: #550816; /* Chocolate cosmos para el fondo del modal */
  border-radius: 1rem;
  padding: 2.5rem;
  max-width: 400px;
  width: 90%;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
  text-align: center;
  color: #E0E0E0;
  border: 1px solid #6A040F;
}

.modal-icon {
  font-size: 3rem;
  margin-bottom: 1.5rem;
}

.modal-title-text {
  font-size: 1.5rem;
  font-weight: 700;
  margin-bottom: 0.8rem;
  color: #FAEAEB; /* Rosa muy claro para el título del modal */
}

.modal-message {
  font-size: 1rem;
  margin-bottom: 2rem;
  color: #C4C4C4;
}

.modal-button {
  padding: 0.8rem 2rem;
  border-radius: 0.5rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
  border: none;
  color: white;
}

.modal-button-success {
  background-color: #6A040F; /* Rosewood para éxito */
  box-shadow: 0 4px 10px rgba(106, 4, 15, 0.4);
}

.modal-button-success:hover {
  background-color: #9A0D1B; /* Carmín en hover */
  transform: translateY(-1px);
}

.modal-button-error {
  background-color: #9A0D1B; /* Carmín para error */
  box-shadow: 0 4px 10px rgba(154, 13, 27, 0.4);
}

.modal-button-error:hover {
  background-color: #6A040F; /* Rosewood en hover */
  transform: translateY(-1px);
}

</style>