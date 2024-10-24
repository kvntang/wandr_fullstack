<template>
  <div class="carousel-container">
    <div class="relative carousel-wrapper">
      <!-- Loop through visible posts and placeholders -->
      <article v-for="(item, index) in feedItems" :key="index" class="carousel-card" :style="calculateStyle(index - centerIndex)">
        <div v-if="index === 0">
          <!-- Show the actual post on the first card -->
          <PostComponent :post="item" @refreshPosts="refreshPosts" @editPost="editPost" />
        </div>
        <div v-else>
          <!-- Placeholder for the rest of the carousel -->
          <div class="placeholder-card">
            <p>More content coming soon...</p>
          </div>
        </div>
      </article>
    </div>

    <!-- Navigation buttons -->
    <div class="carousel-nav">
      <button @click="navigate('left')" class="carousel-button">←</button>
      <button @click="navigate('right')" class="carousel-button">→</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import PostComponent from "@/components/Post/PostComponent.vue";

// Props
const props = defineProps({
  post: Object, // The first post to display
  totalItems: { type: Number, default: 5 }, // Total number of items (1 post + 4 placeholders)
  offset: { type: Number, default: 50 }, // Offset value for scaling/spacing (control passed from parent)
});
const emit = defineEmits(["refreshPosts", "editPost"]);

// Local state
const centerIndex = ref(0);

// Calculate the style for each card in the carousel
const calculateStyle = (relativeIndex: number) => {
  const absIndex = Math.abs(relativeIndex);
  const scale = 1 - (absIndex * props.offset) / 1000;
  const translateX = relativeIndex * (props.offset / 1.5) * Math.pow(1.2, absIndex);
  const zIndex = 5 - absIndex;

  return {
    transform: `translateX(-50%) scale(${scale}) translate(${translateX}px)`, // Add translateX(-50%) to center the card
    zIndex,
    opacity: absIndex > 2 ? 0 : 1,
    transition: "all 0.3s ease-in-out",
  };
};

// Placeholder items array (first one is the post, others are empty)
const feedItems = computed(() => {
  return [props.post, ...Array(props.totalItems - 1).fill(null)];
});

// Navigate left or right in the carousel
const navigate = (direction: "left" | "right") => {
  if (direction === "left" && centerIndex.value > 0) {
    centerIndex.value--;
  } else if (direction === "right" && centerIndex.value < props.totalItems - 1) {
    centerIndex.value++;
  }
};

// Emit events to refresh or edit posts
const refreshPosts = () => emit("refreshPosts");
const editPost = (id: string) => emit("editPost", id);
</script>

<style scoped>
.carousel-container {
  display: flex;
  flex-direction: column;
  gap: 1em;
  width: 100%;
  margin: 0 auto;
}

.carousel-wrapper {
  position: relative;
  width: 100%;
  height: 450px;
  overflow: hidden;
}

.carousel-card {
  position: absolute;
  top: 0;
  left: 50%; /* Set the left position to 50% */
  transform-origin: center;
  background-color: #eeeeee;
  border-radius: 30px;
  width: 300px;
  height: 400px;
  padding: 1em;
  transform: translateX(-50%); /* Adjust position to center card by moving half of its width */
}

.placeholder-card {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 300px;
  background-color: #d2d2d2;
  color: #888;
  font-size: 1.2em;
  border-radius: 30px;
}

.carousel-nav {
  display: flex;
  justify-content: center;
  gap: 1em;
}

.carousel-button {
  background-color: #ddd;
  border: none;
  padding: 0.5em;
  border-radius: 50%;
  cursor: pointer;
  font-size: 1.5em;
}
</style>
