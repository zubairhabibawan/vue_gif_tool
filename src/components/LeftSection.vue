<template>
  <div name="col-wrapper-1" :style="getFirstColumnStyle" class="d-flex flex-column">
    <span id="toolTitle">Loafz Art Tool</span>
    <br />
    <div>
      <input
        id="fileUpload"
        @change="getSelectedForeGround"
        type="file"
        accept="image/png , image/gif , image/jpeg , image/jpg"
        hidden
      />
      <button type="button" @click="onUploadFile" class="btn btn-sm btn-dark btn-style">
        Upload your Apez
      </button>
    </div>
    <div>
      <span>PNG , GIF , JPEG files</span>
    </div>
    <br />
    <div>
      <input
        id="backgroundUpload"
        @change="getSelectedBackground"
        type="file"
        accept="image/png , image/gif , image/jpeg , image/jpg"
        hidden
      />
      <button
        type="button"
        @click="onUploadBackground"
        class="btn btn-sm btn-dark btn-style"
      >
        Upload Your Background
      </button>
    </div>
    <div>
      <span>PNG , GIF , JPEG files</span>
    </div>
    <br />
    <div>
      <button type="button" class="btn btn-sm btn-dark btn-style">
        Use our backgrounds
      </button>
    </div>
    <br />
    <div class="d-flex">
      <div
        @click="emitChangeLayoutRequest('square')"
        class="layout-div"
        :style="getSquareLayoutStyle"
      >
        <span>SQUARE</span>
      </div>
      <div
        @click="emitChangeLayoutRequest('banner')"
        class="layout-div"
        :style="getBannerLayoutStyle"
      >
        <span>BANNER</span>
      </div>
    </div>
    <div>
      <span>Choose your layout</span>
    </div>
  </div>
</template>

<script>
import PubSub from "pubsub-js";
import _ from "lodash";

export default {
  name: "LeftSection",
  props: {
    propsData: Object,
  },
  data() {
    return {
      fileSizeLimit: 0.5 * 1024 * 1024,
      defaultImageValues: {
        id: "3456345",
        src: "./assets/1.jpg",
        type: "image",
        fileType: "",
        width: 200,
        height: 200,
        keepRatio: true,
        translateX: 0,
        translateY: 0,
        flipX: 1,
        flipY: 1,
        zIndex: 0,
        draggable: true,
        locked: false,
        angle: 0,
        frameList: [],
      },
    };
  },
  computed: {
    getDownloadButtonStyle() {
      return {
        width: this.propsData.editorWidth - 100 + "px",
        borderRadius: "20px",
      };
    },

    getSquareLayoutStyle() {
      if (this.propsData.layout === "square") {
        return {
          width: "30%",
          border: "2px solid black",
          backgroundColor: "#e5eaeb",
          fontSize: "14px",
          fontWeight: "bold",
        };
      } else {
        return {
          width: "30%",
        };
      }
    },
    getBannerLayoutStyle() {
      if (this.propsData.layout === "banner") {
        return {
          width: "48%",
          border: "2px solid black",
          backgroundColor: "#e5eaeb",
          fontSize: "14px",
          fontWeight: "bold",
        };
      } else {
        return {
          width: "48%",
        };
      }
    },
    getFirstColumnStyle() {
      return {
        width: this.propsData.firstColumnWidth + "%",
      };
    },
  },
  mounted() {},

  methods: {
    addNewImage(data) {
      setTimeout(() => {
        let defaultDataClone = _.clone(this.defaultImageValues, true);
        let newData = Object.assign(defaultDataClone, data);
        PubSub.publish("addImage", newData);
      }, 0);
    },
    emitChangeLayoutRequest(layout) {
      this.$emit("changeLayout", layout);
    },
    getSelectedBackground(fileParam) {
      let file = null;
      let tempFileArray = [];
      if (fileParam.target.files.length === 1) {
        file = fileParam.target.files[0];
        console.log(fileParam.target.files[0]);
        if (file.type.startsWith("image") && +file.size <= this.fileSizeLimit) {
          tempFileArray.push(URL.createObjectURL(file));
          this.getImageFromBase64(tempFileArray[0]).then((img) => {
            let temData = {
              id: "image" + 0,
              type: "background",
              src: img.src,
              fileType: file.type,
              height: this.propsData.editorHeight - 120,
              width: this.propsData.editorWidth - 120,
              zIndex: 0,
            };
            this.addNewImage(temData);
          });
        } else {
          alert("File should be less than 2 MB");
          file = null;
          tempFileArray = [];
        }
      }
    },
    onUploadFile() {
      document.getElementById("fileUpload").click();
    },
    getSelectedForeGround(fileParam) {
      let file = null;
      let tempFileArray = [];
      for (let i = 0; i < fileParam.target.files.length; i++) {
        file = fileParam.target.files[i];
        if (file.type.startsWith("image") && +file.size <= this.fileSizeLimit) {
          tempFileArray.push({ type: file.type, data: URL.createObjectURL(file) });
        } else {
          alert("File should be less than 2 MB");
          file = null;
          tempFileArray = [];
        }
      }
      if (tempFileArray.length) {
        for (let m = 0; m < tempFileArray.length; m++) {
          this.getImageFromBase64(tempFileArray[m].data).then((img) => {
            let temData = {
              id: "image" + m,
              src: img.src,
              fileType: tempFileArray[m].type,
              height: this.propsData.editorHeight - 130,
              width: this.propsData.editorHeight - 130,
              zIndex: m,
              translateX: 5,
              translateY: 5,
              frameList: [],
            };
            this.addNewImage(temData);
          });
        }
      }
    },
    getImageFromBase64(src) {
      return new Promise((resolve, reject) => {
        let img = new Image();
        img.onload = () => {
          resolve(img);
        };
        img.onerror = reject;
        img.src = src;
      });
    },
    onUploadBackground() {
      document.getElementById("backgroundUpload").click();
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@font-face {
  font-family: myCustomHeaderFont;
  src: url("../assets/Regular.ttf");
}
#toolTitle {
  font-family: myCustomHeaderFont;
  font-size: 70px;
  line-height: 1;
}
.btn-style {
  width: 80%;
  border-radius: 20px;
}
.layout-div {
  height: 120px;
  line-height: 120px;
  text-align: center;
  margin-right: 2%;
  border-radius: 10px;
  cursor: pointer;
  border: 2px solid black;
  border-style: dashed;
}
</style>
