<template>
    <main class="container text-white">
        <div class="pt-4 mb-8 relative">
            <div class="relative">
                <div class="absolute border-r left-1 top-2 flex items-center cursor-pointer px-2 ">
                    <i class="fa-solid fa-magnifying-glass text-white text-xl"></i>
                </div>
                <input type="text" placeholder="Search for a city or state" v-model="searchQuery"
                    @input="getSearchResults"
                    class="py-2 pr-[30px] pl-[50px] w-full bg-transparent border rounded-lg focus:border-[#ffd859] focus:outline-none mb-2">
                <div @click="handleClear" class="absolute right-1 top-2 flex items-center cursor-pointer px-2 "
                    v-if="searchQuery !== ''">
                    <i class="fa-solid fa-times text-white text-xl"></i>
                </div>
            </div>
            <ul v-if="mapboxSearchResults"
                class="absolute bg-weather-secondary/70 backdrop-blur-md text-white w-full shadow-md shadow-[#ffd859] py-2 px-2 top-[66px] rounded-lg">
                <p class="py-2 px-2" v-if="loading">
                    <AnimatedPlaceholder v-for="times in 5" class="my-2" />
                </p>
                <p class="py-2 px-2" v-if="searchError && !loading">
                    Sorry, something went wrong, please try again.
                </p>
                <p class="py-2 px-2" v-if="!searchError && !mapboxSearchResults.results && !loading">
                    No results match your query, try a different term.
                </p>
                <template v-else-if="!loading">
                    <li v-for="searchResult in mapboxSearchResults.results" :key="searchResult.place_id"
                        @click="previewCity(searchResult)"
                        class="py-2 px-4 cursor-pointer hover:bg-weather-primary/40 rounded-lg truncate">
                        <i class="fa-solid fa-location-dot text-white mr-2"></i>
                        <!-- {{ searchResult.name }}, {{ searchResult.address.state }}, {{ searchResult.address.country }} -->
                        {{ searchResult.name }}, {{ searchResult.admin1 }}, {{ searchResult.country }}
                    </li>
                </template>
            </ul>

            <!-- <div class="flex flex-col gap-4"> -->
            <Suspense>
                <CityList />
                <template #fallback>
                    <CityCardSkeleton />
                </template>
            </Suspense>
            <!-- </div> -->
        </div>
    </main>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";
import CityList from "../components/CityList.vue";
import CityCardSkeleton from "../components/CityCardSkeleton.vue";
import AnimatedPlaceholder from "@/components/AnimatedPlaceholder.vue";
import { useToast } from 'vue-toastification'

const toast = useToast()

const router = useRouter();

const handleClear = () => {
    searchQuery.value = "";
    mapboxSearchResults.value = null;
};

const previewCity = (searchResults) => {
    // console.log(searchResults);
    // const [city, state] = searchResults.place_name.split(",");
    // const city = searchResults.address.city != null ? searchResults.address.city : searchResults.address.state != null ? searchResults.address.state : searchResults.address.country;
    // const state = searchResults.address.state != null ? searchResults.address.state : searchResults.address.country;
    const city = searchResults.name;
    const state = searchResults.admin1;
    const id = searchResults.id;

    if (localStorage.getItem("savedCities")) {
        const savedCities = JSON.parse(localStorage.getItem("savedCities"));
        if (savedCities.find(city => parseInt(city.id) === parseInt(id))) {
            toast.error(`${searchResults.name} is already added!`)
            return;
        }
    }
    router.push({
        name: "cityView", params: { state: state.replaceAll(" ", ""), city: city },
        query: {
            id: searchResults.id,
            lat: searchResults.latitude,
            lng: searchResults.longitude,
            date: new Date().toLocaleDateString('en-CA', {
                year: 'numeric',
                month: '2-digit',
                day: '2-digit'
            }),
            preview: true
        }
    });
};
const searchQuery = ref("");
const queryTimeout = ref(null);
const mapboxSearchResults = ref(null);
const searchError = ref(null);
const loading = ref(false);

const getSearchResults = () => {
    clearTimeout(queryTimeout.value);
    loading.value = true;
    queryTimeout.value = setTimeout(async () => {
        if (searchQuery.value !== "") {
            try {
                // const result = await axios.get(`https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`)
                // const result = await axios.get(`https://nominatim.openstreetmap.org/search?q=${searchQuery.value}&format=json&addressdetails=1&limit=10`)
                const result = await axios.get(`https://geocoding-api.open-meteo.com/v1/search?name=${searchQuery.value}&count=10&language=en&format=json`)
                mapboxSearchResults.value = result.data;
            } catch (error) {
                searchError.value = true;
            }
            loading.value = false;
            return;
        }

        loading.value = false;
        mapboxSearchResults.value = null;
    }, 300);
};

</script>

<style scoped></style>