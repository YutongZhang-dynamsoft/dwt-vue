<template>
  <div id="scan-panel">
    <!-- Source selection -->
    <v-select
      v-model="selectScanner"
      :items="scanners"
      item-value="id"
      item-text="text"
      label="Scanner Sources"
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
      return-object
    ></v-select>
    <!-- Color Mode -->
    <v-select
      v-model="colorMode"
      :items="colorModeOptions"
      item-text="text"
      item-value="val"
      label="Color Mode"
      outlined
      return-object
    ></v-select>
    <div class="flex">
      <!-- Show UI Control -->
      <v-switch
        v-model="showUI"
        label="Show UI"
      ></v-switch>
      <!-- Auto Feeder Control -->
      <v-switch
        v-model="autoFeeder"
        label="Auto Feeder"
      ></v-switch>
      <!-- Duplex Control -->
      <v-switch
        v-model="duplex"
        label="Duplex"
      ></v-switch>
    </div>
    <!-- Scan button -->
    <v-btn color="primary" depressed large @click="acquireImage">Scan</v-btn>
  </div>
</template>

<script>
export default {
  name: 'scan-panel',
  props: [''],
  data () {
    return {
      dwtObj: null,
      selectScanner: null,
      scanners: [],
      colorMode: { val: 2, text: 'Color' },
      colorModeOptions: [
        { val: 0, text: 'Black & White' }, 
        { val: 1, text: 'Grayscale' },
        { val: 2, text: 'Color' }
      ],
      resolution: { val: 300, text: '300' },
      resolutionOptions: [
        { val: 100, text: '100' }, 
        { val: 150, text:'150' }, 
        { val: 300, text: '300' }
      ],
      showUI: true,
      autoFeeder: false,
      duplex: false
    }
  },
  methods: {
    acquireImage () {
      const DWObject = this.dwtObj;
      if (DWObject) {
        if (DWObject.UseLocalService) {
          let configure = {
            IfShowUI: this.showUI,
            PixelType: this.colorMode,
            Resolution: this.resolution,
            IfFeederEnabled: this.autoFeeder,
            IfDuplexEnabled: this.duplex,
            IfDisableSourceAfterAcquire: true,
            // Advance settings
            IfGetImageInfo: true,
            IfGetExtImageInfo: true,
            extendedImageInfoQueryLevel: 0
          }
          DWObject.SelectSourceByIndex(this.selectScanner)
          DWObject.AcquireImage(
            configure,
            () => { DWObject.CloseSource() },
            () => { DWObject.CloseSource() }
          )
        }
      }
    },
    setupScan (dwtObj) {
      this.dwtObj = dwtObj
      this.scanners = dwtObj.GetSourceNames().map((scanner, idx) => { return { id: idx, text: scanner } })
    }
  }
}
</script>

<style scoped>
.flex {
  display: flex;
  justify-content: left;
}
.flex > * {
  margin-right: 12px;;
}
</style>