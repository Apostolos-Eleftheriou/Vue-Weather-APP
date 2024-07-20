<template>
    <main class="container text-white">
        <div class="pt-4 mb-8 relative">
            <div class="relative">
                <input type="text" placeholder="Search for a city or state" v-model="searchQuery"
                    @input="getSearchResults"
                    class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#ffd859] mb-2">
                <div @click="handleClear"
                    class="absolute right-0 top-2 flex items-center cursor-pointer px-2 bg-weather-primary"
                    v-if="searchQuery !== ''">
                    <i class="fa-solid fa-times text-white text-xl"></i>
                </div>
            </div>
            <ul v-if="mapboxSearchResults"
                class="absolute bg-weather-secondary/70 backdrop-blur-md text-white w-full shadow-md shadow-emerald-400/30 py-2 px-2 top-[66px] rounded-lg">
                <p class="py-2 px-2" v-if="loading">
                    <AnimatedPlaceholder v-for="times in 5" class="my-2" />
                </p>
                <p class="py-2 px-2" v-if="searchError && !loading">
                    Sorry, something went wrong, please try again.
                </p>
                <p class="py-2 px-2" v-if="!searchError && mapboxSearchResults.length === 0 && !loading">
                    No results match your query, try a different term.
                </p>
                <template v-else-if="!loading">
                    <li v-for="searchResult in mapboxSearchResults" :key="searchResult.place_id"
                        @click="previewCity(searchResult)"
                        class="py-2 px-4 cursor-pointer hover:bg-weather-primary/40 rounded-lg truncate">
                        <i class="fa-solid fa-location-dot text-white mr-2"></i>
                        {{ searchResult.name }}, {{ searchResult.address.state }}, {{ searchResult.address.country }}
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

const router = useRouter();

const handleClear = () => {
    searchQuery.value = "";
    mapboxSearchResults.value = null;
};

const previewCity = (searchResults) => {
    console.log(searchResults);
    // const [city, state] = searchResults.place_name.split(",");
    const city = searchResults.address.city != null ? searchResults.address.city : searchResults.address.state != null ? searchResults.address.state : searchResults.address.country;
    const state = searchResults.address.state != null ? searchResults.address.state : searchResults.address.country;
    router.push({
        name: "cityView", params: { state: state.replaceAll(" ", ""), city: city },
        query: {
            lat: searchResults.lat,
            lng: searchResults.lon,
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
                const result = await axios.get(`https://nominatim.openstreetmap.org/search?q=${searchQuery.value}&format=json&addressdetails=1&limit=10`)
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