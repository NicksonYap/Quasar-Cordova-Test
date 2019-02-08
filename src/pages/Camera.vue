<template>
  <q-page padding>
    <q-btn color="primary" label="Get Picture" @click="captureImage"/>

    <img v-if="imageSrc" :src="imageSrc" style="width: 100%; height: 100%">
  </q-page>
</template>

<script>
export default {
  data() {
    return {
      imageSrc: ''
    };
  },
  methods: {
    captureImage() {
      navigator.camera.getPicture(
        data => {
          // on success
          this.imageSrc = `data:image/jpeg;base64,${data}`;
        },
        error => {
          // on fail
          this.$q.notify(error);
        },
        {
          // camera options
          quality: 50,
          destinationType: navigator.camera.DestinationType.DATA_URL,
          encodingType: navigator.camera.DestinationType.JPEG,
          correctOrientation: true,
          saveToPhotoAlbum: true
        }
      );
    }
  }
};
</script>
