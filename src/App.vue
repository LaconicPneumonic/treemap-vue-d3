<template>
  <div class="canvas">
    <transition-group tag="div">
      <div
        class="leaf"
        v-for="(leaf, index) in leaves"
        :key="leaf.data.name"
        :style="test(leaf, index)"
        :mouseover="mouseover(leaf)"
        :mouseleave="mouseleave(leaf)"
      ></div>
    </transition-group>
  </div>
</template>

<script>
import * as d3 from "d3";
import json from "@/assets/flare.json";

export default {
  data: function () {
    return {
      root: json,
      width: 500,
      height: 500,
      selected: {},
    };
  },
  methods: {
    color: d3.scaleSequential(d3.interpolateSinebow),
    mouseover: function (leaf) {
      leaf.zindex = 1;
    },
    mouseleave: function (leaf) {
      leaf.zindex = -1;
    },
    handleResize: function () {
      this.width = window.innerWidth;
      this.height = window.innerHeight;
    },
    test: function (leaf) {
      const ret = {
        border: "solid",
        position: "absolute",
        width: leaf.x1 - leaf.x0 + "px",
        height: leaf.y1 - leaf.y0 + "px",
        left: leaf.x0 + "px",
        top: leaf.y0 + "px",
        background: this.color(leaf.value / this.maxVal),
      };

      return ret;
    },
  },

  computed: {
    leaves: function () {
      const root = d3
        .hierarchy(this.root, (d) => d.children)
        .sum((d) => d.value)
        .sort((a, b) => b.value - a.value);

      d3
        .treemap()
        .size([this.width - 100, this.height - 100])
        .padding(0)(root);

      return root.leaves();
    },

    maxVal: function () {
      const ret = this.leaves
        .map((l) => l.value)
        .reduce((a, b) => {
          return a > b ? a : b;
        });

      return ret;
    },
  },

  created: function () {
    window.addEventListener("resize", this.handleResize);
    this.handleResize();
  },

  destroyed: function () {
    window.removeEventListener("resize", this.handleResize);
  },

  mounted: function () {},
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.canvas {
  position: relative;
}

/* box shadow technique sourced from https://tobiasahlin.com/blog/how-to-animate-box-shadow/ */
.leaf {
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
  -webkit-transition: all 0.6s cubic-bezier(0.165, 0.84, 0.44, 1);
  transition: all 0.6s cubic-bezier(0.165, 0.84, 0.44, 1);
  z-index: 0;
}
/* Pre-render the bigger shadow, but hide it */
.leaf::after {
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
  opacity: 0;
  -webkit-transition: all 0.6s cubic-bezier(0.165, 0.84, 0.44, 1);
  transition: all 0.6s cubic-bezier(0.165, 0.84, 0.44, 1);
  z-index: 0;
}

.leaf:hover {
  -webkit-transform: scale(1.25, 1.25);
  transform: scale(1.25, 1.25);
  z-index: 2;
}

/* Transition to showing the bigger shadow on hover */
.leaf:hover::after {
  opacity: 1;
  z-index: 2;
}
</style>
