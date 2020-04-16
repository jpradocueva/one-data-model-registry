<template>
  <div class="home">
    <header>
      <nav-menu></nav-menu>
    </header>
    <section>
      <vue-table-dynamic :params="params" ref="table"></vue-table-dynamic>
    </section>
    <vue-msg :timeout="2000" :top="100" :right="20" ref="vueMsg"></vue-msg>
  </div>
</template>

<script>
import NavMenu from "./NavMenu.vue";
import VueMsg from "vue-msgs";
import { saveAs } from "file-saver";
import { table, getBorderCharacters } from "table";
import data from "../../../data/convertjson.json";
const cloneDeep = require("lodash.clonedeep");

const defaultTableParams = {
  data: [
    [
      " #",
      `Name`,
      `URN`,
      `Version`,
      `Type`,
      `Submitted`,
      `Title | Description`,
    ],
  ],
  header: "row",
  height: "",
  headerHeight: 30,
  rowHeight: "auto",
  border: true,
  stripe: true,
  enableSearch: true,
  columnWidth: [{ column: 0, width: 50 }],
  sort: [0, 1],
  edit: {},
  highlight: {},
  pagination: true,
};

for (let i = 0; i < data.length; i++) {
  defaultTableParams.data.push([
    data[i].id,
    data[i].Name,
    data[i].URN,
    data[i].Version,
    data[i].Type,
    data[i].Submitter,
    data[i].Description,
  ]);
}

const tableHeaderTypes = ["", "row", "column"];

export default {
  name: "Home",
  data() {
    return {
      params: cloneDeep(defaultTableParams),
      scrollBarOpts: {
        scrollPanel: { scrollingX: false },
        bar: { background: "#DFDFDF", opacity: 0.8 },
      },
      widthIncrement: 1,
      heightIncrement: 1,
      rowHeightIncrement: 1,
    };
  },

  components: { NavMenu, VueMsg },
};
</script>

<style lang="scss" scoped>
.home {
  height: 100%;
  width: 100%;
  position: relative;
  header {
    width: 100%;
    height: 70px;
  }
  section {
    position: absolute;
    top: 5em;
    bottom: 0;
    left: 0;
    right: 0;
    padding: 20px 25px;
  }
  .downloader {
    height: 0;
    width: 0;
    visibility: hidden;
  }
}
</style>
<style>
.v-show-border {
  border: 1px solid #ccc;
}
</style>
