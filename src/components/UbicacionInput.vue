<template>
  <div>
    <!-- Campo de texto donde el usuario escribe la ciudad -->
    <input
      type="text"
      v-model="ciudadInput"
      @input="debouncedAutocompletar"
      list="sugerencias"
      placeholder="Escribe una ciudad"
    />
    <!-- Lista de sugerencias para el autocompletado -->
    <datalist id="sugerencias">
      <option
        v-for="lugar in sugerencias"
        :key="lugar.id"
        :value="`${lugar.name}${lugar.admin1 ? `, ${lugar.admin1}` : ''}, ${lugar.country}`"
      ></option>
    </datalist>
    <!-- Botón que confirma la selección -->
    <button @click="seleccionarLugar">Buscar</button>
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

  const url = `https://geocoding-api.open-meteo.com/v1/search?name=${texto}&count=5`;
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
  const lugar = ultimaBusqueda.find(l => {
    const admin = l.admin1 ? `, ${l.admin1}` : '';
    return `${l.name}${admin}, ${l.country}` === ciudadInput.value;
  });

  if (!lugar) {
    alert('Selecciona una ciudad de la lista');
    return;
  }

  // Emitir el lugar seleccionado al componente padre
  emit('lugar-seleccionado', lugar);
}
</script>