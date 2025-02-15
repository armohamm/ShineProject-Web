<template>
  <modal ref="listBackgroundModal">
    <h6 slot="header" class="mb-0">Change list background</h6>
    <div slot="body" class="p-3">
      <loading-container :is-loading="isSubmitting" message-suffix="photo">
        <p>Upload your own photo</p>
        <error-renderer v-if="statusCode !== 0" :errorMessage="errorMessage"/>
        <div class="input-group mb-3">
          <div class="custom-file">
            <input
              type="file"
              class="custom-file-input"
              v-on:change="fileInputChange($event.target.files)"
            >
            <label class="custom-file-label" for="inputGroupFile01">{{imageFileName}}</label>
          </div>
        </div>
        <hr>
        <p>Or select one of ours</p>
        <div
          class="selectable-picture"
          style="background-image: url('https://shinestorage.azureedge.net/productlistbackgrounds/1.jpg');"
          v-on:click.prevent="selectPicture('https://shinestorage.azureedge.net/productlistbackgrounds/1.jpg')"
        />
        <div
          class="selectable-picture"
          style="background-image: url('https://shinestorage.azureedge.net/productlistbackgrounds/2.jpg');"
          v-on:click.prevent="selectPicture('https://shinestorage.azureedge.net/productlistbackgrounds/2.jpg')"
        />
        <div
          class="selectable-picture"
          style="background-image: url('https://shinestorage.azureedge.net/productlistbackgrounds/3.jpg');"
          v-on:click.prevent="selectPicture('https://shinestorage.azureedge.net/productlistbackgrounds/3.jpg')"
        />
        <div
          class="selectable-picture"
          style="background-image: url('https://shinestorage.azureedge.net/productlistbackgrounds/4.jpg');"
          v-on:click.prevent="selectPicture('https://shinestorage.azureedge.net/productlistbackgrounds/4.jpg')"
        />
        <div
          class="selectable-picture"
          style="background-image: url('https://shinestorage.azureedge.net/productlistbackgrounds/5.jpg');"
          v-on:click.prevent="selectPicture('https://shinestorage.azureedge.net/productlistbackgrounds/5.jpg')"
        />
        <div
          class="selectable-picture"
          style="background-image: url('https://shinestorage.azureedge.net/productlistbackgrounds/6.jpg');"
          v-on:click.prevent="selectPicture('https://shinestorage.azureedge.net/productlistbackgrounds/6.jpg')"
        />
      </loading-container>
    </div>
  </modal>
</template>

<script>
import listService from "@/features/lists/lists.service.js";
import { mapGetters } from "vuex";

export default {
  data() {
    return {
      statusCode: 0,
      imageFileName: "Choose a photo...",
      file: null,
      isSubmitting: false
    };
  },
  computed: {
    ...mapGetters("listsModule", ["currentListId"]),
    errorMessage() {
      switch (this.statusCode) {
        case -1:
          return "Please attach a picture";
        case 500:
          return "Something went wrong, we are investigating";
        case 415:
          return "Please make sure your picture is .PNG, .JPG or .GIF";
        case 413:
          return "Please ensure the picture is 6MB or less";
        default:
          return "Something went wrong, we are investigating";
      }
    }
  },
  methods: {
    selectPicture(imageSource) {
      listService.setCover(this.currentListId, imageSource);
      this.$store.dispatch("listsModule/setBackground", imageSource);
      this.$refs.listBackgroundModal.hide();
    },
    show() {
      this.$refs.listBackgroundModal.show();
    },
    fileInputChange(fileList) {
      this.imageFileName = fileList[0].name;
      this.file = fileList[0];
      this.uploadPhoto();
    },
    uploadPhoto() {
      if (this.isSubmitting) {
        return;
      }

      if (!this.file) {
        this.statusCode = -1;
        return;
      }

      this.isSubmitting = true;
      this.statusCode = 0;

      let formData = new FormData();
      formData.append("file", this.file);

      listService
        .uploadCover(this.currentListId, formData)
        .then(() => {
          this.imageFileName = "Choose a picture...";
          this.isSubmitting = false;
          this.$refs.listBackgroundModal.hide();
          this.$store.dispatch("listsModule/getList", this.currentListId);
        })
        .catch(error => {
          this.statusCode = error.response.status;
          this.isSubmitting = false;
        });

      return false;
    }
  }
};
</script>

<style scoped lang="scss">
.selectable-picture {
  cursor: pointer;
  border: 1px solid transparent;
  display: inline-block;
  width: 265px;
  height: 150px;
  background-size: cover;
  margin-right: 10px;
}

.selectable-picture:hover {
  border: 1px solid $brand-primary-colour;
}

@media (max-width: 500px) {
  .selectable-picture {
    width: 45%;
    height: 100px;
  }
}
</style>