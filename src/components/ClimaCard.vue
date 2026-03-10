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
  <span style="font-size: 5rem">{{ getWeatherDisplay(clima.weathercode).icon }}</span>
</div>


      </div>
      
     <div class="estado-clima-badge" >
  <span>{{ getWeatherDisplay(clima.weathercode).texto }}</span>
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
import { ref, watch } from 'vue';
import { defineProps } from 'vue';

const props = defineProps({
  lugar: Object
});

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

const clima = ref(null);

async function obtenerClima(lat, lon) {
  // URL de la API con datos horarios
  const url = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&hourly=temperature_2m,relative_humidity_2m,precipitation_probability,windspeed_10m,weathercode&temperature_unit=celsius&windspeed_unit=kmh&precipitation_unit=mm`;

  const res = await fetch(url);
  const datos = await res.json();

  // Tomamos la hora actual en formato ISO: YYYY-MM-DDTHH:00
  const ahora = new Date();
  const ahoraISO = ahora.toISOString().slice(0, 13) + ":00";

  // Buscamos el índice del tiempo actual en los datos horarios
  const horaIndex = datos.hourly.time.findIndex(t => t === ahoraISO);

  if (horaIndex === -1) {
    console.warn("No se encontró la hora actual en los datos de Open-Meteo");
    return null;
  }

  return {
    temperatura: datos.hourly.temperature_2m[horaIndex],
    precipitacion: datos.hourly.precipitation_probability[horaIndex], //error
    probabilidad: datos.hourly.precipitation_probability[horaIndex],
    viento: datos.hourly.windspeed_10m[horaIndex],
    humedad: datos.hourly.relative_humidity_2m[horaIndex],
    weathercode: datos.hourly.weathercode[horaIndex]
  };
}

const weatherMap = {
  0: { texto: 'CLEAR SKY', color: '#ffcc00', icon: '☀️' },
  1: { texto: 'MAINLY CLEAR', color: '#ffe066', icon: '🌤️' },
  2: { texto: 'PARTLY CLOUDY', color: '#d9d9d9', icon: '⛅' },
  3: { texto: 'OVERCAST', color: '#a0a0a0', icon: '☁️' },
  45: { texto: 'FOG', color: '#cfcfcf', icon: '🌫️' },
  48: { texto: 'RIME FOG', color: '#d0d0d0', icon: '🌫️' },
  51: { texto: 'DRIZZLE LIGHT', color: '#a0c4ff', icon: '🌦️' },
  53: { texto: 'DRIZZLE MODERATE', color: '#70a1ff', icon: '🌦️' },
  55: { texto: 'DRIZZLE DENSE', color: '#3366ff', icon: '🌧️' },
  61: { texto: 'RAIN SLIGHT', color: '#3399ff', icon: '🌧️' },
  63: { texto: 'RAIN MODERATE', color: '#0066ff', icon: '🌧️' },
  65: { texto: 'RAIN HEAVY', color: '#0033cc', icon: '🌧️' },
  71: { texto: 'SNOW SLIGHT', color: '#e6f7ff', icon: '🌨️' },
  73: { texto: 'SNOW MODERATE', color: '#b3e5fc', icon: '🌨️' },
  75: { texto: 'SNOW HEAVY', color: '#80d4ff', icon: '❄️' },
  80: { texto: 'SHOWERS SLIGHT', color: '#66ccff', icon: '🌦️' },
  81: { texto: 'SHOWERS MODERATE', color: '#3399ff', icon: '🌦️' },
  82: { texto: 'SHOWERS HEAVY', color: '#0066cc', icon: '🌧️' },
  95: { texto: 'THUNDERSTORM', color: '#ff9900', icon: '⛈️' },
  99: { texto: 'THUNDERSTORM HAIL', color: '#cc6600', icon: '⛈️' },
};

function getWeatherDisplay(code) {
  return weatherMap[code] || { texto: 'UNKNOWN', color: '#999999', icon: '❔' };
}

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