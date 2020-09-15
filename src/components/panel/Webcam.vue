<template>
  <div id="webcam-panel">
    <!-- Source selection -->
    <v-select
      v-model="selectedWebcam"
      :items="webcamSources"
      item-value="val"
      item-text="text"
      label="Webcam Sources"
      outlined
    ></v-select>
    <!-- Resolution setting -->
    <v-select
      v-model="resolution"
      :items="resolutionOptions"
      item-text="text"
      item-value="val"
      label="Resolution"
      outlined
    ></v-select>
    <!-- Framerates -->
    <v-select
      v-model="framerate"
      :items="framerateOptions"
      item-text="text"
      item-value="val"
      label="Frame Rate"
      outlined
    ></v-select>
    <!-- Color Mode -->
    <v-select
      v-model="mediaType"
      :items="mediaTypeOptions"
      item-text="text"
      item-value="val"
      label="Media Type"
      outlined
    ></v-select>
    <!-- Rotation -->
    <v-select
      v-model="rotate"
      :items="rotationOptions"
      item-text="text"
      item-value="val"
      label="Rotation"
      outlined
    ></v-select>
    <!-- Operation button group -->
    <v-btn color="primary" depressed large @click="capture">Capture</v-btn>
    <v-btn depressed large @click="toggleLiveView">Live Viewing {{ liveViewing ? 'OFF':'ON' }}</v-btn>
  </div>
</template>

<script>
export default {
  name: 'webcam-panel',
  props: ['ssl', 'rotateMode'],
  data () {
    return {
      dwtObj: null,
      webcamObj: null,
      selectedWebcam: null,
      webcamSources: [],
      resolution: null,
      resolutionOptions: [],
      mediaType: null,
      mediaTypeOptions: [],
      framerate: null,
      framerateOptions: [],
      rotate: 0,
      rotationOptions: [],
      liveViewing: false
    }
  },
  watch: {
    selectedWebcam () {
        this.updateWebcamOptions()
    }
  },
  methods: {
    setupWebcam (ssl, rotateMode, dwtObj) {
      this.dwtObj = dwtObj
      this.webcamObj = dwtObj.Addon.Webcam
      // Set rotate mode
      const rotation = [
        { text: 'OFF', val: rotateMode.VRM_NONE },
        { text: 'CLOCKWISE 90°', val: rotateMode.VRM_90_DEGREES_CLOCKWISE },
        { text: 'CLOCKWISE 180°', val: rotateMode.VRM_180_DEGREES_CLOCKWISE },
        { text: 'CLOCKWISE 270°', val: rotateMode.VRM_270_DEGREES_CLOCKWISE },
        { text: 'FLIP VERTICAL', val: rotateMode.VRM_FLIP_VERTICAL },
        { text: 'FLIP HORIZONTAL', val: rotateMode.VRM_FLIP_HORIZONTAL }
      ]
      this.rotationOptions = rotation
      // Set Default Webcam
      if (ssl) {
        dwtObj.IfSSL = true
        dwtObj.HTTPPort = 443
      }
      this.webcamSources = this.webcamObj.GetSourceList().map(e => { return { text: e, val: e } })
    },
    updateWebcamOptions () {
      this.webcamObj.StopVideo()
      this.webcamObj.SelectSource(this.selectedWebcam)
      
      function ObjToArr(obj) {
        let count = obj.GetCount()
        let arr = []
        for (let i = 0; i < count; i++) {
          arr.push({ text: obj.Get(i), val: obj.Get(i) })
        }
        return arr
      }

      new Promise((res) => {
        this.framerateOptions = ObjToArr(this.webcamObj.GetFrameRate())
        this.resolutionOptions = ObjToArr(this.webcamObj.GetResolution())
        this.mediaTypeOptions = ObjToArr(this.webcamObj.GetMediaType())
        res()
      })
      .then(() => {
        this.resolution = this.resolutionOptions[0]
        this.mediaType = this.mediaTypeOptions[0]
        this.framerate = this.framerateOptions[0]
      })
    },
    updateWebcamSetting () {
      if (this.liveViewing) { this.webcamObj.StopVideo() }
      this.webcamObj.SetFrameRate(this.framerate)
      this.webcamObj.SetMediaType(this.mediaType)
      this.webcamObj.SetResolution(this.resolution)
      this.webcamObj.SetVideoRotateMode(this.rotate)
      if (this.liveViewing) { this.webcamObj.PlayVideo() }
    },
    capture () {
      if (this.liveViewing) { this.toggleLiveView(false) }
      this.webcamObj.CaptureImage(()=>{}, (errCode, errStr) => { console.error(`${errCode} - ${errStr}`) })
    },
    toggleLiveView () {
      if (this.liveViewing) { this.setVideoPlayback(false) }
      else { this.setVideoPlayback(true) }
    },
    setVideoPlayback (show) {
      if (show) {
        this.webcamObj.StopVideo()
        setTimeout(
          () => {
            this.webcamObj.PlayVideo(this.dwtObj, 80, () => {})
            this.liveViewing = true
          },
          30
        )
      } else {
        this.webcamObj.StopVideo()
        this.liveViewing = false
      }
    }
  },
}
</script>