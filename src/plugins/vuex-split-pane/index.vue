<template>
  <div
    ref="splitbox"
    :style="{ cursor, userSelect }"
    class="vue-splitter-container clearfix"
    @mouseup="onMouseUp"
    @mousemove="onMouseMove"
  >
    <pane
      class="splitter-pane splitter-paneL"
      :split="split"
      :style="{ [type]: size + 'px' }"
    >
      <slot name="paneL"></slot>
    </pane>

    <resizer
      :class-name="className"
      :style="{ [resizeType]: size + 'px' }"
      :split="split"
      @mousedown.native="onMouseDown"
      @click.native="onClick"
    ></resizer>

    <pane
      class="splitter-pane splitter-paneR"
      :split="split"
      :style="{ [type]: `calc(100% - ${size}px)` }"
    >
      <slot name="paneR"></slot>
    </pane>

    <div class="vue-splitter-container-mask" v-if="active"></div>
  </div>
</template>

<script>
import Resizer from "./resizer.vue";
import Pane from "./pane.vue";
export default {
  name: "SplitPane",
  components: { Resizer, Pane },
  props: {
    defaultSize: {
      type: Number,
      default: 250,
    },
    minSize: {
      type: Number,
      default: 100,
    },
    maxSize: {
      type: Number,
      default: 0,
    },
    split: {
      validator(value) {
        return ["vertical", "horizontal"].indexOf(value) >= 0;
      },
      required: true,
    },
    className: {
      type: String,
      default: "",
      required: false,
    },
  },
  computed: {
    userSelect() {
      return this.active ? "none" : "";
    },
    cursor() {
      return this.active
        ? this.split === "vertical"
          ? "col-resize"
          : "row-resize"
        : "";
    },
  },
  watch: {
    defaultSize(newValue) {
      this.size = newValue;
    },
  },
  data() {
    return {
      active: false,
      hasMoved: false,
      height: null,
      size: this.defaultSize,
      type: this.split === "vertical" ? "width" : "height",
      resizeType: this.split === "vertical" ? "left" : "top",
    };
  },
  methods: {
    onClick() {
      if (!this.hasMoved) {
        this.size = 200;
        this.$emit("resize", this.size);
      }
    },
    onMouseDown() {
      this.active = true;
      this.hasMoved = false;
    },
    onMouseUp() {
      this.active = false;
    },
    onMouseMove(e) {
      if (e.buttons === 0 || e.which === 0) {
        this.active = false;
      }
      if (this.active) {
        let offset = 0;
        let target = e.currentTarget;
        if (this.split === "vertical") {
          while (target) {
            offset += target.offsetLeft;
            target = target.offsetParent;
          }
        } else {
          while (target) {
            offset += target.offsetTop;
            target = target.offsetParent;
          }
        }
        const currentPage = this.split === "vertical" ? e.pageX : e.pageY;
        // const targetOffset =
        //   this.split === 'vertical'
        //     ? e.currentTarget.offsetWidth
        //     : e.currentTarget.offsetHeight;
        const size = currentPage - offset;
        // console.log("this.maxSize: ", this.maxSize);
        if (size > this.minSize) {
          if (this.maxSize > 0) {
            if (size < this.maxSize + offset) {
              this.size = size;
            }
          } else {
            this.size = size;
          }
        }
        this.$emit("resize", this.size);
        this.hasMoved = true;
      }
    },
  },
};
</script>

<style scoped>
.clearfix:after {
  visibility: hidden;
  display: block;
  font-size: 0;
  content: " ";
  clear: both;
  height: 0;
}
.vue-splitter-container {
  height: 100%;
  position: relative;
}
.vue-splitter-container-mask {
  z-index: 9999;
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
}
</style>
