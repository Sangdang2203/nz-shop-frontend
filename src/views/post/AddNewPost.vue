<script setup>
import { ref } from "vue";
import axios from "../../axiosComfig";
import { useRouter } from "vue-router";
import ContentEditor from "../../components/globals/ContentEditor.vue";
import { Cropper } from "vue-advanced-cropper";
import "vue-advanced-cropper/dist/style.css";
import { siteData } from "@/stores/globals";
const siteStore = siteData();

//------ Upload & crop ảnh ----------//
const callModel = ref(false);
const fileInput = ref(null);
const cropperArea = ref(null);
const uploadedImage = ref(null);
const croppedImageData = ref();
const newImage = ref();
const getUploadedImage = (e) => {
  const file = e.target.files[0];
  uploadedImage.value = URL.createObjectURL(file);
};
const crop = () => {
  const { canvas } = cropperArea.value.getResult();
  croppedImageData.value = canvas.toDataURL();
};
const done = () => {
  // nhiu tac vu khac
  newImage.value = croppedImageData.value;
  callModel.value = false;
};

//------ Create a new post ----------//
const router = useRouter();
const newPost = ref({
  title: "",
  author: "",
  image: [],
  description: "",
  content: "",
  type: "",
});

async function createPost() {
  const formData = new FormData();
  Object.entries(newPost.value).forEach(([key, value]) => {
    if (key === "image") {
      formData.append(key, value[0]);
    }
    else {
      formData.append(key, value);
    }
  });
  try {
    const response = await axios.post("posts", {
      title: newPost.value.title,
      author: newPost.value.author,
      image: newImage.value,
      description: newPost.value.description,
      content: newPost.value.content,
      type: newPost.value.type,
    });
    if (response.data.status === 201) {
      router.push("/admincp/post");
      siteStore.hasRes({ data: { status: "ok", message: "Tạo mới thành công." } });
    }
  } catch (e) {
    siteStore.hasRes({ data: { status: "error", message: "Tạo mới thất bại." } });
    console.log("error", e);
  }
};

//------ Create content by AI technology ----------//
const contentByAIModal = ref(false);
const contentByAI = ref({
  title: "",
  description: "",
});
const validateContentByAI = ref([
  description => {
    if (description.length > 9) { return true; }
    return "Nhập tối thiểu 10 ký tự.";
  },
  title => {
    if (title.length > 9) { return true; }
    return "Nhập tối thiểu 10 ký tự.";
  },
]);

async function createContentByAI() {
  try {
    siteStore.isLoading = true;
    const response = await axios.post("content", {
      name: contentByAI.value.title,
      description: contentByAI.value.description,
    });

    if (response.status === 200) {
      newPost.value.content = response.data.data;
      siteStore.isLoading = false;
      contentByAIModal.value = false;
      siteStore.hasRes({ data: { status: "ok", message: "Tạo nội dung thành công." } });
    }
  } catch (error) {
    console.log(error);
    siteStore.hasRes({ data: { status: "error", message: "Tạo nội dung thất bại." } });
  }
};

function editContent(event) {
  newPost.value.content = event;
};
</script>

<style></style>

