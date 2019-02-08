<template>
  <q-page padding>
    <q-btn color="primary" label="Scan" @click="scan()"/>

    <q-table v-if="scan_results" title="Scan Results" :data="scan_results" :columns="columns"></q-table>
  </q-page>
</template>

<script>
export default {
  data() {
    return {
      scan_results: [],
      columns: [
        {
          name: 'text',
          field: 'text',
          label: 'Text',
          align: 'left',
          sortable: true
        },
        {
          name: 'format',
          field: 'format',
          label: 'Format',
          align: 'left',
          sortable: true
        }
      ]
    };
  },
  methods: {
    scan() {
      cordova.plugins.barcodeScanner.scan(
        result => {
          this.$q.notify({
            icon: 'fullscreen',
            message: `Scanned: ${result.text}`,
            type: 'positive'
          });

          console.log(result);
          this.scan_results.push(result);
        },
        error => {
          // on fail
          this.$q.notify(error);
        },
        {
          preferFrontCamera: false, // iOS and Android
          showFlipCameraButton: true, // iOS and Android
          showTorchButton: true, // iOS and Android
          // torchOn: true, // Android, launch with the torch switched on (if available)
          saveHistory: true, // Android, save scan history (default false)
          prompt: 'Place a barcode inside the scan area', // Android
          resultDisplayDuration: 500, // Android, display scanned text for X ms. 0 suppresses it entirely, default 1500
          // formats : "QR_CODE,DATA_MATRIX,AZTEC", // default: all but PDF_417 and RSS_EXPANDED
          // formats : "QR_CODE,DATA_MATRIX,UPC_A,UPC_E,EAN_8,EAN_13,CODE_39,CODE_93,CODE_128,CODABAR,ITF,RSS14,PDF_417,RSS_EXPANDED,MSI,AZTEC",  //some formats may confict
          formats : "QR_CODE,DATA_MATRIX,EAN_13,PDF_417",
          // orientation : "landscape", // Android only (portrait|landscape), default unset so it rotates with the device
          disableAnimations: true, // iOS
          disableSuccessBeep: false // iOS and Android
        }
      );
    }
  }
};
</script>
