<template>
  <div
    name="col-wrapper-2"
    :style="getSecondColumnStyle"
    class="d-flex align-items-start"
  >
    <div>
      <div class="d-flex label-wrapper" :style="getLabelWrapperStyle">
        <span @click="emitChangeLayoutRequest('square')" :style="getSquareLabelStyle"
          >Square</span
        >
        <span @click="emitChangeLayoutRequest('banner')" :style="getBannerLabelStyle"
          >Banner</span
        >
      </div>
      <div
        :style="getContainerStyle"
        @click="removeAllSelectionOnOuterClick"
        class="containerBox d-flex"
        id="editorDiv"
      >
        <div class="editor" @click="stopEventBubbling" :style="getEditorStyle">
          <div v-for="(image, index) in imageList" :key="index">
            <image-component :propsData="image" :index="+index" />
          </div>
        </div>
      </div>
      <br />
      <div class="w-100 d-flex justify-content-center">
        <button
          type="button "
          :style="getDownloadButtonStyle"
          @click="downloadArt"
          class="btn btn-sm btn-dark"
        >
          Download your Art
        </button>
      </div>
    </div>
    <div
      class="editor-button-container align-items-center d-flex flex-column"
      :style="getEditorButtonContainerStyle"
      @click="stopEventBubbling"
    >
      <button class="editor-btn" @click="setEditorSize(5)">
        <img src="../assets/plus.jpg" style="height: 100%; width: 100%" alt="" />
      </button>
      <button class="editor-btn" @click="setEditorSize(-5)">
        <img src="../assets/minus.jpg" style="height: 100%; width: 100%" alt="" />
      </button>
      <button class="editor-btn" v-if="targets.length > 0" @click="onFlipObject('flipX')">
        <img src="../assets/flipX.jpg" style="height: 100%; width: 100%" alt="" />
      </button>
      <button class="editor-btn" v-if="targets.length > 0" @click="onFlipObject('flipY')">
        <img src="../assets/flipY.jpg" style="height: 100%; width: 100%" alt="" />
      </button>
      <button class="editor-btn" v-if="targets.length > 0" @click="deleteObject">
        <img src="../assets/deleteIcone.jpg" style="height: 100%; width: 100%" alt="" />
      </button>
    </div>

    <div id="h1"></div>
    <div id="h2"></div>
    <div id="h3"></div>
    <div id="h4"></div>
  </div>
</template>

