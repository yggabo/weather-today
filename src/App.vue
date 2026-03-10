<template>
  <div>
    <h2>Mini App del Clima</h2>

    <!-- Componente de entrada de ubicación 
    Emite el evento 'lugar-seleccionado' cuando el usuario elige un lugar
    El evento se maneja con la función actualizarLugar -->
    <UbicacionInput @lugar-seleccionado="actualizarLugar" />

    <!-- Componente que muestra la información del clima actual-->
    <ClimaCard v-if="lugarSeleccionado" :lugar="lugarSeleccionado" />

    <!-- Componente que muestra el pronóstico semanal-->
    <PronosticoSemanal v-if="lugarSeleccionado" :lugar="lugarSeleccionado" />
  </div>
  <a href="https://open-meteo.com/" target="blank">Datos de open-meteo</a>
</template>

<script setup>
import { ref } from 'vue';
import UbicacionInput from './components/UbicacionInput.vue';
import ClimaCard from './components/ClimaCard.vue';
import PronosticoSemanal from './components/PronosticoSemanal.vue';

// Variable reactiva para almacenar el lugar seleccionado por el usuario
const lugarSeleccionado = ref(null);

// Función que actualiza la variable lugarSeleccionado cuando se emite el evento desde UbicacionInput
function actualizarLugar(lugar) {
  lugarSeleccionado.value = lugar;
}
</script>
