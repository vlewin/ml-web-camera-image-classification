<template>
  <div class="container">
    <div class="row">
      <div class="col-md-6">
        <h2>{{ text }}</h2>
        <button type="button" class="btn btn-success" @click="toggle">{{ interval ? 'Stop' : 'Start' }}</button>

        <code v-if="device">{{ device.label }}</code>
        <div class="border" :class="{ active: interval }">
          <web-cam ref="webcam"
                width="255px"
                height="255px"
                :deviceId="deviceId"
                @started="onStarted"
                @stopped="onStopped"
                @error="onError"
                @cameras="onCameras"
                @camera-change="onCameraChange" />
        </div>

        <div class="row">
          <div class="col-md-12">
            <select v-model="camera">
              <option>-- Select Device --</option>
              <option v-for="device in devices" :key="device.deviceId" :value="device.deviceId">{{ device.label }}</option>
            </select>
          </div>
          <div class="col-md-12">
            <button type="button" class="btn btn-primary" @click="onCapture">Capture Photo</button>
            <button type="button" class="btn btn-danger" @click="onStop">Stop Camera</button>
            <button type="button" class="btn btn-success" @click="onStart">Start Camera</button>
          </div>
        </div>
      </div>
      <div class="col-md-6">
        <p v-for="result in results">
          {{ result.className }}
          {{ result.probability }}
        </p>

        <h2>Captured Image</h2>

        <a v-if="img" class="btn btn-primary" v-bind:href="img" download="filename.jpg">Download image</a>

        <figure class="figure">
          <a v-bind:href="img" download="filename.jpg">
            <img :src="img" class="img-responsive" />
          </a>
        </figure>

      </div>
    </div>
  </div>
</template>

<script>
import { WebCam } from 'vue-web-cam'

export default {
  name: 'Camera',
  components: {
    WebCam
  },
  data () {
    return {
      img: null,
      camera: null,
      deviceId: null,
      devices: [],
      results: [],
      message: new SpeechSynthesisUtterance(),
      lang: 'en-US',
      text: null,
      interval: null
    }
  },

  async mounted () {
    this.message.lang = this.lang
    this.message.voice = window.speechSynthesis.getVoices().find((v) => { return v.lang === this.lang })
    this.text = 'Unknown'

    this.mobilenet = await window.mobilenet.load()

    // setInterval(() => {
    //   this.mobilenet.classify(document.getElementsByTagName('video')[0]).then((r) => {
    //     this.results = r
    //     this.text = this.results[0].className
    //     // this.text = this.results[0].className.split(',')[0]
    //   })
    // }, 5000)
  },

  computed: {
    device: function () {
      return this.devices.find((d) => { return d.id === this.deviceId })
    }
  },

  watch: {
    text: function (text) {
      this.message.text = text
      window.speechSynthesis.speak(this.message)
    },

    camera: function (id) {
      this.deviceId = id
    },

    devices: function () {
      // Once we have a list select the first one
      let first = this.devices[0]
      if (first) {
        this.camera = first.deviceId
        this.deviceId = first.deviceId
      }
    }
  },

  methods: {
    toggle () {
      if (this.interval) {
        this.interval = clearInterval(this.interval)
        this.results = []
      } else {
        clearInterval(this.interval)

        this.interval = setInterval(() => {
          this.mobilenet.classify(document.getElementsByTagName('video')[0]).then((r) => {
            this.results = r

            if (this.results[0].probability > 0.5) {
              this.text = this.results[0].className
            } else {
              this.text = 'Unknown'
            }
            // this.text = this.results[0].className
            // this.text = this.results[0].className.split(',')[0]
          })
        }, 3000)
      }
    },

    onCapture () {
      this.img = this.$refs.webcam.capture()
      console.log('onCapture')
      console.log(this.img)
    },

    onStarted (stream) {
      console.log('On Started Event', stream)
    },

    onStopped (stream) {
      console.log('On Stopped Event', stream)
    },

    onStop () {
      this.$refs.webcam.stop()
    },

    onStart () {
      this.$refs.webcam.start()
    },

    onError (error, stream) {
      console.log('On Error Event', error, stream)
    },

    onCameras (cameras) {
      this.devices = cameras
      console.log('On Cameras Event', cameras)
    },

    onCameraChange (deviceId) {
      this.deviceId = deviceId
      this.camera = deviceId
      console.log('On Camera Change Event', deviceId)
    }
  }
}
</script>

<style>
.active video {
  border: 5px solid green;
}
</style>
