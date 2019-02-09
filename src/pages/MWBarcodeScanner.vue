<template>
  <q-page padding>
    <div class="row">
      <div class="auto">
        <q-btn color="primary" label="Scan" @click="scan()"/>
      </div>
    </div>

    <br>
    <div class="row">
      <div class="auto">
        <q-toggle
          v-model="options.torch_on"
          color="amber"
          label="Torch"
          checked-icon="brightness_high"
          unchecked-icon="brightness_low"
          style="margin-left: 25px"
        />
      </div>
      <div class="auto">
        <q-toggle
          v-model="options.use_front_camera"
          color="secondary"
          label="Front"
          checked-icon="brightness_high"
          unchecked-icon="brightness_low"
          style="margin-left: 25px"
        />
      </div>
      <div class="auto">
        <q-toggle
          v-model="options.high_performance"
          color="secondary"
          label="HP"
          checked-icon="brightness_high"
          unchecked-icon="brightness_low"
          style="margin-left: 25px"
        />
      </div>
    </div>

    <br>
    <q-select
      float-label="Formats"
      multiple
      v-model="options.formats"
      :options="format_options"
      style="overflow:hidden"
    />
    <br>
    <q-table v-if="scan_results" title="Scan Results" :data="scan_results" :columns="columns"></q-table>
  </q-page>
</template>

<script>
export default {
  data() {
    return {
      scan_properly_cancelled: true,
      options: {
        torch_on: true,
        use_front_camera: false,
        high_performance: true,
        formats: [
          'MWB_CODE_MASK_QR',
          'MWB_CODE_MASK_DM',
          'MWB_CODE_MASK_EANUPC',
          'MWB_CODE_MASK_PDF'
        ]
      },
      scan_results: [],
      // format_selection: [],
      format_options: [
        {
          label: 'QR Code',
          value: 'MWB_CODE_MASK_QR'
        },
        {
          label: 'DATA_MATRIX',
          value: 'MWB_CODE_MASK_DM'
        },
        {
          label: 'UPC & EAN',
          value: 'MWB_CODE_MASK_EANUPC'
        },
        {
          label: 'PDF417',
          value: 'MWB_CODE_MASK_PDF'
        }
      ],
      columns: [
        {
          name: 'code',
          field: 'code',
          label: 'Code',
          align: 'left',
          sortable: true
        },
        {
          name: 'type',
          field: 'type',
          label: 'Type',
          align: 'left',
          sortable: true
        }
      ]
    };
  },
  methods: {
    initMwbScanner(options = {}) {
      console.log(options);

      var mw_c = mwbScanner.getConstants();

      let active_codes = false;

      options.formats.forEach(format => {
        active_codes |= mw_c[format];
        console.log(format);
      });

      /* active_codes =
        mw_c.MWB_CODE_MASK_QR |
        mw_c.MWB_CODE_MASK_DM |
        mw_c.MWB_CODE_MASK_EANUPC |
        mw_c.MWB_CODE_MASK_PDF; */

      console.log(active_codes);

      var settings = [
        {
          method: 'MWBsetActiveCodes',
          value: [active_codes]
        },
        { method: 'MWBenableZoom', value: [true] },
        {
          method: 'MWBturnFlashOn',
          value: [options.torch_on || false]
        },
        {
          method: 'MWBuseFrontCamera',
          value: [options.use_front_camera || false]
        },
        {
          method: 'MWBsetInterfaceOrientation',
          // value: [mw_c.OrientationLandscapeLeft]
          value: [mw_c.OrientationPortrait]
        },
        {
          method: 'MWBsetDirection',
          // value: [mw_c.MWB_SCANDIRECTION_VERTICAL]
          // value: [mw_c.MWB_SCANDIRECTION_OMNI | mw_c.MWB_SCANDIRECTION_AUTODETECT]
          value: [mw_c.MWB_SCANDIRECTION_AUTODETECT]
        },
        { method: 'MWBsetOverlayMode', value: [mw_c.OverlayModeMW] }
      ];

      if (options.high_performance == true) {
        settings.push(
          { method: 'MWBsetLevel', value: [5] }, //max effort
          { method: 'MWBenableHiRes', value: [true] },
          { method: 'MWBsetMaxThreads', value: [999] } //mx out threads
        );
      } else {
        settings.push(
          { method: 'MWBsetLevel', value: [3] }, //max effort
          { method: 'MWBenableHiRes', value: [true] },
          { method: 'MWBsetMaxThreads', value: [2] } //mx out threads
        );
      }

      var keys = {
        Android: '5VaKlRNBzlaQlEEiLsSsUaOLaOzMM/kkM2I+uQ12OUM=',
        iOS: 'sTRdWE0JYpm2BVsZzEQNZDTx34RH2kaDBMt/h3sAvYM=',
        Win32NT: 'VALID_WIN_WP8_KEY',
        windows: 'VALID_WIN_10_UWP_KEY'
      };
      var key = keys[device.platform] ? keys[device.platform] : '';

      return mwbScanner.setKey(key).then(response => {
        return mwbScanner
          .loadSettings(settings)
          .then(response => {})
          .catch(reason => {
            console.error(reason);
          });
      });
    },
    scan() {
      if (!this.options.formats.length) {
        this.$q.notify({
          icon: 'fullscreen',
          message: `No Scan Format Selected`,
          type: 'negative'
        });
        return;
      }

      this.initMwbScanner(this.options);

      mwbScanner.startScanning(result => {
        console.log(result);
        if (result.type !== 'Cancel') {
          this.$q.notify({
            icon: 'fullscreen',
            message: `Scanned: ${result.code}`,
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
      });
    }
  }
};
</script>
