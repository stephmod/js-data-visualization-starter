<template>
  <div class="maincontainer">
    <h1>Starter</h1>
    <p>Starter for data visualization tool.</p>
    <div class="frame">
      <div class="side">
        <ul>
          <li
            v-scroll-to="{ el: '#s1', offset: -150 }"
            @click="updateSelection('s1')"
            class="s1"
          >
            Starter Component
          </li>
        </ul>
      </div>
      <div class="main">
        <div v-if="!dataLoaded && !dataError">
          <p>Loading...</p>
        </div>
        <div v-if="dataError">
          <p style="color: red">{{ errorMsg }}</p>
        </div>
        <div v-if="dataLoaded">
          <h2 id="s1" class="s1">Starter Component</h2>
          <StarterComponent :id="1" :dataset="dataset" class="s1" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { token } from "@/main";
import StarterComponent from "@/components/StarterComponent.vue";

export default {
  name: "Starter",
  components: { StarterComponent },
  data() {
    return {
      dataset: {},
      dataError: false,
      dataLoaded: false,
      errorMsg: "Error in loading data.",
      section: "s1",
    };
  },
  async created() {
    this.fetchData();
    document.addEventListener("scroll", this.handleScroll);
  },
  destroyed() {
    document.removeEventListener("scroll", this.handleScroll);
  },
  mounted() {
    document.getElementsByClassName(this.section)[0].style.backgroundColor =
      "rgba(231, 242, 249, 0.5)";
  },
  methods: {
    async fetchData() {
      const data_in = await fetch(``, { method: "GET" })
        .then((response) => response.json())
        .catch((err) => console.log(err));

      this.dataset = Object.freeze(data_in);

      this.dataset.length === undefined
        ? (this.dataError = true)
        : (this.dataLoaded = true);
    },
    updateSection(val) {
      this.section = val;
    },
    handleScroll() {
      var allSections = document.querySelectorAll(".side ul li");

      allSections.forEach((i) => {
        var sect = document.getElementsByClassName(i.className);
        Array.from(sect)
          .filter((k) => k.localName != li)
          .forEach((j) => {
            this.elView(j) == true ? (this.section = i.className) : null;
          });
      });

      Object.values(allSections).filter(
        (i) => i.className == this.section
      )[0].style.backgroundColor = "rgba(231, 242, 249, 0.5)";

      Object.values(allSections)
        .filter((i) => i.className != this.section)
        .forEach((j) => (j.style.backgroundColor = ""));
    },
    elView() {
      var top = el.offsetTop;
      var height = el.offsetHeight;

      while (el.offsetParent) {
        el = el.offsetParent;
        top += el.offsetTop;
      }

      return (
        top >= window.pageYOffset &&
        top + height <= window.pageYOffset + window.innerHeight - 200
      );
    },
  },
};
</script>

<style lang="scss" scoped>
.maincontainer {
  padding: 0 10% 10% 50px;
  max-width: 1000px;
  margin: 0 auto;
  overflow-y: unset;
  overflow-x: unset;
  .frame {
    display: flex;
    flex-wrap: wrap;
    @media screen and (min-width: 720px) {
      position: relative;
      -webkit-box-orient: vertical;
      -webkit-box-direction: normal;
      -ms-flex-direction: column;
      flex-direction: column;
    }
    .side {
      @media screen and (min-width: 720px) {
        position: sticky;
        position: -webkit-sticky;
        top: 100px;
        align-self: flex-start;
        -ms-flex-item-align: start;
        -webkit-box-flex: 0;
        flex: 0 0 220px;
        margin-left: -60px;
      }
      margin: 2px 50px 40px 0;
      border-radius: 10px;
      box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1), 0 20px 40px #e7f2f9;
      transition: 0.8s cubic-bezier(0.2, 0.8, 0.2, 1);
      padding-bottom: 12px;
      & ul {
        list-style-type: none;
        margin: 0;
        padding-left: 0;
        & li {
          display: block;
          text-align: left;
          padding: 2px 0 2px 20px;
          cursor: pointer;
          &:last-of-type {
            border-bottom: 0;
          }
          &:hover {
            background-color: rgba(216, 241, 233, 0.5);
          }
        }
      }
    }
    .main {
      width: 100%;
      flex: 1;
      min-width: 0;
      box-sizing: inherit;
    }
  }
}
</style>
