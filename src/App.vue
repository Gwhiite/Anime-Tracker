<script setup>
import { ref, onMounted, computed } from "vue";
const query = ref("");
const my_anime = ref([]);
const search_results = ref([]);
const my_anime_asc = computed(() => {
  return my_anime.value.sort((a, b) => {
    return a.title.localeCompare(b.title);
  });
});
const searchAnime = () => {
  const url = `https://api.jikan.moe/v4/anime?q=${query.value}`;
  fetch(url)
    .then((res) => res.json())
    .then((data) => {
      search_results.value = data.data;
    });
};
const handleInput = (e) => {
  if (!e.target.value) {
    search_results.value = [];
  }
};
const addAnime = (anime) => {
  search_results.value = [];
  query.value = "";
  my_anime.value.push({
    id: anime.mal_id,
    title: anime.title,
    image: anime.images.jpg.image_url,
    total_episodes: anime.episodes,
    watched_episodes: 0,
  });
  localStorage.setItem("my_anime", JSON.stringify(my_anime.value));
};
const increaseWatch = (anime) => {
  anime.watched_episodes++;
  localStorage.setItem("my_anime", JSON.stringify(my_anime.value));
};
const decreaseWatch = (anime) => {
  anime.watched_episodes--;
  localStorage.setItem("my_anime", JSON.stringify(my_anime.value));
};
onMounted(() => {
  my_anime.value = JSON.parse(localStorage.getItem("my_anime")) || [];
});
</script>

<template>
  <main>
    <h1>Anime Tracker</h1>

    <form @submit.prevent="searchAnime">
      <input
        type="text"
        placeholder="Search for an anime..."
        v-model="query"
        @input="handleInput"
      />
      <button type="submit" class="button">Search</button>
    </form>

    <div class="results" v-if="search_results.length > 0">
      <div v-for="anime in search_results" class="result">
        <img :src="anime.images.jpg.image_url" />
        <div class="details">
          <h3>{{ anime.title }}</h3>
          <p :title="anime.synopsis" v-if="anime.synopsis">
            {{ anime.synopsis.slice(0, 120) }}...
          </p>
          <span></span>
          <button @click="addAnime(anime)" class="button">Add</button>
        </div>
      </div>
    </div>
    <h1 v-if="my_anime.length > 0">Watching</h1>
    <div class="myanime" v-if="my_anime.length > 0">
      <div v-for="anime in my_anime_asc" class="anime">
        <img :src="anime.image" />
        <h3>{{ anime.title }}</h3>
        <div class="flex-1"></div>
        <span class="episodes"
          >{{ anime.watched_episodes }} / {{ anime.total_episodes }}</span
        >
        <button
          v-if="anime.total_episodes !== anime.watched_episodes"
          @click="increaseWatch(anime)"
          class="button"
        >
          +
        </button>
        <button
          v-if="anime.watched_episodes > 0"
          @click="decreaseWatch(anime)"
          class="button"
        >
          -
        </button>
      </div>
    </div>
  </main>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
body {
  font-family: sans-serif;
  background-color: black;
  color: white;
}
main {
  display: flex;
  flex-direction: column;
  align-items: center;
}
h1 {
  color: crimson;
  margin-top: 40px;
  margin-bottom: 10px;
}
form {
  display: flex;
  gap: 15px;
  margin-bottom: 30px;
}
input {
  border: solid 2px crimson;
  font-size: 1.15rem;
  border-radius: 5px;
  color: crimson;
  padding: 0.25rem 1rem;
}
.button {
  border: none;
  font-size: 1.15rem;
  border-radius: 5px;

  background-color: crimson;
  padding: 0.25rem 1rem;
  cursor: pointer;
}

.results {
  display: flex;
  align-items: flex-start;
  justify-content: space-around;
  flex-wrap: wrap;
  gap: 25px;
}
.details {
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.details p {
  max-width: 20ch;
}
h3 {
  max-width: 20ch;
}
img {
  display: block;
  max-width: 100%;
  border-radius: 5px;
  margin-bottom: 20px;
}
.myanime {
  display: flex;
  align-items: flex-start;
  justify-content: space-around;
  flex-wrap: wrap;
  gap: 25px;
}
.anime button {
  margin-left: 5px;
}
</style>
