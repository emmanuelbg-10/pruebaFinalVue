<script setup>
import { ref, watch, onMounted } from 'vue';
import Componente from './components/Componente.vue';
import MapDestination from './components/MapDestination.vue';

let budget = ref(0);
let price = ref(0);
let numPersons = ref(1);
let numDays = ref(1);
const accommodationTypes = ref(["Albergue", "Hostal", "Bed and Breakfast", "Hotel 3*", "Hotel Superior"]);
let destino = ref('Hostal'); // Valor inicial
let sitio = ref('');
const costsDestinations = {
  cheap: 0.7,
  moderate: 1,
  expensive: 1.3
};
const costsAccommodations = {
  "Albergue": 25,
  "Hostal": 40,
  "Bed and Breakfast": 65,
  "Hotel 3*": 100,
  "Hotel Superior": 200,
};

const array = ref([]);
const coordenadas = ref('');
let economicLevel = ref('moderate'); // Default economic level

const calculatePrice = () => {
  if (!numDays.value || !numPersons.value || !destino.value || !economicLevel.value) {
    price.value = 0;
    return;
  }
  const accommodationCost = costsAccommodations[destino.value];
  const destinationCostMultiplier = costsDestinations[economicLevel.value];
  price.value = (numDays.value * numPersons.value * accommodationCost * destinationCostMultiplier).toFixed(2);
};

watch([numDays, numPersons, destino, economicLevel], calculatePrice);

watch(numPersons, (newValue) => {
  newValue = Math.floor(newValue);
  if (newValue < 1) {
    numPersons.value = 1;
  } else if (newValue > 10) {
    numPersons.value = 10;
  } else {
    numPersons.value = newValue;
  }
});

watch(numDays, (newValue) => {
  newValue = Math.floor(newValue);
  if (newValue < 1) {
    numDays.value = 1;
  } else if (newValue > 30) {
    numDays.value = 30;
  } else {
    numDays.value = newValue;
  }
});

onMounted(() => {
  fetch("http://localhost:3000/api/destinations")
    .then(response => response.json())
    .then(data => {
      array.value = data.sort((a, b) => a.name.localeCompare(b.name));
      console.log(array.value);
    })
    .catch(error => console.log('error', error));
});

function defineSitio(item) {
  sitio.value = item.name;
  coordenadas.value = item.coordinates;
  economicLevel.value = item.economic_level; // Set the economic level based on the selected destination
  calculatePrice();
}
</script>

<template>
  <h1>Destinos turísticos</h1>
  <p>{{ numPersons }} personas durante {{ numDays }} días <span v-if="sitio !== ''">en {{ sitio }}</span> alojándose en {{ destino }}</p>
  <h2 v-if="price !== 0">Se estima {{ price }}€</h2>
  <Componente v-model:budget="budget" :price="price" />
  <p>Numero de personas: <input type="number" v-model="numPersons" min="1" max="10"> <input type="range" v-model="numPersons" min="1" max="10"></p>
  <p>Numero de días: <input type="number" v-model="numDays" min="1" max="30"> <input type="range" v-model="numDays" min="1" max="30"></p>
  <p>
    Tipo de alojamiento:
    <select v-model="destino">
      <option v-for="type in accommodationTypes" :key="type">{{ type }}</option>
    </select>
  </p>
  <div v-if="coordenadas !== ''">
    <MapDestination :lon="coordenadas.longitude" :lat="coordenadas.latitude" :zoom="2" />
  </div>
  <ul>
    <li v-for="item in array" :key="item.id" :class="item.economic_level" @click="defineSitio(item)">{{ item.name }}</li>
  </ul>
</template>

<style scoped>
.cheap {
  color: green;
}

.expensive {
  color: red;
}

.moderate {
  color: orange;
}
</style>