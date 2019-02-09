<template>
  <q-page padding>
    <q-btn color="primary" label="Scan" @click="scan()"/>

    <q-toggle
      v-model="torch_on"
      color="amber"
      label="Torch"
      checked-icon="brightness_high"
      unchecked-icon="brightness_low"
      style="margin-left: 25px"
    />
    <br>
    <q-select
      float-label="Formats"
      multiple
      v-model="format_selection"
      :options="format_options"
      style="overflow:hidden"
    />
    <br>
    <q-table
      v-if="scan_results"
      title="Scan Results"
      :data="scan_results"
      :columns="columns"
      :pagination.sync="pagination"
    ></q-table>
  </q-page>
</template>

<script>
export default {
  data() {
    return {
      scan_properly_cancelled: true,
      torch_on: false,
      scan_results: [],
      format_selection: ['QR_CODE', 'DATA_MATRIX', 'EAN_13', 'PDF_417'],
      // format_selection: [],
      format_options: [
        {
          label: 'QR_CODE',
          value: 'QR_CODE'
        },
        {
          label: 'DATA_MATRIX',
          value: 'DATA_MATRIX'
        },
        {
          label: 'EAN_13',
          value: 'EAN_13'
        },
        {
          label: 'EAN_8',
          value: 'EAN_8'
        },
        {
          label: 'UPC_A',
          value: 'UPC_A'
        },
        {
          label: 'UPC_E',
          value: 'UPC_E'
        },
        {
          label: 'PDF_417',
          value: 'PDF_417'
        }
      ],
      columns: [
        {
          name: 'index',
          label: 'No.',
          align: 'right',
          field: row => row.__index,
          format: val => val + 1,
          sortable: true
        },
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
      ],

      pagination: {
        sortBy: 'index', // String, column "name" property value
        descending: true
      }
    };
  },
  methods: {
    scan() {
      if (!this.format_selection.length) {
        this.$q.notify({
          icon: 'fullscreen',
          message: `No Scan Format Selected`,
          type: 'negative'
        });
        return;
      }

      console.log(this.format_selection.join());

      this.scan_properly_cancelled = false;

      cordova.plugins.barcodeScanner.scan(
        result => {
          if (result.cancelled == false) {
            this.scan_properly_cancelled = true;
            this.$q.notify({
              icon: 'fullscreen',
              message: `Scanned: ${result.text}`,
              type: 'positive'
            });

            console.log(result);
            this.scan_results.push(result);
          } else {
            this.$q.notify({
              icon: 'fullscreen',
              message: `User Cancelled`,
              type: 'negative'
            });
          }
        },
        error => {
          // on fail
          this.scan_properly_cancelled = true;
          this.$q.notify(error);
        },
        {
          preferFrontCamera: false, // iOS and Android
          showFlipCameraButton: true, // iOS and Android
          showTorchButton: true, // iOS and Android
          torchOn: this.torch_on, // Android, launch with the torch switched on (if available)
          // saveHistory: true, // Android, save scan history (default false)
          // prompt: 'Place a barcode inside the scan area', // Android
          prompt: '', // Android
          resultDisplayDuration: 500, // Android, display scanned text for X ms. 0 suppresses it entirely, default 1500
          // formats : "QR_CODE,DATA_MATRIX,AZTEC", // default: all but PDF_417 and RSS_EXPANDED
          // formats : "QR_CODE,DATA_MATRIX,UPC_A,UPC_E,EAN_8,EAN_13,CODE_39,CODE_93,CODE_128,CODABAR,ITF,RSS14,PDF_417,RSS_EXPANDED,MSI,AZTEC",  //some formats may confict
          formats: this.format_selection.join(),
          // orientation : "landscape", // Android only (portrait|landscape), default unset so it rotates with the device
          disableAnimations: true, // iOS
          disableSuccessBeep: false // iOS and Android
        }
      );
    }
  },
  beforeRouteLeave(to, from, next) {
    // called when the route that renders this component is about to
    // be navigated away from.
    // has access to `this` component instance.

    if (this.scan_properly_cancelled) {
      next();
    } else {
      next(false);
    }
    this.scan_properly_cancelled = true;
  }
};
</script>
