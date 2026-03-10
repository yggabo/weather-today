<template>
  <div class="pronostico-seccion" v-if="pronostico.length">
    <div class="pronostico-header">
      <h2 class="pronostico-titulo">Weekly Forecast</h2>
      <button class="btn-ver-mas">View 7 days</button>
    </div>
    
    <div class="pronostico-lista">
      <!-- Tarjeta de cada día -->
      <div 
        class="pronostico-item" 
        v-for="(dia, index) in pronostico" 
        :key="index"
        :class="{ 'item-activo': index === 0 }"
      >
        <span class="dia-nombre">{{ index === 0 ? 'NOW' : formatearDia(dia.fecha) }}</span>
        
        <div class="icono-clima-pequeno">
          {{ obtenerIcono(dia.weathercode) }}
        </div>
        <div class="temp-container">
  <span class="temp-max">{{ Math.round(dia.tempMax) }}°</span>
  <span class="temp-min">{{ Math.round(dia.tempMin) }}°</span>
</div>
        <span class="dia-precipitacion">{{ dia.precipitacion.toFixed(1) }} mm</span>
      </div>
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
  const url = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&daily=weathercode,temperature_2m_max,temperature_2m_min,precipitation_sum&temperature_unit=celsius&precipitation_unit=mm&timezone=auto&forecast_days=7`;
  const res = await fetch(url);
  const datos = await res.json();

  const dias = datos.daily.time.map((fecha, i) => ({
    fecha,
    tempMax: datos.daily.temperature_2m_max[i],
    tempMin: datos.daily.temperature_2m_min[i],
    precipitacion: datos.daily.precipitation_sum[i] ?? 0,
    weathercode: datos.daily.weathercode[i]
  }));

  pronostico.value = dias;
}

function obtenerIcono(weathercode) {
  if (weathercode === 0) return '☀️';
  if ([1,2].includes(weathercode)) return '🌤️';
  if (weathercode === 3) return '☁️';
  if ([45,48].includes(weathercode)) return '🌫️';
  if ([51,53,55,61,63,65,80,81,82].includes(weathercode)) return '🌧️';
  if ([71,73,75,77].includes(weathercode)) return '❄️';
  if ([95,96,99].includes(weathercode)) return '⛈️';

  return '☀️';
}

// Función para formatear la fecha
function formatearFecha(fechaStr) {
  const fecha = new Date(fechaStr + 'T00:00:00');
  return fecha.toLocaleDateString('es-ES', { weekday: 'short', day: 'numeric', month: 'short' });
}

// Función para obtener solo el día corto o la hora (para el diseño)
function formatearDia(fechaStr) {
  const fecha = new Date(fechaStr + 'T00:00:00');
  return fecha.toLocaleDateString('en-US', { weekday: 'short' }).toUpperCase();
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
.pronostico-seccion {
  margin-top: 40px;
}

.pronostico-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.pronostico-titulo {
  font-size: 1.2rem;
  font-weight: 700;
  color: var(--texto-oscuro);
}

.btn-ver-mas {
  color: var(--azul-primario);
  font-weight: 700;
  font-size: 0.85rem;
}

.pronostico-lista {
  display: flex;
  justify-content: space-between;
  gap: 12px;
  overflow-x: auto;
  padding-bottom: 10px;
}

/* Ocultar scrollbar */
.pronostico-lista::-webkit-scrollbar {
  display: none;
}

.pronostico-item {
  background-color: var(--blanco);
  border-radius: 15px;
  padding: 20px 15px;
  min-width: 100px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
  gap: 15px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.02);
  border: 1px solid rgba(0, 0, 0, 0.03);
  transition: all 0.3s ease;
}

.item-activo {
  background-color: var(--azul-primario);
  color: var(--blanco);
  box-shadow: 0 10px 20px rgba(0, 123, 255, 0.2);
  border-color: var(--azul-primario);
}

.dia-nombre {
  font-size: 0.75rem;
  font-weight: 700;
  color: var(--texto-gris);
}

.item-activo .dia-nombre {
  color: rgba(255, 255, 255, 0.8);
}

.icono-clima-pequeno {
  display: flex;
  align-items: center;
  justify-content: center;
}

.temp-container{
  display: flex;
  gap: 6px;
  align-items: center;
}

.temp-max{
  font-size: 1.1rem;
  font-weight: 800;
}

.temp-min{
  font-size: 0.8rem;
  font-weight: 600;
}

</style>
