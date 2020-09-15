<template>
  <div id="viewer-container" :style="{ width:width, height: height }">
    <div :id="id" style="height: inherit;"></div>
    <div id="edit-button-group">
      <v-btn outlined tile color="grey darken-1" small @click="rotate(-1)">Rotate Left</v-btn>
      <v-btn outlined tile color="grey darken-1" small @click="rotate(1)">Rotate Right</v-btn>
      <v-btn outlined tile color="grey darken-1" small @click="flip()">Flip</v-btn>
      <v-btn outlined tile color="grey darken-1" small @click="mirror()">Mirror</v-btn>
      <v-btn tile dark depressed small @click="openEditor">Open Editor</v-btn>
    </div>
  </div>
</template>

<script>
export default {
  name:'dwt-viewer',
  props: ['id', 'width', 'height', 'dwtRef'],
  data () {
    return {
      dwtObj: null,
      viewerObj: null,  // reference of viewer instance
      currIdx: 0,  // indicator of selected image
    }
  },
  methods: {
    mountViewer (dwtObj) {
      if (!dwtObj) {
        alert('WebTwain Object has not been initialized yet.')
      } else {
        let viewOptions = {
          width: this.width,
          height: this.height,
          view: {
            bShow: true,
            Width: this.width,
            Height: this.height
          }
        }
        if (dwtObj.BindViewer(this.id, viewOptions)) {
          this.dwtObj = dwtObj
          this.viewerObj = this.dwtRef.Viewer
          dwtObj.RegisterEvent('OnMouseClick', (idx) => { this.currIdx = idx })
        }
      }
    },
    rotate (direction) {
      switch (direction) {
        case -1:
        case '-1': {
          this.dwtObj.Rotate(this.currIdx, -90, true);
          break;
        }
        case 1:
        case '1': {
          this.dwtObj.Rotate(this.currIdx, 90, true);
          break;
        }
      }
    },
    flip: function() {
      this.dwtObj.Flip(this.currIdx);
    },
    mirror: function() {
      this.dwtObj.Mirror(this.currIdx);
    },
    openEditor: function () {
      if (this.dwtObj) {
        this.dwtObj.ShowImageEditor()
      }
    }
  }
}
</script>

<style scoped>
#viewer-container {
  box-sizing: content-box;
  text-align: center;
  /* width: 100%;
  height: inherit; */
}
#edit-button-group {
  display: inline-block;
}
.inherit-width {
  width: inherit;
}
.inherit-height {
  height: inherit;
}
</style>