<template>
  <v-form
    ref="form"
    @submit.prevent="createPost"
  >
    <v-container class="m-card my-3">
      <v-row>
        <v-col
          cols="12"
          md="12"
        >
          <h2>Chi tiết bài viết mới</h2>
        </v-col>
      </v-row>
      <v-row>
        <v-col
          cols="12"
          md="12"
        >
          <v-text-field
            variant="outlined"
            v-model="newPost.title"
            :rules="[v => !!v || 'Vui lòng không để trống.']"
            :counter="20"
            label="Tiêu đề bài viết:"
          >
          </v-text-field>
        </v-col>
      </v-row>
      <v-row>
        <v-col
          cols="12"
          md="12"
        >
          <v-text-field
            variant="outlined"
            v-model="newPost.author"
            :rules="[v => !!v || 'Vui lòng không để trống.']"
            :counter="20"
            label="Tác giả:"
          >
          </v-text-field>
        </v-col>
      </v-row>
      <v-row>
        <v-col>
          <v-btn
            width=""
            @click="callModel = !callModel"
          >Upload ảnh</v-btn>
        </v-col>
      </v-row>

      <v-row>
        <v-col
          cols="12"
          md="12"
          class="d-flex align-center"
        >
          <v-img
            v-if="newImage"
            :src="newImage"
            class="rounded-lg"
          ></v-img>
          <v-img
            v-else
            src="https://dummyimage.com/930x300/dddddd/cd3545&text=Upload+Image"
            class="rounded-lg"
          ></v-img>
        </v-col>
      </v-row>
      <v-row>
        <v-col
          cols="12"
          md="12"
        >
          <v-select
            variant="outlined"
            v-model="newPost.type"
            :rules="[v => !!v || 'Vui lòng lựa chọn.']"
            label="Loại tin tức:"
            :items="['Tin sản phẩm', 'Tin thị trường']"
          ></v-select>
        </v-col>
      </v-row>
      <v-row>
        <v-col
          cols="12"
          md="12"
        >
          <v-label class="text-caption">Mô tả bài viết:</v-label>
          <v-textarea
            name="editor"
            variant="outlined"
            v-model="newPost.description"
            :rules="[v => !!v || 'Vui lòng không để trống']"
          >
          </v-textarea>
        </v-col>
      </v-row>

      <v-row>
        <v-col
          cols="12"
          md="12"
        >
          <div class="d-flex justify-space-between my-2">
            <v-label class="text-caption">Nội dung bài viết</v-label>
            <v-btn @click="contentByAIModal = !contentByAIModal">Sử dụng công nghệ AI</v-btn>
          </div>
          <ContentEditor
            :editorContent="newPost.content"
            @editContent="editContent"
            :rules="[v => !!v || 'Vui lòng không để trống']"
          />
        </v-col>
      </v-row>
      <v-row>
        <v-col
          cols="12"
          md="12"
        >
          <v-btn
            class="me-2"
            type="submit"
          >Hoàn tất</v-btn>
          <v-btn
            :to="`/admincp/post`"
            type="reset"
          >Hủy bỏ</v-btn>
        </v-col>
      </v-row>
    </v-container>
  </v-form>

  <!-- Modal xử lý ảnh -->
  <v-dialog
    v-model="callModel"
    width="auto"
  >
    <v-card class="rounded w-50 h-25 mx-auto">
      <v-file-input
        chips
        id="image"
        ref="fileInput"
        prepend-inner-icon="mdi-image-outline"
        prepend-icon=""
        label="Upload ảnh:"
        @change="getUploadedImage"
        class="d-flex flex-column justify-center ma-3"
      />
      <Cropper
        ref="cropperArea"
        :src="uploadedImage"
        :stencil-props="{
          aspectRatio: 1000 / 450,
        }"
        :canvas="{
          width: 1000,
          height: 450
        }"
      />
      <br>
      <div>
        <v-img
          :src="croppedImageData"
          width="1000"
          height="450"
        ></v-img>
      </div>
      <div class="d-flex justify-center my-5">
        <v-btn
          v-if="uploadedImage"
          @click="crop"
          append-icon="mdi-content-cut"
          variant="elevated"
          class="w-25 pa-2 border-double rounded-lg mx-1"
        >
          CẮT ẢNH
        </v-btn>

        <v-btn
          v-if="croppedImageData"
          @click="done"
          color="success"
          append-icon="mdi-check"
          variant="elevated"
          class="w-25 pa-2 border-double rounded-lg mx-1"
        >
          XÁC NHẬN
        </v-btn>

        <v-btn
          color="#c50000"
          variant="elevated"
          class="w-25 pa-2 border-double rounded-lg mx-1"
          append-icon="mdi-close-outline"
          @click="callModel = false"
        >ĐÓNG
        </v-btn>
      </div>
    </v-card>
  </v-dialog>

  <!-- Modal tạo content bằng AI -->
  <v-dialog
    v-model="contentByAIModal"
    class="w-50"
  ><v-card class="pa-4">
      <v-text-field
        v-model="contentByAI.title"
        variant="outlined"
        :counter="100"
        label="Tiêu đề bài viết:"
        :rules="validateContentByAI"
      ></v-text-field>

      <v-textarea
        v-model="contentByAI.description"
        variant="outlined"
        placeholder="Tóm tắt nội dung: "
        :rules="validateContentByAI"
      ></v-textarea>

      <div class="d-flex justify-center">
        <v-btn
          @click="createContentByAI"
          color="success"
          class="mx-1"
        >Xác nhận</v-btn>
        <v-btn
          @click="contentByAIModal = !contentByAIModal"
          color="#d50000"
          class="mx-1"
        >Đóng</v-btn>
      </div>
    </v-card>
  </v-dialog>
</template>
