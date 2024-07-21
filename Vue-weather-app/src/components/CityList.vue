<template>
    <div v-for="city in savedCities" :key="city.id">
        <CityCard :city="city" @click="goToCityView(city)" />
    </div>

    <p v-if="savedCities.length === 0" class="p-3 my-3 bg-weather-secondary rounded-lg">
        No locations added. To start tracking a location, search in
        the field above.
    </p>
</template>

<script setup>
import axios from "axios";
import { ref } from "vue";
import { useRouter } from "vue-router";
import CityCard from "./CityCard.vue";

const savedCities = ref([]);
const weatherParams = "&current=temperature_2m,apparent_temperature,is_day,precipitation,rain,showers,snowfall,wind_speed_10m,wind_direction_10m,wind_gusts_10m&hourly=temperature_2m,relative_humidity_2m&daily=weather_code,temperature_2m_max,temperature_2m_min,apparent_temperature_max,apparent_temperature_min,sunrise,sunset,daylight_duration,sunshine_duration,uv_index_max,uv_index_clear_sky_max,precipitation_sum,rain_sum,showers_sum,snowfall_sum,precipitation_hours,precipitation_probability_max,wind_speed_10m_max,wind_gusts_10m_max&timezone=auto"
const getCities = async () => {
    if (localStorage.getItem("savedCities")) {
        savedCities.value = JSON.parse(
            localStorage.getItem("savedCities")
        );

        const requests = [];
        // console.log(savedCities)
        savedCities.value.forEach((city) => {
            requests.push(
                axios.get(
                    `https://api.open-meteo.com/v1/forecast?latitude=${city.coords.lat}&longitude=${city.coords.lng}${weatherParams}`
                )
            );
        });

        const weatherData = await Promise.all(requests);

        // await new Promise((resolve) => setTimeout(resolve, 1000));

        weatherData.forEach((value, index) => {
            savedCities.value[index].temps = {
                high: value.data.daily.temperature_2m_max[0],
                low: value.data.daily.temperature_2m_min[value.data.daily.temperature_2m_min.length - 1],
                current: value.data.current.temperature_2m,
            };
        });
    }
};
await getCities();

const router = useRouter();
const goToCityView = (city) => {
    router.push({
        name: "cityView",
        params: { state: city.state, city: city.city },
        query: {
            id: city.id,
            lat: city.coords.lat,
            lng: city.coords.lng,
            date: city.date,
        },
    });
};
</script>