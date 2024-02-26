<template>
  <div class="container mx-auto my-10 max-w-lg">
    <div class="flex justify-center">
      <input type="file"
             multiple
             class="text-sm text-slate-500
                    file:mr-4 file:py-2 file:px-4
                    file:rounded-full file:border-0
                    file:text-sm file:font-semibold
                    file:bg-blue-50 file:text-blue-700
                    hover:file:bg-blue-100"
             id="imageInput"
             @change="handleFileChange">
    </div>
  
    <div class="flex justify-center mt-4">
      <input type="text"
             v-model="zipFileName"
             placeholder="请输入压缩包名称"
             class="mb-4 p-2 border border-gray-300 rounded-md w-full max-w-sm">
    </div>
    <div class="flex justify-center">
      <button @click="compressImages"
              class="mt-4 px-4 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600">压缩图片并打包</button>
    </div>
    <div class="flex justify-center">
      <a v-if="zipFileUrl"
         :href="zipFileUrl"
         :download="zipFileName + '.zip'"
         class="block mt-4 px-4 py-2 bg-green-500 text-white rounded-md hover:bg-green-600">下载压缩包</a>
    </div>

    <div class="flex justify-center">
      <div class="flex flex-wrap w-full">
        <div v-for="(image, index) in images" :key="index" class="w-1/2 p-2">
          <img :src="image.url" :alt="'Image ' + index" class="w-full cursor-pointer" @click="previewImage(index)">
        </div>
      </div>
    </div>
  </div>
  
  <!-- 图片预览模态框 -->
  <show-photo :url="images[previewImageIndex]?.url" :visible="previewImageIndex !== null" @closeClick="closePreview" />
</template>

<script>
import JSZip from 'jszip';
import Compressor from 'compressorjs';
import ShowPhoto from './components/ShowPhoto.vue'; // 请将组件路径调整为正确的路径

export default {
  components: {
    ShowPhoto
  },
  data() {
    return {
      files: [],
      zipFileName: '',
      zipFileUrl: '',
      images: [],
      previewImageIndex: null
    };
  },
  methods: {
    handleFileChange(e) {
      this.files = Array.from(e.target.files);
      this.images = [];
      for (let i = 0; i < this.files.length; i++) {
        const file = this.files[i];
        const reader = new FileReader();
        reader.onload = (e) => {
          this.images.push({ url: e.target.result });
        };
        reader.readAsDataURL(file);
      }
    },
    async compressImages() {
      if (this.files.length === 0) {
        alert('请先选择图片');
        return;
      }
      
      const zip = new JSZip();
      const promises = this.files.map(async (file, index) => {
        const compressedImage = await this.compressImage(file);
        zip.file(file.name, compressedImage, { binary: true });
      });
      await Promise.all(promises);
      
      const zipFile = await zip.generateAsync({ type: 'blob' });
      const url = URL.createObjectURL(zipFile);
      this.zipFileUrl = url;
    },
    async compressImage(file) {
      return new Promise((resolve, reject) => {
        new Compressor(file, {
          quality: 0.6, // 压缩质量
          success(result) {
            resolve(result);
          },
          error(err) {
            reject(err);
          },
        });
      });
    },
    previewImage(index) {
      this.previewImageIndex = index;
    },
    closePreview() {
      this.previewImageIndex = null;
    }
  }
};
</script>
