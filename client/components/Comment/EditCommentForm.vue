<script setup lang="ts">
import { ref } from "vue";
import { fetchy } from "../../utils/fetchy";
import { formatDate } from "../../utils/formatDate";

const props = defineProps({
  comment: {
    type: Object,
    required: true,
  },
});
const content = ref(props.comment.content);
const emit = defineEmits(["editComment", "refreshComments"]);

// Function to edit the comment
const editComment = async () => {
  try {
    await fetchy(`/api/comments/${props.comment._id}`, "PATCH", { body: { content: content.value } });
  } catch (e) {
    return;
  }
  emit("editComment");
  emit("refreshComments");
};

// Function to delete the comment
const deleteComment = async () => {
  try {
    await fetchy(`/api/comments/${props.comment._id}`, "DELETE");
    emit("refreshComments");
  } catch (e) {
    console.error("Error deleting comment:", e);
  }
};
</script>

<template>
  <form @submit.prevent="editComment">
    <p class="author">{{ props.comment.author }}</p>
    <textarea id="content" v-model="content" placeholder="Edit your comment" required></textarea>
    <div class="base">
      <menu>
        <li><button class="btn-small pure-button-primary pure-button" type="submit">Save</button></li>
        <li><button class="btn-small pure-button" @click="emit('editComment')">Cancel</button></li>
        <li><button class="btn-small button-error pure-button" @click.prevent="deleteComment">Delete</button></li>
      </menu>
      <p v-if="props.comment.dateCreated !== props.comment.dateUpdated" class="timestamp">Edited on: {{ formatDate(props.comment.dateUpdated) }}</p>
      <p v-else class="timestamp">Created on: {{ formatDate(props.comment.dateCreated) }}</p>
    </div>
  </form>
</template>

<style scoped>
form {
  background-color: #eeeeee;
  display: flex;
  flex-direction: column;
  gap: 0.5em;
  padding: 1em;
  border-radius: 0.5em;
}

textarea {
  font-family: inherit;
  font-size: inherit;
  height: 4em;
  border-radius: 4px;
  resize: none;
}

p {
  margin: 0em;
}

.author {
  font-weight: bold;
  font-size: 1.2em;
}

menu {
  list-style-type: none;
  display: flex;
  flex-direction: row;
  gap: 1em;
  padding: 0;
  margin: 0;
}

.base {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.timestamp {
  display: flex;
  justify-content: flex-end;
  font-size: 0.9em;
  font-style: italic;
}
</style>
