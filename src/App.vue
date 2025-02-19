<template>
  <div class="flex justify-center align-middle">
    <div class="w-full h-screen flex justify-center items-center" v-if="!isLoad">
      <Button @click="submitFile">Загрузить фото</Button>
      <input
        type="file"
        id="files"
        ref="filesRef"
        accept="image/*"
        style="display: none"
        v-on:change="handleFileUpload"
      />
    </div>
    <div v-else class="!h-[800px] !pb-0 rounded-3xl bg-white w-full !max-w-[1152px] px-11 py-10 sm:p-7 md:rounded-none md:!h-full md:!w-full">
      <PhotoEditor :def-img="file" @save-image="saveImage($event)" @close="file = null, isLoad = false" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
// import HelloWorld from "./components/HelloWorld.vue";
import { useObjectUrl } from "@vueuse/core";
import PhotoEditor from "./components/PhotoEditor.vue";
import Button from "./components/ui/button/Button.vue";

const file = ref<any>();
const isLoad = ref(false)

const filesRef = ref();

function submitFile() {
  filesRef.value.click();
}

function handleFileUpload(e: any) {
  isLoad.value = false
  file.value = useObjectUrl(e.target.files[0]).value;

  e.target.value = "";
  isLoad.value = true
}
const createLinkForDownload = (name: string, url: string) => {
  const link = document.createElement('a');
  link.setAttribute('href', url as string);
  link.setAttribute('download', name);
  link.setAttribute('target', '_blank');
  link.style.display = 'none';
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
};

const saveImage = (url: { src: string }) => {
  console.log(url.src);
  createLinkForDownload('file', url.src as string);
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
