<script setup lang="js">
  import { onMounted } from "vue";
  import { debounce } from "lodash";

  const dogApiWebsite = "https://dog.ceo";

  const dogImageLink = useState("dogImageLink", () => "");
  const dogBreeds = useState("dogBreeds", () => []);
  const selectedBreed = useState("selectedBreed", () => "all breeds");

  //void -> void
  //MODIFIES: dogImageLink, dogBreeds
  //EFFECTS: on initial render, fetch a random dog image and a list of all dog breeds
  onMounted(async () => {
    dogImageLink.value = await fetchDogImage();
    dogBreeds.value = await fetchDogBreeds();
  })
  
  //void -> String
  //EFFECTS: fetches a random dog image (url link), of any breed, using API. If an error occurs, returns an "error occurred" image link.
  const fetchDogImage = async () => {
    try {
      const response = await $fetch(`${dogApiWebsite}/api/breeds/image/random`);
      return response.message;
    } catch (error) {
      console.error(error.message);
      return "https://i.imgur.com/3d1sBhG.png";
    }
  }

  //String -> String
  //EFFECTS: given a breed, fetches a random dog image (url link) of the given breed. If an error occurred, returns an "error occurred" image url link.
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

      const response = breedSecondWord ? 
                          await $fetch(`${dogApiWebsite}/api/breed/${breedSecondWord}/${breedFirstWord}/images/random`)
                        : 
                          await $fetch(`${dogApiWebsite}/api/breed/${breedFirstWord}/images/random`);
      return response.message;
    } catch (error) {
      console.error(error.message);
      return "https://i.imgur.com/3d1sBhG.png";
    }
  }

  //void -> void
  //MODIFIES: dogImageLink
  //EFFECTS: updates the dogImageLink state with a new image link. Asynchronous: uses API.
  const updateImage = async () => {
    dogImageLink.value = await fetchDogImageFromBreed(selectedBreed.value);
  }

  //String -> String
  //EFFECTS: given a sentence string, capitalize the first letter of each word in the string.
  const capitalizeWords = (sentence) => {
    return sentence
    .split(' ')
    .map(word => 
      word.charAt(0).toUpperCase() + word.slice(1).toLowerCase()
    )
    .join(' ');
  }

  //Object -> (listof String)
  //EFFECTS: Given the response object from the fetchDogBreeds API call, produces a list of strings representing all breeds.
  //The original response object (breedData) has fields representing the breed, and a list of describer words (e.g. in "golden retriever", "golden" is the describer word" and "retriever" is the breed).
  //For each breed, add all combinations of "[describer word] [breed]" into the final list of breeds. E.g. "Kelpie Australian", "Shepherd Australian".
  //If a given breed has no describer words given, return only the breed name. E.g. "Akita".
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

  //void -> Object
  //EFFECTS: calls the dog.ceo API call to get all dog breeds. The response is an object with fields representing breed names, and values being a list of possible describer words for that breed.
  const fetchDogBreeds = async () => {
    try {
      const response = await $fetch(`${dogApiWebsite}/api/breeds/list/all`);
      const allBreedsList = allBreedsListFromResponse(response.message);
      return allBreedsList;
    } catch (error) {
      console.error(error.message);
    }
  }

  //void -> void
  //MODIFIES: dogImageLink
  //EFFECTS: limits how often updateImage is called. Multiple api calls within 200ms of each other are ignored.
  const debouncedUpdateImage = debounce(() => {
    updateImage();
  }, 200);
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
      <button v-on:click="debouncedUpdateImage" class="p-12 border border-black rounded-md bg-green-300 hover:bg-green-400 active:bg-green-500" type="button">Display a new dog image</button> 
    </div>
  </div>
</template>
