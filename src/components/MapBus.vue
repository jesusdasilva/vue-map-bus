<script setup>
import { ref, onMounted, onUnmounted,  watchEffect } from "vue";
import { GoogleMap, CustomMarker } from "vue3-google-map";

const positionBus = ref({ lat: 0, lng: 0 });
const ws = ref(null);
const isConnected = ref(false);

// Conectar con el servidor de WebSockets
const connect = () => {
  // Crear conexión
  ws.value = new WebSocket('wss://masgps.witservices.io/api-v2/event/subscription');

  // Escuchar eventos de la conexión
  ws.value.onopen = (event) => {
    isConnected.value = true;
    ws.value.send('{"action":"subscribe","hash":"7d52e8d9ab605b78c839b90c47263d81","events":["state"],"trackers":[10180276]}');
  };

  // Escuchar eventos de mensajes recibidos
  ws.value.onmessage = (event) => {
    
    // Convertir mensaje a objeto
    const tramaGps = JSON.parse(event.data);

    // Actualizar posición del bus si el mensaje contiene una posición
    if(tramaGps.data.hasOwnProperty('gps')){
      positionBus.value = tramaGps.data.gps.location;
      // Comenta esto si no quieres ver la posición del bus en la consola
      console.log(positionBus.value);
    } 

  };

  // Escuchar eventos de errores
  ws.value.onerror = (event) => {
    console.error('Error', event);
    isConnected.value = false;
  };

  // Escuchar eventos de conexión cerrada
  ws.value.onclose = (event) => {
    isConnected.value = false;
  };
};

// Conectar cuando se monte el componente
onMounted(connect);

// Reconectar cuando se pierda la conexión
watchEffect(() => {
  if (!isConnected.value) {
    setTimeout(connect, 5000); // Reconectar después de 5 segundos
  }
});

// Desconectar cuando se desmonte el componente
onUnmounted(() => {
  if (ws.value) {
    ws.value.close();
  }
});


</script>

<template>
  <GoogleMap
    api-key="AIzaSyDHNeCjHnNbwyLMkPTIsGEBJQ_1AjxMCz8"
    style="width: 100%; height: 500px"
    :center="positionBus"
    :zoom="15"
  >
    <CustomMarker :options="{ position: positionBus, anchorPoint: 'BOTTOM_CENTER' }">
      <div style="width: 30px; height: 30px;">
        <span style="background-color: aqua;">10180276</span>
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 576 512"><!--! Font Awesome Pro 6.4.0 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license (Commercial License) Copyright 2023 Fonticons, Inc. --><path d="M288 0C422.4 0 512 35.2 512 80V96l0 32c17.7 0 32 14.3 32 32v64c0 17.7-14.3 32-32 32l0 160c0 17.7-14.3 32-32 32v32c0 17.7-14.3 32-32 32H416c-17.7 0-32-14.3-32-32V448H192v32c0 17.7-14.3 32-32 32H128c-17.7 0-32-14.3-32-32l0-32c-17.7 0-32-14.3-32-32l0-160c-17.7 0-32-14.3-32-32V160c0-17.7 14.3-32 32-32h0V96h0V80C64 35.2 153.6 0 288 0zM128 160v96c0 17.7 14.3 32 32 32H272V128H160c-17.7 0-32 14.3-32 32zM304 288H416c17.7 0 32-14.3 32-32V160c0-17.7-14.3-32-32-32H304V288zM144 400a32 32 0 1 0 0-64 32 32 0 1 0 0 64zm288 0a32 32 0 1 0 0-64 32 32 0 1 0 0 64zM384 80c0-8.8-7.2-16-16-16H208c-8.8 0-16 7.2-16 16s7.2 16 16 16H368c8.8 0 16-7.2 16-16z"/></svg>
      </div>
    </CustomMarker>

  </GoogleMap>

</template>

<style scoped></style>
