<script setup>
import { onMounted, ref } from "vue";

const data = ref(null);
const error = ref(null);
const query = ref(null);
const favorites = ref(new Map());

const API = "https://api.github.com";

const getGithubUser = async ({ query }) => {
  data.value = null;
  error.value = null;

  if (!query) return;

  try {
    const response = await fetch(`${API}/search/users?q=${query}`);
    const result = await response.json();
    if (!result.total_count) {
      throw new Error("User not found");
    }
    data.value = result;
  } catch (e) {
    error.value = e;
  }
};

const addFavorite = ({ id, value }) => {
  favorites.value.set(id, value);
  updateStorage();
};

const removeFavorite = ({ id }) => {
  favorites.value.delete(id);
  updateStorage();
};

const updateStorage = () => {
  window.localStorage.setItem(
    "favorites",
    JSON.stringify(Array.from(favorites.value.values()))
  );
};

onMounted(() => {
  const savedStorage = JSON.parse(window.localStorage.getItem("favorites"));
  if (savedStorage) {
    favorites.value = new Map(
      savedStorage.map((favorite) => [favorite.id, favorite])
    );
  }
});
</script>

<template>
  <main class="main">
    <h1>Github User Search</h1>
    <form class="search-form" @submit.prevent="getGithubUser({ query })">
      <input
        class="search"
        placeholder="alinpr18"
        v-model="query"
        type="text"
      />
      <button class="submit">Search</button>
    </form>
    <Transition>
      <div v-if="favorites.size > 0">
        <h2>Favorites</h2>
        <TransitionGroup tag="ul" name="list">
          <li v-for="[_, favorite] in favorites" :key="favorite.id">
            <p>
              {{ favorite.login }}
              <button @click="removeFavorite({ id: favorite.id })">🗑️</button>
            </p>
          </li>
        </TransitionGroup>
      </div>
    </Transition>
      <p class="error" v-if="error">{{ error }}</p>
      <div v-else-if="data">
        <h2>Results</h2>
        <ul class="users-list">
          <li class="user" v-for="user in data.items" :key="user.id">
            <img
              width="50"
              height="50"
              :src="user.avatar_url"
              :alt="user.login"
            />
            <a :href="user.html_url" target="_blank">{{ user.login }}</a>
            <button
              v-if="!favorites.has(user.id)"
              @click="addFavorite({ id: user.id, value: user })"
            >
              ☆
            </button>
            <button v-else @click="removeFavorite({ id: user.id })">★</button>
          </li>
        </ul>
      </div>
  </main>
</template>

<style>
/* we will explain what these classes do next! */
.v-enter-active,
.v-leave-active {
  transition: opacity 0.5s ease;
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
}

.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

.main {
  max-width: 600px;
  margin: auto;
}

.error {
  color: white;
  padding: 10px;
  font-weight: 600;
  border-radius: 4px;
  text-align: center;
  background-color: red;
}

.search-form {
  display: flex;
  justify-content: space-evenly;
  gap: 5px;
}

.search {
  width: 280px;
  padding: 10px;
  box-sizing: border-box;
  border-radius: 4px;
  border: 1px solid black;
}

.submit {
  width: 280px;
  background-color: chartreuse;
  padding: 10px;
  border-color: chartreuse;
  border: 1px solid chartreuse;
  border-radius: 4px;
}

.users-list {
  padding: 0;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.user {
  padding: 10px;
  border: 1px solid black;
  border-radius: 4px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  gap: 10px;
}
</style>
