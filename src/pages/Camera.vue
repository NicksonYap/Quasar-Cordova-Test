<template>
  <div>
    <q-btn color="primary" label="Get Picture" @click="captureImage" />

    <img v-if="imageSrc" :src="imageSrc" style="width: 100%; height: 100%">
  </div>
</template>

<script>
export default {
  data () {
    return {
      imageSrc: ''
    }
  },
  methods: {
    captureImage () {
      navigator.camera.getPicture(
        data => { // on success
          this.imageSrc = `data:image/jpeg;base64,${data}`
        },
        () => { // on fail
          this.$q.notify('Could not access device camera.')
        },
        {
          // camera options
          quality: 50,
          destinationType: navigator.camera.DestinationType.DATA_URL,
          encodingType: navigator.camera.DestinationType.JPEG,
          correctOrientation: true,
          saveToPhotoAlbum: true

        }
      )
    }
  }
}
</script>
