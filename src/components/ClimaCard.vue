<template>
   <!-- Tarjeta de clima actual -->
  <div class="clima-card" v-if="clima">
    <p>{{ lugar.name }}{{ lugar.admin1 ? `, ${lugar.admin1}` : '' }}:</p>
    <p>Temperatura: {{ clima.temperatura }} °C</p>
    <p>Precipitación: {{ clima.precipitacion }} mm</p>
    <p>Probabilidad de lluvia: {{ clima.probabilidad }} %</p>
    <p>Viento: {{ clima.viento }} km/h</p>
    <p>Humedad: {{ clima.humedad }} %</p>
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

// Variable reactiva que almacenará los datos del clima actual
const clima = ref(null);

// Función para obtener clima actual
async function obtenerClima(lat, lon) {
  const url = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current=temperature_2m,relativehumidity_2m,precipitation,precipitation_probability,windspeed_10m&temperature_unit=celsius&windspeed_unit=kmh&precipitation_unit=mm`;

  const res = await fetch(url);
  const datos = await res.json();

  return {
    temperatura: datos.current.temperature_2m,
    precipitacion: datos.current.precipitation,
    probabilidad: datos.current.precipitation_probability,
    viento: datos.current.windspeed_10m,
    humedad: datos.current.relativehumidity_2m
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
.clima-card {
  border: 1px solid #ccc;
  padding: 15px;
  border-radius: 10px;
  width: 280px;
  margin-top: 10px;
}

p {
  margin: 5px 0;
}
</style>