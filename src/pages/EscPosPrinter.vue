<template>
  <q-page padding>
    <q-btn
      color="primary"
      icon="bluetooth_searching"
      label="List Paired Devices"
      @click="tryListPaired()"
    />
    <q-table v-if="devices" title="Devices" :data="devices" :columns="columns" row-key="id">
      <!-- slot name syntax: body-cell-<column_name> -->
      <q-td slot="body-cell-connect" slot-scope="props" :props="props">
        <q-btn round color="positive" @click="tryConnect(props.row)">
          <q-icon name="link"/>
        </q-btn>
      </q-td>
    </q-table>
  </q-page>
</template>

<script>
export default {
  data() {
    return {
      devices: null,
      columns: [
        {
          name: 'name',
          field: 'name',
          label: 'Name',
          align: 'left',
          sortable: true
        },
        {
          name: 'id',
          label: 'ID',
          field: 'id',
          align: 'left',
          sortable: true
        },
        {
          name: 'connect',
          label: 'ID'
        }
      ]
    };
  },
  methods: {
    tryListPaired() {
      window.bluetoothSerial.isEnabled(
        isEnabled => {
          // on success
          console.log(isEnabled);
          this.listPaired();
        },
        error => {
          // on fail
          this.$q.notify({
            icon: 'bluetooth_disabled',
            message: `Not Connected`,
            type: 'negative'
          });
          window.bluetoothSerial.enable(
            success => {
              this.$q.notify({
                icon: 'bluetooth_connected',
                message: `Connected`,
                type: 'positive'
              });
              this.listPaired();
            },
            error => {
              this.$q.notify({
                icon: 'cancel_presentation',
                message: `User Rejected`,
                type: 'warning'
              });
            }
          );
        }
      );
    },
    listPaired() {
      window.bluetoothSerial.list(
        devices => {
          console.log(devices);
          this.devices = devices;
        },
        error => {
          this.$q.notify(error);
        }
      );
    },
    tryConnect(device) {
      window.bluetoothSerial.disconnect(
        success => {
          this.connect(device);
          console.log('disconnect success');
          //   this.$q.notify('Disconnected');
        },
        error => {
          console.log('disconnect failed');
          this.$q.notify(error);
        }
      );

      /*  window.bluetoothSerial.isConnected(
        success => {
          this.$q.notify('Already connected');
        },
        error => {
          this.$q.notify(error);
          this.connect(device);
        }
      ); */
    },
    connect(device) {
      let device_id = device.id;
      let device_name = device.name;

      this.$q.notify({
        icon: 'bluetooth_searching',
        message: `Connecting to ${device_name}...`,
        type: 'info'
      });
      console.log('attempting to connect');
      window.bluetoothSerial.connect(
        device_id,
        success => {
          this.$q.notify({
            icon: 'bluetooth_connected',
            message: `Connected to ${device_name}`,
            type: 'positive'
          });
        },
        error => {
          this.$q.notify({
            icon: 'bluetooth_disabled',
            message: `Unable to connect to ${device_name}`,
            type: 'negative'
          });
        }
      );
    }
  }
};
</script>