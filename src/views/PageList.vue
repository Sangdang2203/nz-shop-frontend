<script setup>
// Post API
import { ref, onMounted } from "vue";
import axios from "../axiosComfig";
// import useNewsStore from "../stores/useNewsStore.js";
// import { storeToRefs } from "pinia";
import getSlugByName from "../utils/getSlugByName.js";
const url = import.meta.env.VITE_PUBLIC_URL;
const pages = ref([]);
// get posts []
const fetchPost = async () => {
  try {
    const response = await axios.get(`pages`);
    // loading.value = false;
    if (response.data.status === 200) {
      pages.value = response.data.data.reverse();
    }
  } catch (error) {
    console.log("Error: ", error);
    // loading.value = false;
  }
};

onMounted(fetchPost);
</script>

<style></style>

<template>
  <v-row>
    <v-col
      :cols="12"
      lg="8"
      md="8"
      v-if="pages.length > 0"
    >
      <h4 class="py-2 text-danger">PageList</h4>
      <v-sheet
        v-for="item in pages"
        :key="item.id"
        class="d-flex mb-3 text-body-1 border rounded-lg w-100"
        rounded="3"
      >
        <div class="d-flex flex-column justify-space-between py-1">
          <RouterLink
            :to="`/page/${getSlugByName(item.name)}`"
            class=""
          >{{ item.name }}</RouterLink>
        </div>
      </v-sheet>
    </v-col>
  </v-row>
</template>