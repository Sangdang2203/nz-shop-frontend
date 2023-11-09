<script setup>
import { ref, onMounted } from "vue";
import axios from "../../axiosComfig";
import { siteData } from "@/stores/globals";
import getSlugByName from "../../utils/getSlugByName";

const siteStore = siteData();
const pages = ref([]);
const filteredPages = ref([]);
const selected = ref(null);

const fetchPage = async () => {
  try {
    siteStore.isLoading = true;
    const response = await axios.get("pages");
    if (response.data.status === 200) {
      pages.value = response.data.data.reverse();
      filteredPages.value = pages.value;
      selected.value = 0;
      siteStore.isLoading = false;
    }
  } catch (error) {
    console.log("Error: ", error);
  }
};

onMounted(fetchPage);
</script>

<template>
  <v-sheet class="mt-10">
    <v-sheet
      class="d-flex justify-space-between py-4 rounded"
      style=""
      :color="siteStore.siteSetings.main_color"
    >
      <div style="width: 30%;" class=" px-3">
        <v-img :src="siteStore.siteSetings.logo_bg"></v-img>
        <p>{{ siteStore.siteSetings.meta_tag_description }}</p>
      </div>

      <v-sheet :color="siteStore.siteSetings.main_color">
        <h3>Thông tin và chính sách</h3>
        <div
          v-for="item in pages"
          :key="item.id"
        >
          <RouterLink
            class="text-white"
            :to="`page/${getSlugByName(item.name)}`"
          >{{ item.name }}
          </RouterLink>
        </div>
      </v-sheet>

      <div style="width: 30%;">
        <h3>Thông tin liên hệ</h3>
        <p>Địa chỉ: {{ siteStore.siteSetings.shop_address }}</p>
        <p>Email: {{ siteStore.siteSetings.shop_email }}</p>
        <p>Chăm sóc KH: {{ siteStore.siteSetings.shop_cskh }}</p>
        <p>Gọi khiếu nại: {{ siteStore.siteSetings.shop_cskhkn }}</p>
        <p>Gọi bảo hành: {{ siteStore.siteSetings.shop_cskhbh }}</p>
        <p>Thời gian làm việc: {{ siteStore.siteSetings.shop_timeWork }}</p>
      </div>
    </v-sheet>
</v-sheet></template>

<style scoped></style>