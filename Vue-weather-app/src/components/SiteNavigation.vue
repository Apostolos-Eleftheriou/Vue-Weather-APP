<template>
    <header class="sticky top-0 bg-weather-primary/60 shadow-lg z-10 backdrop-blur-md">
        <nav class="container flex flex-row item-center justify-between gap-4 text-white py-6 ">
            <RouterLink :to="{ name: 'home' }" class="w-full">
                <div class="flex items-center gap-3">
                    <img src="/src/assets/tornado.svg" alt="logo" class="max-w-[30px] invert">
                    <p class="text-2xl">Vue Weather</p>
                </div>
            </RouterLink>
            <div class="flex gap-3 justify-end items-center">
                <i @click="toggleModal"
                    class="fa-solid fa-circle-info text-xl hover:text-weather-secondary duration-150 cursor-pointer"></i>
                <i @click="addCity" v-if="route.query.preview"
                    class="fa-solid fa-plus text-xl hover:text-weather-secondary duration-150 cursor-pointer"></i>
            </div>
            <BaseModal :modalActive="modalActive" @close-modal="toggleModal">
                <div class="text-black">
                    <h1 class="text-2xl mb-1">About:</h1>
                    <p class="mb-4">
                        The Local Weather allows you to track the current and
                        future weather of cities of your choosing.
                    </p>
                    <h2 class="text-2xl">How it works:</h2>
                    <ol class="list-decimal list-inside mb-4">
                        <li>
                            Search for your city by entering the name into the
                            search bar.
                        </li>
                        <li>
                            Select a city within the results, this will take
                            you to the current weather for your selection.
                        </li>
                        <li>
                            Track the city by clicking on the "+" icon in the
                            top right. This will save the city to view at a
                            later time on the home page.
                        </li>
                    </ol>

                    <h2 class="text-2xl">Removing a city</h2>
                    <p>
                        If you no longer wish to track a city, simply select
                        the city within the home page. At the bottom of the
                        page, there will be am option to delete the city.
                    </p>
                </div>
            </BaseModal>
        </nav>
    </header>
</template>

<script setup>
import { RouterLink, useRoute, useRouter } from 'vue-router';
import BaseModal from './BaseModal.vue';
import { ref } from 'vue';
import { uid } from 'uid';
import { useToast } from 'vue-toastification'

const toast = useToast()

const route = useRoute();
const router = useRouter();

const modalActive = ref(false);

const savedCities = ref([]);
const addCity = () => {
    if (localStorage.getItem('savedCities')) {
        savedCities.value = JSON.parse(localStorage.getItem('savedCities'));
    }
    const locationObj = {
        id: route.query.id,
        state: route.params.state,
        city: route.params.city,
        date: route.query.date,
        coords: {
            lat: route.query.lat,
            lng: route.query.lng
        }
    }

    savedCities.value.push(locationObj);
    localStorage.setItem('savedCities', JSON.stringify(savedCities.value));
    let query = Object.assign({}, route.query);
    query.id = locationObj.id;
    delete query.preview;
    router.replace({ query });
    toast.success(`${route.params.city} added to favorites!`)
}

const toggleModal = () => {
    modalActive.value = !modalActive.value;
}
</script>

<style scoped></style>