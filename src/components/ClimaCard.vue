<template>
  <div class="clima-principal" v-if="clima">
    <div class="info-cabecera">
      <h1 class="ciudad">{{ lugar.name }}{{ lugar.country ? `, ${lugar.country}` : '' }}</h1>
    </div>

    <div class="tarjeta-principal">
      <div class="cuerpo-tarjeta">
        <div class="temperatura-seccion">
          <span class="numero-temperatura">{{ Math.round(clima.temperatura) }}</span>
          <span class="unidad-celcius">°C</span>
        </div>
        <div class="icono-clima-grande">
          <!-- Icono de sol simplificado -->
          <svg xmlns="http://www.w3.org/2000/svg" width="80" height="80" viewBox="0 0 24 24" fill="none" stroke="#ffcc00" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="5"></circle><line x1="12" y1="1" x2="12" y2="3"></line><line x1="12" y1="21" x2="12" y2="23"></line><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line><line x1="1" y1="12" x2="3" y2="12"></line><line x1="21" y1="12" x2="23" y2="12"></line><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line></svg>
        </div>
      </div>
      
      <div class="estado-clima-badge">
        <span>MAINLY SUNNY</span>
      </div>

      <div class="estadisticas-grid">
        <div class="stat-item">
          <svg class="stat-icon" xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="#007bff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 2.69l5.66 5.66a8 8 0 1 1-11.31 0z"></path></svg>
          <span class="stat-label">PRECIPITATION</span>
          <span class="stat-valor">{{ clima.probabilidad }}%</span>
        </div>
        <div class="stat-item">
          <svg class="stat-icon" xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="#007bff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M9.59 4.59A2 2 0 1 1 11 8H2m10.59 11.41A2 2 0 1 0 14 16H2m15.73-8.27A2.5 2.5 0 1 1 19.5 12H2"></path></svg>
          <span class="stat-label">WIND</span>
          <span class="stat-valor">{{ clima.viento }} km/h</span>
        </div>
        <div class="stat-item">
          <svg class="stat-icon" xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="#007bff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 22a7 7 0 0 0 7-7c0-2-1-3.9-3-5.5s-3.5-4-4-6.5c-.5 2.5-2 4.9-4 6.5s-3 3.5-3 5.5a7 7 0 0 0 7 7z"></path></svg>
          <span class="stat-label">HUMIDITY</span>
          <span class="stat-valor">{{ clima.humedad }}%</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue';
import { defineProps } from 'vue';

// Props recibidos desde el componente padre
// Se espera un objeto 'lugar' con latitud y longitud
const props = defineProps({
  lugar: Object
});

// Función para obtener la fecha y hora actual formateada
function obtenerFechaFormateada() {
  const ahora = new Date();
  return ahora.toLocaleDateString('en-US', { 
    weekday: 'long', 
    month: 'short', 
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit',
    hour12: true 
  }).replace(',', '');
}

// Variable reactiva que almacenará los datos del clima actual
const clima = ref(null);

// Función para obtener clima actual
async function obtenerClima(lat, lon) {
  const url = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current=temperature_2m,relative_humidity_2m,precipitation,precipitation_probability,windspeed_10m&temperature_unit=celsius&windspeed_unit=kmh&precipitation_unit=mm`;

  const res = await fetch(url);
  const datos = await res.json();

  return {
    temperatura: datos.current.temperature_2m,
    precipitacion: datos.current.precipitation,
    probabilidad: datos.current.precipitation_probability,
    viento: datos.current.windspeed_10m,
    humedad: datos.current.relative_humidity_2m,
  };
}

// Watcher para props.lugar
watch(
  () => props.lugar,
  async (nuevoLugar) => {
    if (nuevoLugar) {
      clima.value = await obtenerClima(nuevoLugar.latitude, nuevoLugar.longitude);
    }
  },
  { immediate: true }
);
</script>

<style scoped>
.clima-principal {
  margin-top: 20px;
}

.info-cabecera {
  margin-bottom: 30px;
}

.ciudad {
  font-size: 2.8rem;
  color: var(--texto-oscuro);
  letter-spacing: -1px;
  line-height: 1;
}

.fecha-hoy {
  color: var(--texto-gris);
  font-weight: 600;
  margin-top: 8px;
  font-size: 1.1rem;
}

.tarjeta-principal {
  background-color: var(--blanco);
  border-radius: var(--radio);
  padding: 40px;
  box-shadow: var(--sombra);
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
  overflow: hidden;
  border: 1px solid rgba(0, 0, 0, 0.03);
}

.cuerpo-tarjeta {
  display: flex;
  align-items: center;
  gap: 30px;
  margin-bottom: 20px;
}

.temperatura-seccion {
  display: flex;
  align-items: flex-start;
}

.numero-temperatura {
  font-size: 8rem;
  font-weight: 800;
  line-height: 1;
  color: #1a1a1a;
}

.unidad-celcius {
  font-size: 3rem;
  font-weight: 700;
  color: var(--azul-primario);
  margin-top: 15px;
}

.estado-clima-badge {
  background-color: var(--azul-suave);
  color: var(--azul-primario);
  padding: 8px 24px;
  border-radius: 50px;
  font-weight: 800;
  font-size: 0.85rem;
  letter-spacing: 0.5px;
  margin-bottom: 50px;
}

.estadisticas-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  width: 100%;
  gap: 20px;
  border-top: 1px solid #f0f3f7;
  padding-top: 30px;
}

.stat-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
}

.stat-icon {
  margin-bottom: 5px;
}

.stat-label {
  font-size: 0.65rem;
  font-weight: 700;
  color: var(--texto-gris);
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.stat-valor {
  font-size: 1.1rem;
  font-weight: 700;
  color: var(--texto-oscuro);
}
</style>