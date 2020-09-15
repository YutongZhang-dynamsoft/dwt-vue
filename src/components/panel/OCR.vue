<template>
  <div id="ocr-panel">
    <v-btn depressed color="primary" @click="doOCR">Recognize</v-btn>
    <div class="outlined-box">
      <p v-for="(text, idx) in result" :key="idx">{{ text }}</p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ocr-panel',
  data () {
    return {
      ocrObj: null,
      dwtLib: null,
      result: ''
    }
  },
  methods: {
    setupOcr (resPath, downloadDLL, dwtObj, dwtLib) {
      new Promise((res, rej) => {
        const ocrObj = dwtObj.Addon.OCR
        let corePath = resPath + '/addon/OCR.zip'
        let langPath = resPath + '/addon/OCRBasicLanguages/English.zip'

        if (downloadDLL) {
          ocrObj.Download(
            corePath, 
            this.setupOcr(resPath, false, dwtObj, dwtLib), 
            (errCode, errStr) => {
              rej({errCode: errCode, errStr: errStr})
            }
          )
        } else {
          ocrObj.DownloadLangData(
            langPath,
            () => { res([ocrObj, dwtLib]); },
            (errCode, errStr) => { 
              rej({errCode: errCode, errStr: errStr})
            }
          )
        }
      })
      .then((ocr) => {
        this.ocrObj = ocr[0]
        this.dwtLib = ocr[1]
      })
    },
    doOCR () {
      this.ocrObj.SetLanguage('eng')
      this.ocrObj.SetOutputFormat(0)
      this.ocrObj.RecognizeSelectedImages(
        (result) => {
          let _textResult = this.dwtLib.base64.decode(result.Get()).split(/\r?\n/g)
          let res = []
           for (let i = 0; i < _textResult.length; i++) {
            if (!_textResult[i].trim()) { continue }
            res.push(_textResult[i].trim() + '\n')
          }
          this.result = res
        },
        (errCode, errStr) => {
          console.error(`${errCode} - ${errStr}`)
        }
      )
    }
  }
}
</script>

<style scoped>
.outlined-box {
  border: 1px solid black;
}
</style>