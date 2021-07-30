<template>
  <div :id="'wrapper_' + id"></div>
</template>

<script>
import * as d3 from "d3";
import { gsap } from "gsap/all";
import { ScrollTrigger } from "gsap/ScrollTrigger";
gsap.registerPlugin(ScrollTrigger);

export default {
  props: {
    id: {
      type: Number,
    },
    dataset: {
      type: Object, // type: Array
    },
  },
  computed: {
    dimensions() {
      let dimensions = {
        width: this.windowWidth,
        height: 400,
        margin: {
          top: 20,
          right: 20,
          bottom: 20,
          left: 20,
        },
      };

      dimensions.boundedWidth =
        dimensions.width - dimensions.margin.left - dimensions.margin.right;
      dimensions.boundedHeight =
        dimensions.height - dimensions.margin.top - dimensions.margin.bottom;

      return dimensions;
    },
    pack() {
      return d3
        .pack()
        .size([this.dimensions.boundedWidth, this.dimensions.boundedHeight])
        .padding(1)(d3.hierarchy(this.dataset).sum((d) => d.value));
    },
    simulation() {
      return d3
        .forceSimulation(this.pack.leaves())
        .force("collision", d3.forceCollide().radius(this.r))
        .force("x", d3.forceX(this.dimensions.boundedWidth / 2).strength(0.01))
        .force(
          "y",
          d3.forceY(this.dimensions.boundedHeight / 2).strength(0.02)
        );
    },
  },
  mounted() {
    d3.select(`#wrapper_${this.id}`)
      .append("svg")
      .attr("viewBox", [0, 0, this.dimensions.width, this.dimensions.height])
      .attr("overflow", "visible");

    this.drawChart();
  },
  created() {
    window.addEventListener("resize", this.resizeChart);
  },
  beforeDestroyed() {
    window.removeEventListener("resize", this.resizeChart);
  },
  data() {
    return {
      windowWidth:
        window.innerWidth > 1000
          ? window.innerWidth * 0.6
          : window.innerWidth > 700
          ? window.innerWidth * 0.8
          : window.innerWidth * 1.2,
    };
  },
  methods: {
    resizeChart() {
      this.windowWidth =
        window.innerWidth > 1000
          ? window.innerWidth * 0.6
          : window.innerWidth > 700
          ? window.innerWidth * 0.8
          : window.innerWidth * 1.2;

      d3.select(`#wrapper_${this.id} svg`).remove();

      d3.select(`#wrapper_${this.id}`)
        .append("svg")
        .attr("viewBox", [0, 0, this.dimensions.width, this.dimensions.height])
        .attr("overflow", "visible");

      this.drawChart();
    },
    drawChart() {
      const bounds = d3
        .select(`#wrapper_${this.id} svg`)
        .append("g")
        .style(
          "transform",
          `translate(${this.dimensions.margin.left}, ${this.dimensions.margin.top})`
        );

      const xScale = d3
        .scaleLinear() // .scaleSymLog() // .scalePoint()
        .domain([0, 1]) // if .scalePoint(), use arr.map(i => i)
        .range([0, this.dimensions.boundedWidth]);

      const xAxis = bounds
        .append("g")
        .attr("transform", `translate(0,${this.dimensions.boundedHeight})`)
        .call(d3.axisBottom(xScale)); //.tickFormat(d3.format(".0%")))

      xAxis.selectAll("path").attr("visibility", "hidden");
      xAxis.selectAll("line").attr("visibility", "hidden");

      xAxis
        .selectAll("text")
        .attr("font-family", "Montserrat")
        .attr("color", "gray")
        .attr("transform", "translate(0,15)");

      bounds
        .append("text")
        .attr("text-anchor", "middle")
        .attr("x", this.dimensions.boundedWidth / 2)
        .attr("y", this.dimensions.boundedHeight + 20)
        .attr("font-size", 10)
        .attr("fill", "gray")
        .text("Axis Title");

      const yScale = d3
        .scaleLinear() // .scaleSymLog() // .scalePoint()
        .domain([0, 1]) // if .scalePoint(), use arr.map(i => i)
        .range([this.dimensions.boundedHeight, 0]);

      const yAxis = bounds.append("g").call(d3.axisLeft(yScale)); //.tickFormat(d3.format(".0%")))

      yAxis.selectAll("path").attr("visibility", "hidden");
      yAxis.selectAll("line").attr("visibility", "hidden");

      yAxis
        .selectAll("text")
        .attr("font-family", "Montserrat")
        .attr("color", "gray");

      const zScale = d3.scaleSqrt().domain([0, 1]).range([1, 10]);

      d3.selection.prototype.moveToFront = function () {
        return this.each(function () {
          this.parentNode.appendChild(this);
        });
      };

      d3.selection.prototype.moveToBack = function () {
        return this.each(function () {
          var firstChild = this.parentNode.firstChild;
          if (firstChild) {
            this.parentNode.insertBefore(this, firstChild);
          }
        });
      };

      const shapes = bounds
        .append("g")
        .selectAll("shapes")
        .data(this.dataset)
        .enter();

      const shape = shapes
        .append("path") // .append("rect") // .append("circle") // .append("line")
        .attr("class", "shapes")
        .attr("id", (d) => `shape_${d.name}`);

      shapes
        .append("text")
        .attr("x", 6)
        .attr("dy", 12)
        .append("textPath")
        .attr("xlink:href", (d) => `#shape_${d.name}`)
        .attr("startOffset", "20%")
        .attr("text-anchor", "middle")
        .attr("font-size", 11)
        .text((d) => d.name);

      shape.append("title").text((d) => d.description);

      const shapes = bounds
        .append("g")
        .selectAll("shapes")
        .data(this.pack.descendants().filter((d) => d.height !== 1))
        .join("g")
        .attr("id", (d, i) => `shape_${i}_${this.id}`);

      const shape = shapes
        .append("circle")
        .attr("transform", (d) => `translate(0,${d.category})`)
        .attr("fill", "#00a0de")
        .attr("cx", (d) => d.x)
        .attr("cy", (d) => d.y)
        .attr("r", this.r);

      this.simulation.on("tick", () => {
        shape
          .attr("cx", (d) => d.x)
          .attr("cy", (d) => d.y)
          .attr("r", this.r);
      });

      this.simulation
        .force("x", d3.forceX((d) => xScale(d.value)).strength(0.2))
        .force("y", d3.forceY(yScale(0)).strength(0.1))
        .alpha(1)
        .restart();

      this.animation();
    },
    animation() {
      const tl = gsap.timeline({
        scrollTrigger: {
          trigger: `#wrapper_${this.id}`,
          pin: true,
        },
      });

      gsap.utils.toArray(".shapes").forEach((el, i) => {
        tl.to(el, {
          attr: {
            d: ``,
          },
        });
      });
    },
  },
  watch: {
    dataset() {
      d3.select(`#wrapper_${this.id} svg`).remove();

      d3.select(`#wrapper_${this.id}`)
        .append("svg")
        .attr("viewBox", [0, 0, this.dimensions.width, this.dimensions.height])
        .attr("overflow", "visible");

      this.drawChart();
    },
  },
};
</script>

<style lang="scss" scoped></style>
