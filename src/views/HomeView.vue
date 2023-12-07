<template>
  <main>
    <div class="container">
      <h2 class="text-center mt-3 mb-3">Image Gallery</h2>
      <!-- <div class="alert" role="alert" :class="{'alert-danger': isError, 'alert-success': isSuccess}" v-if="message" style="animation:fade-out 0.5s ease-in-out">{{ message }}</div> -->
      <p>Upload Image</p>
      <div class="input-group mb-3">
        <input type="file" class="form-control" id="inputGroupFile01" @change="handleFileChange" ref="fileInput">
        <button class="btn btn-primary" @click="uploadImage">Upload</button>
      </div>
      <section>
        <div class="row">
          <div class="col-3 col-lg-3 col-md-3 mb-4" v-for="img,i in uploadedImages" :key="`img-${i}`">
            <img :src="img" alt="Uploaded image" class="w-100">
          </div>
        </div>
      </section>
    </div>
  </main>
</template>
<script>
import axios from 'axios';
import { useToast } from "vue-toastification";

export default {
  data() {
    return{
      image: null,
      imageUrl: null,
      uploadedImages: JSON.parse(localStorage.getItem('uploadedImages')) || [],
    }
  },
  methods: {
    handleFileChange(event) {
      this.image = event.target.files[0];
    },
    async uploadImage() {
      const toast = useToast();
      try {
        if (!this.image) {
          toast.error("Pilih gambar terlebih dahulu");
          return;
        }

        if (typeof this.image !== 'object' || typeof this.image.size === 'undefined') {
          toast.error("File yang dipilih tidak valid.");
          return;
        }

        if (this.image.size > 2 * 1024 * 1024) {
          toast.error("Ukuran gambar terlalu besar. Maksimal 2MB!");
          return;
        }

        const formatImage = ['image/jpeg', 'image/jpg', 'image/png', 'image/bmp', 'image/gif'];
        if (!formatImage.includes(this.image.type)) {
          toast.error("Format gambar tidak valid, Gambar yang diupload harus berupa .jpg, .jpeg, .png, .bmp, atau .gif.");
          return;
        }

        const formData = new FormData();
        formData.append('image', this.image);

        const response = await axios.post('https://api.imgbb.com/1/upload', formData, {
          params: {
            key: '175601a98e210d2e2b4632202b581126',
          },
        });

        this.imageUrl = response.data.data.url;
        this.uploadedImages.push(this.imageUrl);

        localStorage.setItem('uploadedImages', JSON.stringify(this.uploadedImages));
        
        toast.success("Gambar berhasil diunggah");
        this.$refs.fileInput.value = '';

      } catch (error) {
        toast.success("Error saat mengunggah gambar. Silakan coba lagi.");
      }
    },
  },
}
</script>
<style>

</style>
