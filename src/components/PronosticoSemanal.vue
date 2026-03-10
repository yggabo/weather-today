<template>
  <!-- Contenedor principal del pronóstico -->
  <div class="pronostico-container" v-if="pronostico.length">
    <!-- Tarjeta de cada día del pronóstico -->
    <div class="pronostico-card" v-for="(dia, index) in pronostico" :key="index">
      <p class="fecha">{{ formatearFecha(dia.fecha) }}</p>
      <p>Max: {{ dia.tempMax }}°C</p>
      <p>Min: {{ dia.tempMin }}°C</p>
      <p>Precip: {{ dia.precipitacion }} mm</p>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue';
import { defineProps } from 'vue';

// Props que recibe el componente desde el padre
// Se espera un objeto 'lugar' con latitud y longitud
const props = defineProps({
  lugar: Object
});

// Variable reactiva que guarda el pronóstico de 7 días
const pronostico = ref([]);

// Función para obtener pronóstico
async function obtenerPronostico(lat, lon) {
  const url = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&daily=temperature_2m_max,temperature_2m_min,precipitation_sum&temperature_unit=celsius&precipitation_unit=mm&timezone=auto&forecast_days=7`;
  const res = await fetch(url);
  const datos = await res.json();

  const dias = datos.daily.time.map((fecha, i) => ({
    fecha,
    tempMax: datos.daily.temperature_2m_max[i],
    tempMin: datos.daily.temperature_2m_min[i],
    precipitacion: datos.daily.precipitation_sum[i] ?? 0
  }));

  pronostico.value = dias;
}

// Función para formatear la fecha
function formatearFecha(fechaStr) {
  const fecha = new Date(fechaStr + 'T00:00:00');
  return fecha.toLocaleDateString('es-ES', { weekday: 'short', day: 'numeric', month: 'short' });
}

// Watcher para props.lugar
watch(
  () => props.lugar,
  async (nuevoLugar) => {
    if (nuevoLugar) {
      await obtenerPronostico(nuevoLugar.latitude, nuevoLugar.longitude);
    }
  },
  { immediate: true }
);
</script>

<style scoped>
.pronostico-container {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: 15px;
}

.pronostico-card {
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 10px;
  width: 130px;
  text-align: center;
  font-size: 14px;
}

.fecha {
  font-weight: bold;
  margin-bottom: 5px;
}

p {
  margin: 3px 0;
}
</style>
