<template>
  <div class="d-flex w-100 justify-content-center">
    <div
      name="parent-container"
      :style="getParentContainerStyle"
      class="d-flex w-100 justify-content-center"
    >
      <left-section
        :propsData="getLeftSectionProps"
        @changeLayout="onChangeLayout"
      ></left-section>
      <right-section
        :propsData="getRightSectionProps"
        @changeLayout="onChangeLayout"
        @changeEditorSize="onChangeEditorSize"
      ></right-section>
    </div>
  </div>
</template>

<script>
import leftSection from "./LeftSection.vue";
import rightSection from "./RightSection.vue";
import PubSub from "pubsub-js";

export default {
  name: "loafz",
  components: {
    "left-section": leftSection,
    "right-section": rightSection,
  },
  data() {
    return {
      fileSizeLimit: 0.5 * 1024 * 1024,
      editorWidth: 400,
      editorHeight: 400,
      firstColumnWidth: 30,
      secondColumnWidth: 30,
      layout: "square",
    };
  },
  computed: {
    getParentContainerStyle() {
      return {
        marginTop: "150px",
        maxWidth: "1600px",
        minWidth: "900px",
      };
    },
    getLeftSectionProps() {
      return {
        editorWidth: this.editorWidth,
        editorHeight: this.editorHeight,

        layout: this.layout,
        firstColumnWidth: this.firstColumnWidth,
      };
    },
    getRightSectionProps() {
      return {
        editorWidth: this.editorWidth,
        editorHeight: this.editorHeight,
        layout: this.layout,
        firstColumnWidth: this.firstColumnWidth,
        secondColumnWidth: this.secondColumnWidth,
      };
    },
  },

  methods: {
    onChangeLayout(layout) {
      this.layout = layout;
    },
    onChangeEditorSize(value) {
      this.editorHeight += +value;
    },
    updateEditorSize() {
      if (this.layout === "square") {
        this.secondColumnWidth = this.firstColumnWidth;
        this.editorWidth = this.editorHeight;
      } else if (this.layout === "banner") {
        this.secondColumnWidth = this.firstColumnWidth * 3;
        this.editorWidth = this.editorHeight * 3;
      }

      setTimeout(() => {
        PubSub.publish("updateLayout");
        PubSub.publish("changeTranslate");
      }, 0);
    },
  },
  watch: {
    layout: {
      handler() {
        this.updateEditorSize();
      },
    },
    editorHeight: {
      handler() {
        this.updateEditorSize();
      },
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped></style>
