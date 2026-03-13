# Weather Today! 

Una aplicación web moderna y elegante para consultar el clima en tiempo real de cualquier ciudad del mundo.

## Características

- **Búsqueda Inteligente**: Autocompletado de ciudades utilizando la API de Geocodificación de Open-Meteo.
- **Clima Actual Detallado**: Visualización premium de la temperatura, probabilidad de precipitación, velocidad del viento y humedad.
- **Pronóstico Semanal**: Vista de los próximos 7 días con un diseño limpio y adaptativo.
- **Interfaz Premium**: Diseño inspirado en interfaces modernas, con tipografía cuidada (Inter), sombras suaves y una paleta de colores equilibrada.
- **Componentización**: Arquitectura basada en componentes de Vue 3 (SFC) para una mejor organización y mantenibilidad.

## Tecnologías

- **Vue 3**: Framework progresivo de JavaScript.
- **Vite**: Herramienta de compilación rápida para el desarrollo.
- **CSS Vanilla**: Estilos personalizados sin dependencias externas pesadas.
- **Open-Meteo API**: Datos meteorológicos gratuitos y precisos sin necesidad de clave API.

## Estructura del Proyecto

- `src/App.vue`: Contenedor principal y gestión del estado de la ubicación.
- `src/components/UbicacionInput.vue`: Componente de búsqueda y autocompletado.
- `src/components/ClimaCard.vue`: Visualización del clima actual.
- `src/components/PronosticoSemanal.vue`: Listado del pronóstico para la semana.
- `src/assets/main.css`: Sistema de diseño global y variables.

## Instalación y Ejecución

1. Instalar dependencias:
   ```bash
   npm install
   ```

2. Iniciar el servidor de desarrollo:
   ```bash
   npm run dev
   ```

3. Abrir en el navegador:
   `http://localhost:5173` (o el puerto indicado en la terminal)

---
*Desarrollado con enfoque en estética y funcionalidad.*
