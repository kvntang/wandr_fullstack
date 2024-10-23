<script setup lang="ts">
import SearchPostForm from "../components/post/SearchPostForm.vue";
</script>

<template>
  <main>
    <h1>SEARCH</h1>
    <section>
      <SearchPostForm @getPostsByAuthor="getPosts" />
    </section>
  </main>
</template>

<style scoped>
h1 {
  text-align: center;
}
</style>
