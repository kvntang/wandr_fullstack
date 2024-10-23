<script setup lang="ts">
import CreatePostForm from "../components/Post/CreatePostForm.vue";
import { useUserStore } from "@/stores/user";
import { storeToRefs } from "pinia";
import { fetchy } from "@/utils/fetchy";

const { isLoggedIn } = storeToRefs(useUserStore());

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
</script>

<template>
  <main>
    <h1>Create a post:</h1>
    <section v-if="isLoggedIn">
      <h2>Create a post:</h2>
      <CreatePostForm @refreshPosts="getPosts" />
    </section>
  </main>
</template>

<style scoped>
h1 {
  text-align: center;
}
</style>
