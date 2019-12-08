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
                :rules="rules.ssidRules"
                label="SSID"
                required
              />
              <v-select
                v-model="formData.encryption"
                :items="encryptionItems"
                :rules="rules.encryptionRules"
                label="Encryption"
              />
              <v-text-field
                v-model="formData.key"
                :rules="rules.keyRules"
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
      <input
        id="ufile"
        @change="uploadImage"
        name="ufile"
        type="file"
        accept="image/jpeg,image/png"
      /><br />
      <div class="text-center">
        <canvas
          id="qrcode__canvas"
          :width="canvasSize.width"
          :height="canvasSize.height"
        />
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
    rules: {
      ssidRules: [(v) => !!v || 'SSID is required'],
      keyRules: [(v) => !!v || 'KEY is required'],
      encryptionRules: [(v) => !!v || 'Encryption is required']
    },
    canvasSize: {
      height: 200,
      width: 200
    },
    logoSize: {
      height: 40,
      width: 40
    },
    showErrorMessage: false,
    version: 5,
    errorCorrectionLevel: 'H',
    originalQRCode: '',
    uploadImageBlobUrl: ''
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
    /**
     * アップロードした画像ファイルの blobUrl を取得する
     */
    uploadImage(value) {
      this.image = value
      const file = this.image.target.files[0]
      this.uploadImageBlobUrl = window.URL.createObjectURL(file)
    },
    /**
     * フォームがすべて入力されているとき、 QR コードを生成できる
     */
    createImage() {
      this.showErrorMessage = false
      if (!this.existEmptyForm) {
        this.showErrorMessage = true
      }
      this.generateQR()
    },
    /**
     * QR コードを生成する
     */
    async generateQR() {
      await QRCode.toDataURL(this.wifiString)
        .then((url) => {
          this.originalQRCode = url
        })
        .catch((err) => {
          console.error(err)
        })
      // await QRCode.toCanvas(
      //   string,
      //   {
      //     errorCorrectionLevel: this.errorCorrectionLevel,
      //     version: this.version
      //   },
      //   (err, canvas) => {
      //     if (err) {
      //       console.error(err)
      //     }
      //     canvas.className = 'qrcode__canvas'
      //     this.qrCodeCanvas = canvas
      //     const container = document.querySelector('.qrcode__container')
      //     container.appendChild(canvas)
      //   }
      // )
      this.draw()
    },
    draw() {
      const canvas = document.getElementById('qrcode__canvas')
      if (!canvas || !canvas.getContext) {
        return
      }
      const ctx = canvas.getContext('2d')
      const qrCodeImage = new Image()
      qrCodeImage.src = this.originalQRCode
      qrCodeImage.onload = () => {
        ctx.drawImage(
          qrCodeImage,
          0,
          0,
          this.canvasSize.width,
          this.canvasSize.height
        )
      }
      const logoImage = new Image()
      logoImage.src = this.uploadImageBlobUrl
      logoImage.onload = () => {
        ctx.drawImage(
          logoImage,
          80,
          80,
          this.logoSize.width,
          this.logoSize.height
        )
      }
    }
  }
}
</script>

<style lang="scss">
.qrcode__container {
  position: relative;
  width: 180px;
  height: 180px;
}

.roxx {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  -webkit-transform: translate(-50%, -50%);
  -ms-transform: translate(-50%, -50%);
  z-index: 1;
}
</style>
