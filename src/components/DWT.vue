<template>
  <div id="dwt-container">
    <Viewer 
      :id="this.viewer.id" 
      :ref="this.viewer.id"
      :dwtRef="this.dwt.obj"
      :width="this.viewer.width"
      :height="this.viewer.height"
    >
      <!-- DWT viewer -->
    </Viewer>
    <v-tabs id="dwt-control">
      <v-tab :key="0" @click="handleTabChange(0)">Scan</v-tab>
      <v-tab :key="1" @click="handleTabChange(1)">Webcam Capture</v-tab>
      <v-tab :key="2" @click="handleTabChange(2)">OCR</v-tab>
      <v-tab-item :key="0">
        <Scan :ref="'scan-panel'"></Scan>
      </v-tab-item>
      <v-tab-item :key="1">
        <Webcam :ref="'webcam-panel'"></Webcam>
      </v-tab-item>
      <v-tab-item>
        <OCR :ref="'ocr-panel'"/>
      </v-tab-item>
    </v-tabs>
  </div>
</template>

<script>
import dwt from 'dwt'
import Viewer from '@/components/panel/Viewer'
import Scan from '@/components/panel/Scan'
import Webcam from '@/components/panel/Webcam.vue'
import OCR from '@/components/panel/OCR.vue'

export default {
  name: 'dwt',
  components: {
    Viewer,
    Scan,
    Webcam,
    OCR
  },
  data () {
    return {
      dwt: {
        obj: null,
        id: 'dwtObject',
        licenseKey: 't01016QAAADyBe7yfb9oPaRKoDodUi2D6w3Dj/XeSforvLiBX6PXItwyqx3NL/4Uso1U/t4Gol58RCjB9B1q+RjxJ2qOVHa1eGzRmGbzga3PGGn1/tDAWpk/DKsyhQmO9F1PDDdxIL+c='
      },
      viewer: {
        id: 'dwtViewer',
        width: '100%',
        height: '600px'
      }
    }
  },
  mounted () {
    this.mountDWT()
      .then(() => {
        // this.handleTabChange(0)
        this.initPanels()
        this.bindViewer()
      })
  },
  methods: {
    bindViewer () {
      this.$refs[this.viewer.id].mountViewer(this.dwt.obj)
    },
    mountDWT () {
      return new Promise((res, rej) => {
        this.unmountDWT()
          .then(() => {
            dwt.WebTwainEnv.UseLocalService = true;
            dwt.WebTwainEnv.ResourcesPath = "resources/dwt";
            dwt.WebTwainEnv.ProductKey = this.dwt.licenseKey
            dwt.WebTwainEnv.AutoLoad = false;
            dwt.WebTwainEnv.Containers = [];
            dwt.WebTwainEnv.IfAddMD5InUploadHeader = false;
            dwt.WebTwainEnv.IfConfineMaskWithinTheViewer = false;
            let dwtConfig = { WebTwainId: this.dwt.id }
            // By default, use local service is true
            dwt.WebTwainEnv.CreateDWTObjectEx(
                  dwtConfig, 
                  (dwtObject) => { this.dwt.obj = dwtObject; res(true);},
                  (errStr) => { console.log(`failed to initialize dwt, message: ${errStr}`); rej(false);}
            )
          })
      })
    },
    /**
     * Delete dwt instance
     */
    unmountDWT () {
      return new Promise((res, rej) => {
        if (dwt.WebTwainEnv.DeleteDWTObject(this.dwt.id)) {
          res(true)
        } else {
          rej(false)
        }
      })
    },
    initPanels () {
      window.refs = this.$refs
      this.$refs['scan-panel'].setupScan(this.dwt.obj)
    },
    handleTabChange (key) {
      let loadOnSwitch = () => {
        switch(key) {
          case 0: { this.$refs['scan-panel'].setupScan(this.dwt.obj); break; }
          case 1: { this.$refs['webcam-panel'].setupWebcam(dwt.Lib.detect.ssl, dwt.EnumDWT_VideoRotateMode, this.dwt.obj); break; }
          case 2: { this.$refs['ocr-panel'].setupOcr(dwt.WebTwainEnv.ResourcesPath, true, this.dwt.obj, dwt.Lib); break; }
        }
      }
      setTimeout(loadOnSwitch, 100)
    }
  },
}
</script>

<style scoped>
#dwt-container {
  display: flex;
  height: inherit;
  width: inherit;
}
#dwt-control {
  max-width: 400px;
}
</style>