<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted, nextTick } from "vue";
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
const carouselHeight = ref(0);

// Function to calculate the height of the tallest card
const updateCarouselHeight = async () => {
  await nextTick(); // Wait for the DOM to update
  const cards = document.querySelectorAll(".carousel-card");
  const tallestHeight = Array.from(cards).reduce((maxHeight, card) => {
    const cardHeight = card.scrollHeight; // Get the full height of each card
    return Math.max(maxHeight, cardHeight);
  }, 0);
  carouselHeight.value = tallestHeight; // Set the tallest card's height
};

// Calculate the style for each card in the carousel
const calculateStyle = (relativeIndex) => {
  const absIndex = Math.abs(relativeIndex);
  const scale = 1 - (absIndex * props.offset) / 1000;
  const translateX = relativeIndex * 1.5 * (props.offset / 1.1) * Math.pow(1.5, absIndex);
  const zIndex = 5 - absIndex;

  // Apply more blur the further the card is from the center
  const blurAmount = absIndex > 0 ? `blur(${absIndex * 10}px)` : "none";

  return {
    transform: `translateX(-50%) scale(${scale}) translate(${translateX}px)`, // Center and translate
    zIndex,
    opacity: absIndex > 2 ? 0 : 1,
    filter: blurAmount, // Apply blur effect
    transition: "all 0.3s ease-in-out",
  };
};

// Placeholder items array (first one is the post, others are empty)
const feedItems = computed(() => {
  return [props.post, ...Array(props.totalItems - 1).fill(null)];
});

// Navigate left or right in the carousel
const navigate = (direction) => {
  if (direction === "left" && centerIndex.value > 0) {
    centerIndex.value--;
  } else if (direction === "right" && centerIndex.value < props.totalItems - 1) {
    centerIndex.value++;
  }
  updateCarouselHeight(); // Update the height on navigation
};

// Emit events to refresh or edit posts
const refreshPosts = () => emit("refreshPosts");
const editPost = (id) => emit("editPost", id);

onMounted(() => {
  const observer = new MutationObserver(() => {
    updateCarouselHeight(); // Trigger height recalculation when DOM changes
  });

  // Observe changes in the carousel-wrapper or its children
  const carouselWrapper = document.querySelector(".carousel-wrapper");
  if (carouselWrapper) {
    observer.observe(carouselWrapper, { childList: true, subtree: true, attributes: true });
  }

  updateCarouselHeight(); // Initial height calculation
});

onUnmounted(() => {
  if (observer) {
    observer.disconnect(); // Clean up observer when the component is destroyed
  }
});
</script>

<template>
  <div class="carousel-container">
    <div class="relative carousel-wrapper" :style="{ height: `${carouselHeight}px` }">
      <!-- Loop through visible posts and placeholders -->
      <article v-for="(item, index) in feedItems" :key="index" class="carousel-card" ref="carouselCardRefs" @mounted="updateCarouselHeightOnCardLoad" :style="calculateStyle(index - centerIndex)">
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

      <!-- Navigation buttons positioned on the left and right of the center card -->
      <button @click="navigate('left')" class="carousel-button left-button">←</button>
      <button @click="navigate('right')" class="carousel-button right-button">→</button>
    </div>
  </div>
</template>

<style scoped>
.carousel-container {
  display: flex;
  flex-direction: column;
  gap: 1em;
  width: 100%;
  margin: 0 auto;
  height: auto;
  padding-bottom: 2em;
}

.carousel-wrapper {
  position: relative;
  width: 100%;
}

.carousel-card {
  position: absolute;
  top: 0;
  left: 50%; /* Set the left position to 50% */
  transform-origin: center;
  background-color: #eeeeee;
  border-radius: 30px;
  width: 300px;
  padding: 1em;
  transform: translateX(-50%); /* Adjust position to center card by moving half of its width */
  transition: height 0.3s ease; /* Smooth transition when height changes */
}

.placeholder-card {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  min-height: 300px;
  background-color: #d2d2d2;
  color: #888;
  font-size: 1.2em;
  border-radius: 30px;
}

.carousel-button {
  background-color: #ddd;
  border: none;
  padding: 0.5em;
  border-radius: 50%;
  cursor: pointer;
  font-size: 1.5em;
  z-index: 10;
  position: absolute;
  top: 50%; /* Vertically center the buttons */
  transform: translateY(-50%);
}

.left-button {
  left: 20%;
}

.right-button {
  right: 20%;
}
</style>
