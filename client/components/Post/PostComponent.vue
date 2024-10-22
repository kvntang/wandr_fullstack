<script setup lang="ts">
import { ref, onMounted } from "vue";
import { useUserStore } from "@/stores/user";
import { formatDate } from "@/utils/formatDate";
import { storeToRefs } from "pinia";
import { fetchy } from "../../utils/fetchy";

const props = defineProps(["post"]);
const emit = defineEmits(["editPost", "refreshPosts"]);
const { currentUsername } = storeToRefs(useUserStore());

const deletePost = async () => {
  try {
    await fetchy(`/api/posts/${props.post._id}`, "DELETE");
  } catch {
    return;
  }
  emit("refreshPosts");
};

// Compute the image source from the base64 string
const imageSrc = ref<string | null>(null);
// Function to handle the base64 image creation
async function createImageFromBase64(base64ImageData: string): Promise<string | null> {
  if (!base64ImageData) return null;

  if (!base64ImageData.startsWith("data:image")) {
    base64ImageData = `data:image/jpeg;base64,${base64ImageData}`;
  }

  const img = new Image();
  img.src = base64ImageData;

  await new Promise<void>((resolve, reject) => {
    img.onload = () => resolve();
    img.onerror = (error) => reject(error);
  });

  return base64ImageData;
}

onMounted(async () => {
  if (props.post.photo) {
    try {
      imageSrc.value = await createImageFromBase64(props.post.photo);
    } catch (error) {
      console.error("Error setting image source:", error);
    }
  }
});
</script>

<template>
  <p class="author">{{ props.post.author }}</p>
  <p>{{ props.post.content }}</p>

  <!-- Lazy load the image -->
  <div v-if="imageSrc">
    <img :src="imageSrc" alt="Uploaded Image" loading="lazy" class="responsive-image" />
  </div>
  <p v-else>No photo available.</p>

  <div class="base">
    <menu v-if="props.post.author == currentUsername">
      <li><button class="btn-small pure-button" @click="emit('editPost', props.post._id)">Edit</button></li>
      <li><button class="button-error btn-small pure-button" @click="deletePost">Delete</button></li>
    </menu>

    <article class="timestamp">
      <p v-if="props.post.dateCreated !== props.post.dateUpdated">Edited on: {{ formatDate(props.post.dateUpdated) }}</p>
      <p v-else>Created on: {{ formatDate(props.post.dateCreated) }}</p>
    </article>
  </div>
</template>

<style scoped>
.responsive-image {
  display: block;
  margin: 0 auto;
  width: 100%; /* Ensure it takes up full width of the container */
  max-width: 500px; /* Set maximum width */
  height: 300px; /* Set a fixed height for all images */
  object-fit: cover; /* Crop the image to fit within the container */
  border-radius: 8px; /* Optional: Add rounded corners */
}

p {
  margin: 0em;
}

.author {
  font-weight: bold;
  font-size: 1.5em;
}

menu {
  list-style-type: none;
  display: flex;
  flex-direction: row;
  gap: 1em;
  padding: 0;
  margin: 0;
}

.timestamp {
  display: flex;
  justify-content: flex-end;
  font-size: 0.9em;
  font-style: italic;
}

.base {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.base article:only-child {
  margin-left: auto;
}
</style>
