<template>
    <div class="flex flex-col flex-1 items-center bg-gradient-to-b to-black"
        :class="{ 'from-sky-400/80': weatherCategoryAndIcon.category === 'Sunny', 'from-gray-600': weatherCategoryAndIcon.category === 'Rainy', 'from-gray-500': weatherCategoryAndIcon.category === 'Snowy', 'from-zinc-700': weatherCategoryAndIcon.category === 'Cloudy/Overcast' }">
        <!-- <div class="container mt-3 flex px-2">
            <RouterLink to="/" class="bg-zinc-300/20 shadow-md px-3 py-2 rounded-lg text-white">
                <i class="fa-solid fa-chevron-left"></i>
                <span class="ml-1"></span>
            </RouterLink>
        </div> -->
        <div v-if="route.query.preview" class="text-white p-4 bg-weather-secondary w-full text-center">
            <p>You are currently previewing this city, click the "+" icon to start tracking this city.</p>
        </div>
        <div class="flex flex-col items-center text-white pb-12 pt-6">
            <h1 class="text-4xl mb-2">
                {{ route.params.city }}
            </h1>
            <p class="text-sm mb-12">
                {{
                    new Date(weatherData.current.time).toLocaleDateString(
                        "en-us",
                        {
                            weekday: "short",
                            day: "2-digit",
                            month: "long",
                        }
                    )
                }}
                {{
                    new Date(weatherData.current.time).toLocaleTimeString(
                        "en-us",
                        {
                            timeStyle: "short",
                        }
                    )
                }}
            </p>
            <p class="text-7xl mb-8">
                {{ Math.round(weatherData.current.temperature_2m) }}{{ weatherData.current_units.temperature_2m }}
            </p>
            <p>
                Feels like
                {{ Math.round(weatherData.current.apparent_temperature) }}{{ weatherData.current_units.temperature_2m }}
            </p>
            <p class="capitalize">
                {{ weatherCategoryAndIcon.category }}
            </p>
            <!-- w-[150px] -->
            <img class="h-auto" :src="getIconPath(weatherCategoryAndIcon.iconCode)" alt=""
                :class="{ 'sun': weatherCategoryAndIcon.category === 'Sunny' }" />
        </div>
        <hr class="border-white border-opacity-10 border w-full" />

        <div class="max-w-screen-md w-full py-6 bg-slate-100/10 m-4 rounded-lg">
            <div class="mx-8 text-white">
                <h2 class="mb-4">Hourly Weather</h2>
                <div class="w-full">
                    <Carousel :transition="600" :wrapAround="false" :itemsToShow="5" ref="hourlyCarousel">
                        <Slide v-for="data in combinedData" :key="data.time"
                            class="flex flex-col gap-4 items-center snap-start"
                            :class="{ 'bg-slate-300/30 p-4 rounded-lg currentTime': new Date(data.time).getHours() === new Date().getHours() }">
                            <p class="whitespace-nowrap text-md">
                                {{
                                    new Date(
                                        data.time
                                    ).toLocaleTimeString("en-us", {
                                        hour: "numeric",
                                    })
                                }}
                            </p>
                            <img class="w-auto h-[50px] object-cover"
                                :src="getIconPath(weatherCategoryAndIcon.iconCode)" alt=""
                                :class="{ 'sun': weatherCategoryAndIcon.category === 'Sunny' }" />
                            <p class="text-xl">
                                {{ Math.round(data.temperature) }}&deg;C
                            </p>
                        </Slide>
                    </Carousel>

                </div>
            </div>
        </div>

        <div class="max-w-screen-md w-full py-6 bg-slate-100/10 m-4 rounded-lg">
            <div class="mx-8 text-white">
                <h2 class="mb-4">Hourly Wind</h2>
                <div class="w-full">
                    <Carousel :transition="600" :wrapAround="false" :itemsToShow="5" ref="hourlyWindCarousel">
                        <Slide v-for="data in combinedData" :key="data.time"
                            class="flex flex-col gap-4 items-center snap-start"
                            :class="{ 'bg-slate-300/30 p-4 rounded-lg currentTime': new Date(data.time).getHours() === new Date().getHours() }">
                            <p class="whitespace-nowrap text-md">
                                {{
                                    new Date(
                                        data.time
                                    ).toLocaleTimeString("en-us", {
                                        hour: "numeric",
                                    })
                                }}
                            </p>
                            <!-- <img class="w-auto h-[50px] object-cover"
                                :src="getIconPath(weatherCategoryAndIcon.iconCode)" alt="" /> -->
                            <p :style="{ transform: `rotate(${data.windDirection}deg)` }">
                                <i class="fa-solid fa-arrow-down"></i>
                            </p>
                            <p class="text-sm">
                                {{ Math.round(data.windSpeed) }}km/h
                            </p>
                        </Slide>
                    </Carousel>

                </div>
            </div>
        </div>

        <hr class="border-white border-opacity-10 border w-full" />

        <!-- Weekly Weather -->
        <div class="max-w-screen-md w-full py-6 bg-slate-100/10 m-4 rounded-lg">
            <div class="mx-8 text-white">
                <h2 class="mb-4">7 Day Forecast</h2>
                <div v-for="dailyData in combinedDailyData" :key="dailyData.day"
                    class="flex items-center [&:not(:last-child)]:border-b-white/15 [&:not(:last-child)]:border-b py-2">
                    <p class="flex-1">
                        {{
                            new Date(dailyData.day).toLocaleDateString(
                                "en-us",
                                {
                                    weekday: "long",
                                }
                            )
                        }}
                    </p>
                    <img class="w-[50px] h-[50px] object-cover" :src="getIconPath(weatherCategoryAndIcon.iconCode)"
                        alt="" :class="{ 'sun': weatherCategoryAndIcon.category === 'Sunny' }" />
                    <div class="flex gap-2 flex-1 justify-end">
                        <p class="bg-slate-100/10 p-2 rounded-lg whitespace-nowrap"><span
                                class="text-rose-300">H:</span> {{
                                    Math.round(dailyData.max)
                                }}&deg;C</p>
                        <p class="bg-slate-100/10 p-2 rounded-lg whitespace-nowrap"><span class="text-emerald-300">
                                L:</span> {{
                                    Math.round(dailyData.min)
                                }}&deg;C</p>
                    </div>
                </div>
            </div>
        </div>

        <div @click="removeCity" v-if="!route.query.preview"
            class="flex items-center gap-2 py-12 text-white cursor-pointer duration-150 hover:text-red-500">
            <i class="fa-solid fa-trash"></i>
            Remove City
        </div>
    </div>
