<template>
  <main class="contenedor-principal">
    <!-- Componente de búsqueda (Buscador superior) -->
    <UbicacionInput @lugar-seleccionado="actualizarLugar" />

    <div v-if="lugarSeleccionado" class="contenido-clima">
      <!-- Componente que muestra la información del clima actual -->
      <ClimaCard :lugar="lugarSeleccionado" />

      <!-- Componente que muestra el pronóstico semanal -->
      <PronosticoSemanal :lugar="lugarSeleccionado" />
    </div>

    <!-- Mensaje inicial si no hay lugar seleccionado -->
    <div v-else class="mensaje-bienvenida">
      <h2>Welcome to Weather Today!</h2>
      <p>Search for a city to start</p>
    </div>
    
    <footer class="footer">
      <a href="https://open-meteo.com/" target="blank" class="link-datos">Data provided by Open-Meteo</a>
    </footer>
  </main>
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

<style scoped>
.mensaje-bienvenida {
  text-align: center;
  margin-top: 100px;
  color: var(--texto-gris);
}

.mensaje-bienvenida h2 {
  font-size: 2rem;
  color: var(--texto-oscuro);
  margin-bottom: 10px;
}

.footer {
  margin-top: 60px;
  text-align: center;
  padding: 20px;
}

.link-datos {
  text-decoration: none;
  color: var(--texto-gris);
  font-size: 0.8rem;
  font-weight: 600;
  transition: color 0.3s;
}

.link-datos:hover {
  color: var(--azul-primario);
}
</style>
