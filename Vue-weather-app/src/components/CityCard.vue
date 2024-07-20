<template>
    <div class="flex py-6 px-3 bg-weather-secondary/60 rounded-md shadow-md cursor-pointer my-2">
        <div class="flex flex-col flex-1 truncate">
            <h2 class="text-3xl truncate">{{ city.city }}</h2>
            <h3>{{ city.state }}</h3>
        </div>

        <div class="flex flex-col gap-2 justify-center">
            <p class="text-3xl self-end">
                {{ Math.round(city.temps.current) }}&deg;C
            </p>
            <div class="flex gap-2">
                <span class="text-xs">
                    <span class="text-rose-300">High:</span>
                    {{ Math.round(city.temps.high) }}&deg;C
                </span>
                <span class="text-xs">
                    <span class="text-emerald-300"> Low:</span>
                    {{ Math.round(city.temps.low) }}&deg;C
                </span>
            </div>
        </div>
    </div>
</template>

<script setup>
defineProps({
    city: {
        type: Object,
        default: () => { },
    },
});
</script>