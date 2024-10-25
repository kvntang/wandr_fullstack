<script setup lang="ts">
// Define props to accept the comment object from the parent component
const props = defineProps(["comment"]);
</script>

<template>
  <div class="comment-container">
    <!-- Display the comment content and author on the same line -->
    <div class="comment-content">
      <span class="comment-author">{{ props.comment.author }}</span>
      <p>{{ props.comment.content }}</p>
    </div>
  </div>
</template>

<style scoped>
.comment-container {
  background-color: #f5f5f5;
  padding: 1em;
  border-radius: 8px;
  margin-bottom: 1em;
}

.comment-content {
  display: flex;
  justify-content: flex-start; /* Align items to the start */
  align-items: center; /* Vertically center the text */
  gap: 10px; /* Adjust the space between the comment and the author */
}

.comment-content p {
  flex-grow: 1; /* Ensure the comment takes up as much space as needed */
  margin: 0; /* Remove any default margins */
}

.comment-author {
  font-weight: bold;
}
</style>
