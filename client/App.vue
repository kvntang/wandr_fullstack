<script setup lang="ts">
import { useToastStore } from "@/stores/toast";
import { useUserStore } from "@/stores/user";
import { watch, computed, onBeforeMount } from "vue";
import { storeToRefs } from "pinia";
import { RouterLink, RouterView, useRoute } from "vue-router";
import StepComponent from "@/components/Step/StepComponent.vue";

// Get the current route for highlighting navigation
const currentRoute = useRoute();
const currentRouteName = computed(() => currentRoute.name);

// Access user data and toast store
const userStore = useUserStore();
const { isLoggedIn, currentUserStep } = storeToRefs(userStore);
const { toast } = storeToRefs(useToastStore());

// Function to compute gradient colors based on user step size (lightness decreases with step size)
const gradientStart = computed(() => {
  const stepSize = Number(currentUserStep.value);
  // Map stepSize to lightness values (90% is bright, 30% is dark)
  const lightness = 90 - (stepSize / 10) * 60; // 90% to 30% lightness range
  return `hsl(200, 30%, ${lightness}%)`; // Keep the hue (200) and saturation (50%) constant, but change lightness
});

const gradientEnd = computed(() => {
  const stepSize = Number(currentUserStep.value);
  // The end of the gradient can be slightly darker or lighter
  const lightness = 40 - (stepSize / 10) * 40; // 70% to 30% lightness range
  return `hsl(200, 30%, ${lightness}%)`; // Same hue and saturation, but darker lightness
});

// Watch for changes in gradientStart and gradientEnd and update CSS variables
watch([gradientStart, gradientEnd], ([newStart, newEnd]) => {
  console.log("New Gradient Start:", newStart); // Debugging log
  console.log("New Gradient End:", newEnd); // Debugging log
  document.documentElement.style.setProperty("--gradient-start", newStart);
  document.documentElement.style.setProperty("--gradient-end", newEnd);
});

// Update the session before mounting the app in case the user is already logged in
onBeforeMount(async () => {
  try {
    await userStore.updateSession();
  } catch {
    // User is not logged in
  }
});
</script>

<template>
  <header>
    <nav>
      <div class="left-section">
        <!-- Only show StepComponent when the user is logged in -->
        <StepComponent v-if="isLoggedIn" />
      </div>

      <div class="center-section">
        <RouterLink :to="{ name: 'Home' }">
          <h1>Wandr.</h1>
        </RouterLink>
      </div>

      <ul class="right-section">
        <li>
          <RouterLink :to="{ name: 'Home' }" :class="{ underline: currentRouteName == 'Home' }"> Home </RouterLink>
        </li>
        <li>
          <RouterLink :to="{ name: 'Search' }" :class="{ underline: currentRouteName == 'Search' }"> Search </RouterLink>
        </li>
        <li>
          <RouterLink :to="{ name: 'Create' }" :class="{ underline: currentRouteName == 'Create' }"> Create </RouterLink>
        </li>
        <li v-if="isLoggedIn">
          <RouterLink :to="{ name: 'Settings' }" :class="{ underline: currentRouteName == 'Settings' }"> Settings </RouterLink>
        </li>
        <li v-else>
          <RouterLink :to="{ name: 'Login' }" :class="{ underline: currentRouteName == 'Login' }"> Login </RouterLink>
        </li>
      </ul>
    </nav>

    <article v-if="toast !== null" class="toast" :class="toast.style">
      <p>{{ toast.message }}</p>
    </article>
  </header>
  <RouterView />
</template>

<style scoped>
@import "./assets/toast.css";

nav {
  padding: 1em 2em;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.left-section {
  display: flex;
  align-items: center;
  flex: 1;
}

.center-section {
  display: flex;
  align-items: center;
  justify-content: center;
  flex: 1;
}

.right-section {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  list-style-type: none;
  gap: 1em;
  flex: 1;
}

h1 {
  font-size: 2em;
  margin: 0;
  color: #4e657d;
}

img {
  height: 2em;
}

a {
  font-size: large;
  color: black;
  text-decoration: none;
}

.underline {
  text-decoration: underline;
}
</style>
