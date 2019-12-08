<template>
  <v-layout column justify-center align-center>
    <h1 class="mb1">QR wi-fi</h1>
    <v-card width="400">
      <v-card-title>
        wi-fi情報
      </v-card-title>
      <v-form>
        <v-container>
          <v-row>
            <v-col cols="12" md="4">
              <v-text-field
                v-model="formData.ssid"
                :rules="ssidRules"
                label="SSID"
                required
              />
              <v-select
                v-model="formData.encryption"
                :items="encryptionItems"
                :rules="encryptionRules"
                label="Encryption"
              />
              <v-text-field
                v-model="formData.key"
                :rules="keyRules"
                label="KEY"
                required
              />
            </v-col>
          </v-row>
        </v-container>
      </v-form>
      <p v-if="showErrorMessage" class="text-center">
        Woops, all forms must be filled.
      </p>
      <v-card-actions class="d-flex align-end flex-column">
        <v-btn
          @click="createImage()"
          class="text-right"
          outlined
          color="primary"
        >
          Get QRCode
        </v-btn>
      </v-card-actions>
      {{ originalQRCode }}
      <div class="text-center">
        <img :src="originalQRCode" alt="" />
        <div id="canvas"></div>
      </div>
    </v-card>
  </v-layout>
</template>

<script>
import QRCode from 'qrcode'

export default {
  data: () => ({
    formData: {
      ssid: '',
      encryption: '',
      key: ''
    },
    ssidRules: [(v) => !!v || 'SSID is required'],
    keyRules: [(v) => !!v || 'KEY is required'],
    encryptionRules: [(v) => !!v || 'Encryption is required'],
    showErrorMessage: false,
    version: 5,
    errorCorrectionLevel: 'H',
    originalQRCode: ''
  }),
  computed: {
    encryptionItems() {
      return ['WPA/WPA2', 'WEP', 'None']
    },
    existEmptyForm() {
      return Object.values(this.formData).every((val) => val.length > 0)
    },
    wifiString() {
      return `WIFI:S:${this.formData.ssid};T:${this.formData.encryption};P:${this.formData.key};;`
    }
  },
  methods: {
    createImage() {
      this.showErrorMessage = false
      if (!this.existEmptyForm) {
        this.showErrorMessage = true
      }
      this.generateQR(this.wifiString)
    },
    async generateQR(string) {
      await QRCode.toCanvas(
        string,
        {
          errorCorrectionLevel: this.errorCorrectionLevel,
          version: this.version
        },
        (err, canvas) => {
          if (err) {
            console.error(err)
          }
          const container = document.getElementById('canvas')
          container.appendChild(canvas)
        }
      )
    }
  }
}
</script>
