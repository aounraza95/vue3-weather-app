<template>
    <div v-for="city in savedCities" :key="city.id">
        <CityCard :city="city" @click="goToCityView(city)" />
    </div>

    <p v-if="savedCities.length === 0">
        No Locations added.
    </p>
</template>

<script setup>
import axios from 'axios';
import { ref } from 'vue';
import router from '../router';
import CityCard from './CityCard.vue';

const openweatherAPIkey = import.meta.env.VITE_OpenWeatherAPIKey;
const savedCities = ref([]);
const getCities = async () => {
    if (localStorage.getItem('savedCities')) {
        savedCities.value = JSON.parse(localStorage.getItem('savedCities'));
    }

    const requests = [];
    savedCities.value.forEach((city) => {
        requests.push(
            axios.get(`https://api.openweathermap.org/data/3.0/onecall?lat=${city.coords.lat}&lon=${city.coords.lng}&appid=${openweatherAPIkey}&units=imperial`)
        )
    });

    const weatherData = await Promise.all(requests);

    // Flicker delay
    await new Promise((res) => setTimeout(res, 1000));
    weatherData.forEach((value, index) => {
        savedCities.value[index].weather = value.data;
    });

};

await getCities();

const goToCityView = (city) => {
    router.push({
        name: "cityView",
        params: { state: city.state, city: city.city },
        query: { id:city.id, lat: city.coords.lat, lng: city.coords.lng }
    })
}
</script>