<script>
/* eslint-disable no-useless-escape */
import imageComponent from "./ImageComponent.vue";
import Selecto from "selecto";
import Moveable from "moveable";
import PubSub from "pubsub-js";
import gifFrames from "gif-frames";
import postscribe from "postscribe";
export default {
  name: "RightSection",
  components: {
    "image-component": imageComponent,
  },
  props: {
    propsData: Object,
  },
  data() {
    return {
      container: document.querySelector(".editor"),
      targets: [],
      moveableObj: null,
      frameMap: new Map(),
      selectoObj: null,
      imageList: [],
      frameDelayTime: 0,
      totalFrames: 1,
    };
  },
  computed: {
    getEditorStyle() {
      return {
        width: this.propsData.editorWidth - 120 + "px",
        height: this.propsData.editorHeight - 120 + "px",
        backgroundColor: "white",
        transform: "translate(50px, 50px)",
        position: "absolute",
        overflow: "hidden",
      };
    },
    getDownloadButtonStyle() {
      return {
        width: this.propsData.editorWidth - 100 + "px",
        borderRadius: "20px",
      };
    },
    getEditorButtonContainerStyle() {
      return {
        transform: "translate(10px, 24px)",
        backgroundColor: "#e8e7e3",
        width: "35px",
        minWidth: "35px",
        paddingTop: "10px",
        paddingBottom: "5px",
        borderRadius: "5px",
      };
    },
    getContainerStyle() {
      return {
        width: this.propsData.editorWidth + "px",
        height: this.propsData.editorHeight + "px",
      };
    },
    getSquareLabelStyle() {
      if (this.propsData.layout === "square") {
        return {
          color: "black",
          fontWeight: "bold",
          marginRight: "40px",
          cursor: "pointer",
        };
      } else {
        return {
          color: "gray",
          marginRight: "40px",
          cursor: "pointer",
        };
      }
    },
    getBannerLabelStyle() {
      if (this.propsData.layout === "banner") {
        return {
          color: "black",
          fontWeight: "bold",
          cursor: "pointer",
        };
      } else {
        return {
          color: "gray",
          cursor: "pointer",
        };
      }
    },
    getLabelWrapperStyle() {
      return {
        width: this.propsData.editorWidth + "px",
        marginBottom: "5px",
      };
    },
    getSecondColumnStyle() {
      return {
        width: this.propsData.secondColumnWidth + "%",
        minWidth: this.propsData.editorWidth + "px",
      };
    },
  },
  mounted() {
    this.container = document.querySelector(".editor");
    PubSub.subscribe("addImage", this.onAddImage);
    PubSub.subscribe("changeTranslate", this.onChangeTranslate);
    PubSub.subscribe("updateLayout", this.onUpdateLayout);
    this.initMoveableManager();
    this.initSelectManager();
    this.loadJsFiles();
  },
  destroyed() {
    PubSub.unsubscribe("addImage", this.onAddImage);
    PubSub.unsubscribe("changeTranslate", this.onChangeTranslate);
    PubSub.unsubscribe("updateLayout", this.onUpdateLayout);
  },
  methods: {
    loadJsFiles() {
      postscribe(
        "#h1",
        `<script src="https://rawgit.com/antimatter15/jsgif/master/LZWEncoder.js"><\/script>`
      );
      postscribe(
        "#h2",
        `<script src="https://rawgit.com/antimatter15/jsgif/master/NeuQuant.js"><\/script>`
      );
      postscribe(
        "#h3",
        `<script src="https://rawgit.com/antimatter15/jsgif/master/GIFEncoder.js"><\/script>`
      );
      postscribe(
        "#h4",
        `<script src="https://cdn.jsdelivr.net/npm/gifler@0.1.0/gifler.min.js"><\/script>`
      );
    },
    downloadArt() {
      let encoder = new GIFEncoder();
      let canvas = document.createElement("canvas");
      canvas.height = this.propsData.editorHeight - 120;
      canvas.width = this.propsData.editorWidth - 120;
      encoder.setRepeat(0);
      encoder.setDelay(+this.frameDelayTime);
      encoder.setSize(canvas.width, canvas.height);
      encoder.start();
      let context = canvas.getContext("2d");
      let x = null;
      let y = null;
      let w = null;
      let h = null;
      let fx = null;
      let fy = null;
      let angle = null;
      for (let i = 0; i < this.totalFrames; i++) {
        context.clearRect(0, 0, canvas.width, canvas.height);
        for (let j = 0; j < this.imageList.length; j++) {
          if (this.imageList[j].fileType === "image/gif") {
            x = null;
            y = null;
            w = null;
            h = null;
            fx = null;
            fy = null;
            angle = null;

            x = parseInt(this.imageList[j].translateX);
            y = parseInt(this.imageList[j].translateY);
            w = parseInt(this.imageList[j].width);
            h = parseInt(this.imageList[j].height);
            fx = parseInt(this.imageList[j].flipX);
            fy = parseInt(this.imageList[j].flipY);
            angle = parseInt(this.imageList[j].angle);

            context.save();
            context.translate(x + w / 2, y + h / 2);
            context.rotate((angle * Math.PI) / 180);
            context.scale(fx, fy);
            context.translate(-(x + w / 2), -(y + h / 2));

            context.drawImage(this.imageList[j].frameList[i], x, y, w, h);
            context.restore();
          } else {
            let imgTag = document.getElementById(this.imageList[0].id).firstElementChild;
            context.drawImage(
              imgTag,
              this.imageList[j].translateX,
              this.imageList[j].translateY,
              this.imageList[j].width,
              this.imageList[j].height
            );
          }
        }

        encoder.addFrame(context);
      }
      encoder.finish();
      encoder.download("download.gif");
    },

    stopEventBubbling(e) {
      e.stopPropagation();
    },
    onFlipObject(flipType) {
      let obj = this.imageList[this.targets[0].dataset.index];
      obj[flipType] = -obj[flipType];
    },
    deleteObject() {
      this.imageList.splice(this.targets[0].dataset.index, 1);
      this.removeAllSelectionOnOuterClick();
    },
    setEditorSize(val) {
      this.$emit("changeEditorSize", val);
    },
    removeAllSelectionOnOuterClick() {
      this.targets = [];
      this.moveableObj.target = this.targets;
    },

    onUpdateLayout() {
      if (this.imageList.length && this.imageList[0].type === "background") {
        this.imageList[0].height = this.propsData.editorHeight - 120;
        this.imageList[0].width = this.propsData.editorWidth - 120;
      }
      this.moveableObj.updateRect();
    },

    onAddImage(event, data) {
      if (data.type === "background") {
        if (this.imageList[0]?.type === "background") {
          this.imageList[0].src = data.src;
        } else {
          this.imageList.unshift(data);
        }
      } else {
        this.imageList.push(data);
      }
      setTimeout(() => {
        this.extractAllFramesFromGif();
      }, 10);
    },

    extractAllFramesFromGif() {
      for (let i = this.imageList.length - 1; i >= 0; i--) {
        if (
          this.imageList[i].fileType === "image/gif" &&
          this.imageList[i].frameList.length === 0
        ) {
          gifFrames({ url: this.imageList[i].src, frames: "all", outputType: "canvas" })
            .then((frameData) => {
              frameData.splice(0, 1);
              this.frameDelayTime = frameData[0].frameInfo.delay;
              this.totalFrames = frameData.length;

              let tempCanvas = null;
              for (let t = 0; t < frameData.length; t++) {
                tempCanvas = frameData[t].getImage();
                this.imageList[i].frameList.push(tempCanvas);
                tempCanvas = null;
              }
            })
            .catch(console.error.bind(console));
        }
      }
    },
    initMoveableManager() {
      this.moveableObj = new Moveable(this.container, {
        draggable: true,
        container: this.container,
        resizable: true,
        rotatable: true,
        origin: false,
        bounds: [0, 0, 200, 200],
        rotationPosition: "top",
      })
        .on("dragStart", (e) => {
          const target = e.target;
          this.setFrameFromTarget(target);
          const frame = this.frameMap.get(target);

          e.set(frame.translate);
        })
        .on("drag", (e) => {
          const target = e.target;

          const frame = this.frameMap.get(target);

          frame.translate = e.beforeTranslate;
          target.style.transform = `translate(${frame.translate[0]}px, ${frame.translate[1]}px) rotate(${frame.rotate}deg)`;
          let index = e.target.dataset.index;
          this.imageList[index].translateX = frame.translate[0];
          this.imageList[index].translateY = frame.translate[1];
          this.imageList[index].angle = frame.rotate;
        })
        .on("resizeStart", (e) => {
          this.moveableObj.keepRatio = true;
          const target = e.target;
          this.setFrameFromTarget(target);
          const frame = this.frameMap.get(target);
          // set last translate
          e.setOrigin(["%", "%"]);
          e.dragStart && e.dragStart.set(frame.translate);
        })
        .on("resize", (e) => {
          const frame = this.frameMap.get(e.target);
          frame.translate = e.drag.beforeTranslate;
          e.target.style.width = `${e.width}px`;
          e.target.style.height = `${e.height}px`;
          e.target.style.transform = `translate(${frame.translate[0]}px, ${frame.translate[1]}px) rotate(${frame.rotate}deg)`;
          let index = e.target.dataset.index;
          this.imageList[index].translateX = frame.translate[0];
          this.imageList[index].translateY = frame.translate[1];
          this.imageList[index].angle = frame.rotate;
          this.imageList[index].width = e.width;
          this.imageList[index].height = e.height;
        })
        .on("rotateStart", (e) => {
          const target = e.target;
          this.setFrameFromTarget(target);
          const frame = this.frameMap.get(target);

          e.set(frame.rotate);
        })
        .on("rotate", (e) => {
          const target = e.target;
          const frame = this.frameMap.get(target);
          frame.rotate = e.beforeRotate;
          target.style.transform = `translate(${frame.translate[0]}px, ${frame.translate[1]}px) rotate(${frame.rotate}deg)`;
          let index = e.target.dataset.index;
          this.imageList[index].translateX = frame.translate[0];
          this.imageList[index].translateY = frame.translate[1];
          this.imageList[index].angle = frame.rotate;
        });
    },
    setFrameFromTarget(target) {
      let { frameMap } = this;
      let object = this.imageList[target.dataset.index];
      frameMap.set(target, {
        translate: [object.translateX, object.translateY, 0],
        rotate: object.angle,
        transformOrigin: "50% 50%",
        clipStyle: "inset",
      });
    },
    initSelectManager() {
      this.selectoObj = new Selecto({
        container: this.container,
        dragContainer: ".editor",
        selectableTargets: [".target"],
        hitRate: 0,
        selectByClick: true,
        selectFromInside: false,
        toggleContinueSelect: ["shift"],
        ratio: 0,
      }).on("selectEnd", (e) => {
        this.targets = [];
        if (e.selected.length > 0) {
          this.targets = [e.selected[0]];
        }
        if (
          this.targets &&
          this.targets.length &&
          this.imageList[this.targets[0].dataset.index].type === "background"
        ) {
          this.disableMoveableControls();
        } else {
          this.enableMoveableControls();
        }
        this.moveableObj.target = this.targets;
        if (e.isDragStart) {
          e.inputEvent.preventDefault();
          setTimeout(() => {
            this.moveableObj.dragStart(e.inputEvent);
          });
        }
      });
    },
    enableMoveableControls() {
      this.moveableObj.draggable = true;
      this.moveableObj.resizable = true;
      this.moveableObj.rotatable = true;
    },
    disableMoveableControls() {
      this.moveableObj.draggable = false;
      this.moveableObj.resizable = false;
      this.moveableObj.rotatable = false;
    },

    emitChangeLayoutRequest(layout) {
      this.$emit("changeLayout", layout);
    },
    onChangeTranslate() {
      let limit = this.propsData.editorWidth - 120;
      this.imageList.forEach((image) => {
        if (image.translateX + image.width > limit && image.type === "image") {
          image.translateX = limit - image.width;
        }
      });
      this.removeAllSelectionOnOuterClick();
    },
    applyStyleToCanvas() {
      let tempCtx = null;
      this.imageList.forEach((image) => {
        if (image.frameList.length > 0) {
          image.frameList.forEach((frame) => {
            tempCtx = null;
            tempCtx = frame.getContext("2d");
            tempCtx.rotate(20);
            console.log(frame.toDataURL());
          });
        }
      });
    },
  },
  // watch: {
  //   imageList: {
  //     deep: true,
  //     handler() {
  //       // this.applyStyleToCanvas();
  //     },
  //   },
  // },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.btn-style {
  width: 80%;
  border-radius: 20px;
}
.editor-btn {
  margin-bottom: 5px;
  width: 25px;
  height: 25px;
  padding: 0px;
  border-radius: 3px;
}
.editor-btn:hover {
  width: 27px;
  height: 27px;
}
.containerBox {
  height: 300px;
  width: 300px;
  border-radius: 5px;
  background-color: #e8e7e3;
  border: 10px solid #242322;
}
</style>
