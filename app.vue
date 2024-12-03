<script setup lang="js">
  import { onMounted } from 'vue'

  const dogApiWebsite = "https://dog.ceo";

  const dogImageLink = useState("dogImageLink", () => "");
  const dogBreeds = useState("dogBreeds", () => []);
  const selectedBreed = useState("selectedBreed", () => "all breeds");

  onMounted(async () => {
    dogImageLink.value = await fetchDogImage();
    dogBreeds.value = await fetchDogBreeds();
  })
 
  const fetchDogImage = async () => {
    try {
      const response = await $fetch(`${dogApiWebsite}/api/breeds/image/random`);
      return response.message;
    } catch (error) {
      console.error(error.message);
    }
  }

  const fetchDogImageFromBreed = async (breed) => {
    if (breed.toLowerCase() === "all breeds") {
      return await fetchDogImage();
    }

    try {
      const breedFirstWord = breed.split(' ')[0].toLowerCase();
      var breedSecondWord = "";

      if (breed.split(' ').length > 1) {
        breedSecondWord = breed.split(' ')[1].toLowerCase();
      }

      //console.log("first", breedFirstWord);

      const response = breedSecondWord ? 
                          await $fetch(`${dogApiWebsite}/api/breed/${breedSecondWord}/${breedFirstWord}/images/random`)
                        : 
                          await $fetch(`${dogApiWebsite}/api/breed/${breedFirstWord}/images/random`);
      return response.message;
    } catch (error) {
      console.error(error.message);
    }
  }

  const updateImage = async () => {
    //dogImageLink.value = await fetchDogImage();

    dogImageLink.value = await fetchDogImageFromBreed(selectedBreed.value);
  }

  const capitalizeWords = (sentence) => {
    return sentence
    .split(' ')
    .map(word => 
      word.charAt(0).toUpperCase() + word.slice(1).toLowerCase()
    )
    .join(' ');
  }

  const allBreedsListFromResponse = (breedData) => {
    const breedsList = [];

    for (const generalBreed in breedData) {
      if (breedData[generalBreed].length === 0) {
        breedsList.push(generalBreed);
      }

      for (let i = 0; i < breedData[generalBreed].length; i++) {
        const breedDescriberWord = breedData[generalBreed][i];

        breedsList.push(`${breedDescriberWord} ${generalBreed}`);
      }
    }

    return breedsList;
  }

  const fetchDogBreeds = async () => {
    try {
      const response = await $fetch(`${dogApiWebsite}/api/breeds/list/all`);
      const allBreedsList = allBreedsListFromResponse(response.message);
      return allBreedsList;
    } catch (error) {
      console.log(error.message);
    }
  }
</script>

<template>
  <div class="min-h-screen flex flex-col space-y-3 justify-center items-center pb-8">
    <NuxtRouteAnnouncer />
    <p class="underline text-6xl font-bold">Dog Fetcher App</p>
    <img v-if="dogImageLink" :src="dogImageLink" alt="[Dog image]"/>
    <p v-if="!dogImageLink">Loading...</p>
    
    <div class="flex flex-row space-x-12">
      <div class="flex flex-col space-y-2">
        <label for="breeds">Filter dog breed:</label>
        <select v-model="selectedBreed" name="breeds" id="breeds" class="p-2 border rounded-md bg-gray-300 hover:bg-gray-400 active:bg-gray-500">
          <option value="all breeds">All Breeds</option>
          <option v-for="breed in dogBreeds" :value="breed">{{ capitalizeWords(breed) }}</option>
        </select> 
      </div>
      <button v-on:click="updateImage" class="p-12 border border-black rounded-md bg-green-300 hover:bg-green-400 active:bg-green-500" type="button">Display a new dog image</button> 
    </div>
  </div>
</template>