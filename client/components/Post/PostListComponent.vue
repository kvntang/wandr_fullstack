<script setup lang="ts">
import { ref, onBeforeMount } from "vue";
import CarouselComponent from "@/components/Post/CarouselComponent.vue";
import { fetchy } from "@/utils/fetchy";

const loaded = ref(false);
let posts = ref<Array<Record<string, string>>>([]);
let editing = ref("");
let searchAuthor = ref("");

async function getPosts(author?: string) {
  let query: Record<string, string> = author !== undefined ? { author } : {};
  let postResults;
  try {
    postResults = await fetchy("/api/posts", "GET", { query });
  } catch (_) {
    return;
  }
  searchAuthor.value = author ? author : "";
  posts.value = postResults;
}

function updateEditing(id: string) {
  editing.value = id;
}

onBeforeMount(async () => {
  await getPosts();
  loaded.value = true;
});
</script>

<template>
  <div class="row">
    <h2 v-if="!searchAuthor">Feed:</h2>
    <h2 v-else>Posts by {{ searchAuthor }}:</h2>
  </div>

  <!-- Load a carousel for each post, first card is the post, rest are placeholders -->
  <section class="posts" v-if="loaded && posts.length !== 0">
    <div v-for="post in posts" :key="post._id">
      <!-- Pass an offset prop to control the scaling and translation -->
      <CarouselComponent :post="post" :totalItems="5" :offset="400" @refreshPosts="getPosts" @editPost="updateEditing" />
    </div>
  </section>

  <p v-else-if="loaded">No posts found</p>
  <p v-else>Loading...</p>
</template>

<style scoped>
section {
  display: flex;
  flex-direction: column;
  gap: 1em;
}

section,
p,
.row {
  margin: 0 auto;
  max-width: 60em;
}

article {
  background-color: #eeeeee;
  border-radius: 1em;
  display: flex;
  flex-direction: column;
  gap: 0.5em;
  padding: 1em;
  width: 90%; /* Set a responsive width */
  max-width: 40em; /* Ensure it doesn’t grow too large on wide screens */
  margin: 1em auto; /* Center the article and add spacing between articles */
}

.posts {
  padding: 1em;
}

.row {
  display: flex;
  justify-content: space-between;
  margin: 0 auto;
  max-width: 60em;
}
</style>
