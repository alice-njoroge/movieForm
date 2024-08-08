<script setup xmlns="http://www.w3.org/1999/html">
import {computed, ref} from "vue";
import {StarIcon} from "@heroicons/vue/24/solid";
import {items} from "./movies.json";
import ModalComponent from "./ModalComponent.vue";

const movies = ref(items);

const isOpen = ref(false)

function closeModal() {
  isOpen.value = false
}

function openModal() {
  isOpen.value = true
}

function updateRating(movieIndex, rating) {
  movies.value[movieIndex].rating = rating;
}

const genres = ref([
  {name: "Drama", value: "drama"},
  {name: "Action", value: "action"},
  {name: "Thriller", value: "thriller"}
]);

const form = ref({
  title: "",
  description: "",
  image: "",
  genres: [],
  inTheatre: false
});

const formDirty = ref({
  title: false,
  description: false,
  image: false,
  genres: false,
  inTheatre: false
});
const validTitle = computed(() => formDirty.value.title && form.value.title.length > 0);
const validGenre = computed(() => formDirty.value.genres && form.value.genres.length > 0)

const formValid = computed(() => {
  return validTitle.value && validGenre.value
});
const resetForm = () => {
  form.value = {
    title: "",
    description: "",
    image: "",
    genres: [],
    inTheatre: false
  };
};
const handleSubmit = () => {
  if (formValid.value) {
    items.push(form.value);
    //clear the form before closing the modal
    resetForm();
    closeModal();
  }
}


</script>

<template>
  <div class="fixed  left-0 top-0">
    <button
        type="button"
        @click="openModal"
        class="rounded-md bg-black/20 px-4 py-2 text-sm font-medium text-white hover:bg-black/30 focus:outline-none focus-visible:ring-2 focus-visible:ring-white/75"
    >
      Open dialog
    </button>
  </div>
  <ModalComponent
      :isOpen="isOpen"
      v-if="isOpen"
      @closeModal="closeModal"
      @openModal="openModal"
      class="w-full shadow-md"
  >
    <template #title> Add a Movie</template>
    <template #body>
      <form class="bg-white rounded px-8 pt-6 pb-8 mb-4" @submit.prevent="handleSubmit">
        {{ form }}
        <div class="mb-4">
          <label class="block text-gray-700 text-sm font-bold mb-2" for="title">Title</label>
          <input v-model="form.title"
                 @blur="formDirty.title = true"
                 class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                 id="title" type="text" placeholder="title">
          <p class="text-red-600 text-xs" v-if="!validTitle"> Title is required </p>
        </div>

        <div class="mb-4">
          <label class="block text-gray-700 text-sm font-bold mb-2" for="description">Description</label>
          <textarea @blur="formDirty.description = true" v-model="form.description"
                    class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                    id="description" type="text"></textarea>
        </div>
        <div class="mb-4">
          <label class="block text-gray-700 text-sm font-bold mb-2" for="images">Images</label>
          <input @blur="formDirty.image = true" v-model="form.image"
                 class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                 id="images" type="text">
        </div>
        <div class="mb-4">
          <label class="block text-gray-700 text-sm font-bold mb-2" for="genre">Genre</label>
          <select @blur="formDirty.genres = true" v-model="form.genres" multiple
                  class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline">
            <option v-for="genre in genres" :key="genre.value"> {{ genre.name }}</option>
          </select>
          <p class="text-red-600 text-xs" v-if="!validGenre"> Genre is required </p>
        </div>
        <div class="mb-4">
          <input @blur="formDirty.inTheatre = true" v-model="form.inTheatre" type="checkbox"/> In Theatres
        </div>
        <div class="flex justify-between">
          <button type="submit" class="bg-blue-300 border-1 px-4 py-1 m-0.5 rounded-full" @click="handleSubmit">
            Submit
          </button>
          <button type="submit" class="border-1 px-4 py-1 m-0.5 rounded-full" @click="resetForm">
            Cancel
          </button>
        </div>
      </form>

    </template>
  </ModalComponent>

  <div class="app">
    <div class="movie-list">
      <div
          class="movie-item"
          v-for="(movie, movieIndex) in movies"
          :key="movie.id"
      >
        <div class="movie-item-image-wrapper">
          <div class="movie-item-star-wrapper">
            <StarIcon
                class="movie-item-star-rating-icon"
                :class="[movie.rating ? 'text-yellow-500' : 'text-gray-500']"
            />
            <div class="movie-item-star-content-wrapper">
              <span
                  v-if="movie.rating"
                  class="movie-item-star-content-rating-rated"
              >
                {{ movie.rating }}
              </span>
              <span v-else class="movie-item-star-content-rating-not-rated">
                -
              </span>
            </div>
          </div>
          <img :src="movie.image" class="movie-item-image" alt=""/>
        </div>

        <div class="movie-item-content-wrapper">
          <div class="movie-item-title-wrapper">
            <h3 class="movie-item-title">{{ movie.name }}</h3>
            <div class="movie-item-genres-wrapper">
              <span
                  v-for="genre in movie.genres"
                  :key="`${movie.id}-${genre}`"
                  class="movie-item-genre-tag"
              >{{ genre }}</span
              >
            </div>
          </div>
          <div class="movie-item-description-wrapper">
            <p class="movie-item-description">{{ movie.description }}</p>
          </div>
          <div class="movie-item-rating-wrapper">
            <span class="movie-item-rating-text">
              Rating: ({{ movie.rating }}/5)
            </span>

            <div class="movie-item-star-icon-wrapper">
              <button
                  v-for="star in 5"
                  :key="star"
                  class="movie-item-star-icon-button"
                  :class="[
                  star <= movie.rating ? 'text-yellow-500' : 'text-gray-500',
                ]"
                  :disabled="star === movie.rating"
                  @click="updateRating(movieIndex, star)"
              >
                <StarIcon class="movie-item-star-icon"/>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style>

</style>