</template>

<script setup>
import axios from 'axios';
import { onMounted, ref, computed, nextTick } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import 'vue3-carousel/dist/carousel.css'
import { Carousel, Slide, Pagination, Navigation } from 'vue3-carousel'
const hourlyCarousel = ref(null);
const hourlyWindCarousel = ref(null);
onMounted(async () => {

    // Wait for the next DOM update cycle
    await nextTick();

    // Select the element with the class 'currentTime'
    const currentTimeElement = document.querySelector('.currentTime');

    if (currentTimeElement) {
        // Assuming the scrollable container is the closest positioned ancestor
        // You might need to adjust the selector based on your actual DOM structure
        const scrollContainer = currentTimeElement.closest('.scrollable-container');

        if (scrollContainer) {
            const elementLeft = currentTimeElement.offsetLeft;
            const elementWidth = currentTimeElement.offsetWidth;
            const containerWidth = scrollContainer.offsetWidth;

            // Calculate the scroll position to center the element
            const scrollLeft = elementLeft - (containerWidth / 2) + (elementWidth / 2);

            // Apply the calculated scroll position to the container
            scrollContainer.scroll({
                left: scrollLeft,
                block: 'center',
                behavior: 'smooth'
            });
        }
    }
    hourlyCarousel.value.slideTo(new Date().getHours());
    hourlyWindCarousel.value.slideTo(new Date().getHours());
});

const weatherIconMap = {
    'Rainy': '10d', // Example icon code for rainy weather
    'Snowy': '13d', // Example icon code for snowy weather
    'Sunny': '01d', // Example icon code for sunny weather
    'Cloudy/Overcast': '04d', // Example icon code for cloudy or overcast weather
};

