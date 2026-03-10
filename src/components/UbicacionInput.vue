<template>
  <div class="buscador-contenedor">
    <div class="buscador-input-wrapper">
      <svg class="icono-busqueda" xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="11" cy="11" r="8"></circle><line x1="21" y1="21" x2="16.65" y2="16.65"></line></svg>
      <!-- Campo de texto donde el usuario escribe la ciudad -->
      <input
        type="text"
        v-model="ciudadInput"
        @input="debouncedAutocompletar"
        @keyup.enter="seleccionarLugar"
        list="sugerencias"
        placeholder="Search city..."
        class="buscador-input"
      />
    </div>
    <!-- Lista de sugerencias para el autocompletado -->
    <datalist id="sugerencias">
      <option
        v-for="lugar in sugerencias"
        :key="lugar.id"
        :value="`${lugar.name}${lugar.admin1 ? `, ${lugar.admin1}` : ''}, ${lugar.country}`"
      ></option>
    </datalist>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { defineEmits } from 'vue';

// Variable reactiva que almacena el texto introducido en el input
const ciudadInput = ref('');
// Lista reactiva de sugerencias que se mostrarán en el datalist
const sugerencias = ref([]);
// Guarda el resultado completo de la última búsqueda en la API
let ultimaBusqueda = [];

// Define el evento que se enviará al componente padre
const emit = defineEmits(['lugar-seleccionado']);

// Funcion debounce
// Evita llamar a la API en cada pulsación de tecla.
function debounce(func, delay) {
  let timeout;
  return function (...args) {
    clearTimeout(timeout);
    timeout = setTimeout(() => func.apply(this, args), delay);
  };
}

// Función de autocompletado
async function autocompletar() {
  const texto = ciudadInput.value;
  if (texto.length < 2) return;

  const url = `https://geocoding-api.open-meteo.com/v1/search?name=${texto}&count=10&language=es`;
  const res = await fetch(url);
  const datos = await res.json();

  if (!datos.results) {
    sugerencias.value = [];
    return;
  }

  ultimaBusqueda = datos.results;
  sugerencias.value = datos.results;
}

// Espera 300ms antes de hacer la petición
const debouncedAutocompletar = debounce(autocompletar, 300);

// Selección de ciudad
function seleccionarLugar() {
  const lugar = ultimaBusqueda.find(l =>
    ciudadInput.value.toLowerCase().includes(l.name.toLowerCase())
  );

  if (!lugar) {
    alert('Selecciona una ciudad de la lista');
    return;
  }

  emit('lugar-seleccionado', lugar);
}


</script>

<style scoped>
.buscador-contenedor {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 30px;
}

.buscador-input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
  background-color: #f0f3f7;
  border-radius: 50px;
  padding: 8px 16px;
  width: 250px;
  transition: all 0.3s ease;
}

.buscador-input-wrapper:focus-within {
  background-color: var(--blanco);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
}

.icono-busqueda {
  color: var(--texto-gris);
  margin-right: 10px;
}

.buscador-input {
  background: transparent;
  width: 100%;
  border: none;
  outline: none;
  font-size: 0.9rem;
  color: var(--texto-oscuro);
}

.buscador-input::placeholder {
  color: var(--texto-gris);
}
</style>