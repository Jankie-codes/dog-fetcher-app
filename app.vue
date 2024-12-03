<script setup lang="js">
  import { onMounted } from 'vue'

  const imageLink = ref("");

  onMounted(async () => {
    imageLink.value = await fetchDogImage();
  })
 
  const fetchDogImage = async () => {
    try {
      const response = await $fetch("https://dog.ceo/api/breeds/image/random");
      return response.message;
    } catch (error) {
      console.error(error.message);
    }
  }

  const updateImage = async () => {
    imageLink.value = await fetchDogImage();
  }
</script>

<template>
  <div class="min-h-screen flex flex-col space-y-3 justify-center items-center pb-8">
    <NuxtRouteAnnouncer />
    <p class="underline text-6xl font-bold">Dog Fetcher App</p>
    <img v-if="imageLink" :src="imageLink" alt="[Dog image]"/>
    <p v-if="!imageLink">Loading...</p>
    <button v-on:click="updateImage" class="border border-black rounded-md bg-gray-300 hover:bg-gray-400 active:bg-gray-500" type="button">Display a new dog image</button> 
  </div>
</template>