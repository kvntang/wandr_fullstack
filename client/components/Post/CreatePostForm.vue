<!-- <script setup lang="ts">
import { ref } from "vue";
import { fetchy } from "../../utils/fetchy";

const content = ref("");
const emit = defineEmits(["refreshPosts"]);

const createPost = async (content: string) => {
  try {
    await fetchy("/api/posts", "POST", {
      body: { content },
    });
  } catch (_) {
    return;
  }
  emit("refreshPosts");
  emptyForm();
};

const emptyForm = () => {
  content.value = "";
};
</script> -->

<script setup lang="ts">
import { ref } from "vue";
import { fetchy } from "../../utils/fetchy";

// Form input fields
const content = ref("");
const photo = ref<File | null>(null);
const emit = defineEmits(["refreshPosts"]);

// Helper function to convert file to base64
const fileToBase64 = (file: File): Promise<string> => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = () => resolve(reader.result as string);
    reader.onerror = (error) => reject(error);
  });
};

// Function to create a post with content and an optional image
const createPost = async (content: string, photo: File | null) => {
  let base64Photo = null;

  // Convert the photo to base64 if it exists
  if (photo) {
    try {
      base64Photo = await fileToBase64(photo);
    } catch (error) {
      console.error("Error converting image to base64:", error);
      return;
    }
  }

  // Send post data including the base64 image (if any)
  try {
    await fetchy("/api/posts", "POST", {
      body: { content, photo: base64Photo }, // Add photo to the post data
    });
  } catch (error) {
    console.error("Error creating post:", error);
    return;
  }

  // Emit event to refresh posts after submission
  emit("refreshPosts");

  // Reset the form
  emptyForm();
};

// Function to reset the form fields
const emptyForm = () => {
  content.value = "";
  photo.value = null;
};
</script>

<!-- <template>
  <form @submit.prevent="createPost(content)">
    <label for="content">Post Contents:</label>
    <textarea id="content" v-model="content" placeholder="Create a post!" required> </textarea>
    <button type="submit" class="pure-button-primary pure-button">Create Post</button>
  </form>
</template> -->

<template>
  <form @submit.prevent="createPost(content, photo)">
    <label for="content">Post Contents:</label>
    <textarea id="content" v-model="content" placeholder="Create a post!" required></textarea>

    <label for="photo">Upload Image:</label>
    <input id="photo" type="file" accept="image/*" @change="photo = $event.target.files[0]" />

    <button type="submit" class="pure-button-primary pure-button">Create Post</button>
  </form>
</template>

<style scoped>
form {
  background-color: var(--base-bg);
  border-radius: 1em;
  display: flex;
  flex-direction: column;
  gap: 0.5em;
  padding: 1em;
}

textarea {
  font-family: inherit;
  font-size: inherit;
  height: 6em;
  padding: 0.5em;
  border-radius: 4px;
  resize: none;
}
</style>