function categorizeWeather(weatherData) {
    let category;
    if (weatherData.rain > 0 || weatherData.showers > 0) {
        category = 'Rainy';
    } else if (weatherData.snowfall > 0) {
        category = 'Snowy';
    } else if (weatherData.is_day === 1 && weatherData.precipitation === 0 && weatherData.rain === 0 && weatherData.showers === 0 && weatherData.snowfall === 0) {
        category = 'Sunny';
    } else {
        category = 'Cloudy/Overcast';
    }
    // Assuming weatherIconMap is defined elsewhere in your script
    const iconCode = weatherIconMap[category] || 'unknown';
    return { category, iconCode };
}

const weatherCategoryAndIcon = computed(() => {
    return categorizeWeather(weatherData.current);
})

const getIconPath = (iconCode) => {
    return `http://openweathermap.org/img/wn/${iconCode}@2x.png`;
}
const router = useRouter();
const removeCity = () => {
    const cities = JSON.parse(localStorage.getItem("savedCities"));
    const updatedCities = cities.filter((city) => city.id !== route.query.id);
    localStorage.setItem("savedCities", JSON.stringify(updatedCities));
    router.push({ name: "home" });
}
// const weatherParams = "&current=temperature_2m,apparent_temperature,is_day,precipitation,rain,showers,snowfall,wind_speed_10m,wind_direction_10m,wind_gusts_10m&hourly=temperature_2m,relative_humidity_2m&daily=weather_code,temperature_2m_max,temperature_2m_min,apparent_temperature_max,apparent_temperature_min,sunrise,sunset,daylight_duration,sunshine_duration,uv_index_max,uv_index_clear_sky_max,precipitation_sum,rain_sum,showers_sum,snowfall_sum,precipitation_hours,precipitation_probability_max,wind_speed_10m_max,wind_gusts_10m_max&timezone=auto"
const weatherParams = "&current=temperature_2m,apparent_temperature,is_day,precipitation,rain,showers,snowfall,wind_speed_10m,wind_direction_10m,wind_gusts_10m&hourly=temperature_2m,relative_humidity_2m,precipitation,wind_speed_10m,wind_direction_10m&daily=weather_code,temperature_2m_max,temperature_2m_min,apparent_temperature_max,apparent_temperature_min,sunrise,sunset,daylight_duration,sunshine_duration,uv_index_max,uv_index_clear_sky_max,precipitation_sum,rain_sum,showers_sum,snowfall_sum,precipitation_hours,precipitation_probability_max,wind_speed_10m_max,wind_gusts_10m_max&timezone=auto"
const combinedData = ref([]);
const combinedDailyData = ref([]);
const route = useRoute();
const getWeatherData = async () => {
    try {
        const weatherData = await axios.get(`https://api.open-meteo.com/v1/forecast?latitude=${route.query.lat}&longitude=${route.query.lng}${weatherParams}`)
        // const combinedData = [];

        const today = new Date().toISOString().split('T')[0];

        // Loop through the hourly data and combine the data for today
        for (let i = 0; i < weatherData.data.hourly.time.length; i++) {
            const timeString = weatherData.data.hourly.time[i];
            const datePart = timeString.split('T')[0]; // Extract the date part from the time string

            // Check if the date part matches today's date
            if (datePart === today) {
                combinedData.value.push({
                    time: timeString,
                    temperature: weatherData.data.hourly.temperature_2m[i],
                    windSpeed: weatherData.data.hourly.wind_speed_10m[i],
                    precipitation: weatherData.data.hourly.precipitation[i],
                    windDirection: weatherData.data.hourly.wind_direction_10m[i],
                });
            }
        }
        for (let i = 0; i < weatherData.data.daily.time.length; i++) {
            const day = weatherData.data.daily.time[i];
            const weatherMin = weatherData.data.daily.temperature_2m_min[i];
            const weatherMax = weatherData.data.daily.temperature_2m_max[i];

            combinedDailyData.value.push({
                day: day,
                min: weatherMin,
                max: weatherMax
            });

        }
        // await new Promise((resolve) => setTimeout(resolve, 1000));
        console.log(combinedData.value);
        return weatherData.data;

    } catch (error) {
        console.log(error);
    }

}
const weatherData = await getWeatherData();
</script>

<style>
.sun {
    filter: hue-rotate(49deg) brightness(1.5);
}
</style>