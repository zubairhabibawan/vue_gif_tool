<template>
  <div :id="propsData.id" class="target" :style="getImageParentStyle" :data-index="index">
    <img :src="propsData.src" alt="" :style="getImageStyle" />
  </div>
</template>
<script>
export default {
  name: "ImageComponent",
  props: {
    propsData: Object,
    index: Number,
  },

  computed: {
    getImageStyle() {
      let data = this.propsData;
      let styles = {
        width: "100%",
        height: "100%",
        transform: "scale(" + data.flipX + "," + data.flipY + ")",
      };
      return styles;
    },
    getImageParentStyle() {
      let data = this.propsData;
      let styles = {
        width: data.width + "px",
        position: "absolute",
        overflow: "hidden",
        height: data.height + "px",
        transform:
          "translate(" +
          data.translateX +
          "px , " +
          data.translateY +
          "px) rotate(" +
          data.angle +
          "deg) ",
        zIndex: data.zIndex,
      };
      return styles;
    },
  },
};
</script>
<style scoped></style>
