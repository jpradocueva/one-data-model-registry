<template>
  <div class="v-table-dynamic">
    <div
      v-if="tableData && tableData.rows && tableData.rows.length > 0"
      :style="{ minWidth: minWidth + 'px', maxWidth: maxWidth + 'px' }"
    >
      <!-- Table Tools -->
      <div class="v-table-tools flex-c-s" v-if="enableTools">
        <vue-input
          v-if="enableSearch"
          class="tools-search"
          v-model="searchValue"
          placeholder="Search: (Tags +/or Name +/or Type +/or Submitter)"
        ></vue-input>
      </div>
      <table
        class="v-table"
        :class="{ 'v-show-border': tableBorder }"
        :style="{ minWidth: minWidth + 'px', maxWidth: maxWidth + 'px' }"
      >
        <!-- Table Header -->
        <thead
          v-if="headerInfirstRow"
          class="v-table-row is-header"
          :class="{ 'is-striped': rowStripe, 'v-show-border': tableBorder }"
          :style="{ height: headerHeight + 'px' }"
          @click="onClickRow(tableData.rows[0], 0)"
        >
          <th
            v-for="(tableCell, j) in tableData.rows[0].cells"
            :key="j"
            class="table-header-cell"
            :class="{
              'v-show-border': tableBorder,
              'is-header': j === 0 && headerInfirstColumn,
            }"
            :style="getCellStyle(0, j)"
            @click="onClickCell(tableCell, 0, j)"
          >
            <span
              class="table-header-cell-content"
              :style="{
                whiteSpace: whiteSpace,
                wordWrap: wordWrap,
                textOverflow: textOverflow,
              }"
              >{{ tableCell.data }}</span
            >
          </th>
        </thead>
        <!-- Table Body -->
        <tbody class="v-table-body" :style="{ height: height }">
          <tr
            v-for="(tableRow, i) in tableData.rows"
            :key="i"
            v-show="
              tableRow.show &&
                !tableRow.filtered &&
                !(pagination && !tableRow.inPage) &&
                !(i === 0 && headerInfirstRow)
            "
            class="v-table-row"
            :class="{
              'is-striped': rowStripe && i % 2 === 0,
              'v-show-border': tableBorder,
            }"
            :style="{ height: rowHeight + 'px' }"
            @click="onClickRow(tableRow, tableRow.index)"
          >
            <td
              v-for="(tableCell, j) in tableRow.cells"
              :key="j"
              class="table-cell"
              :class="{
                'v-show-border': tableBorder,
                'is-header': j === 0 && headerInfirstColumn,
              }"
              :style="getCellStyle(tableRow.index, j)"
              @click="onClickCell(tableCell, tableRow.index, j)"
            >
              <a
                v-show="j === 1"
                :href="
                  `https://raw.githubusercontent.com/one-data-model/playground/master/odmObject/odmobject-${tableCell.data.toLowerCase()}.sdf.json`
                "
                class="table-cell-content"
                :class="{ 'fill-width': i !== 0 }"
                :style="{
                  whiteSpace: whiteSpace,
                  wordWrap: wordWrap,
                  textOverflow: textOverflow,
                }"
                :contenteditable="isEditable(tableRow.index, j)"
                :id="tableCell.key"
                rel="noopener noreferrer"
                target="_blank"
                @blur="onCellBlur(tableCell, tableRow.index, j)"
                @keydown.enter.stop.prevent="onCellKeyEnter"
                >{{ tableCell.data }}</a
              >
              <span
                v-show="j !== 1"
                class="table-cell-content"
                :class="{ 'fill-width': i !== 0 }"
                :style="{
                  whiteSpace: whiteSpace,
                  wordWrap: wordWrap,
                  textOverflow: textOverflow,
                }"
                :contenteditable="isEditable(tableRow.index, j)"
                :id="tableCell.key"
                @blur="onCellBlur(tableCell, tableRow.index, j)"
                @keydown.enter.stop.prevent="onCellKeyEnter"
                >{{ tableCell.data }}</span
              >
            </td>
          </tr>
        </tbody>
      </table>
      <!-- Table Pagination -->
      <div class="table-pagination" v-if="pagination">
        <vue-pagination
          :page-size="pageSize"
          :page-sizes="pageSizes"
          :total="totalPages"
          :disabled="!pagination"
          @current-page="onPageChange"
          @size="onPageSizeChange"
          ref="tablePagination"
        ></vue-pagination>
      </div>
    </div>
  </div>
</template>

<script>
import { unemptyArray, is2DMatrix } from "../utils/array.js";
import { unique } from "../utils/unique.js";
import VueInput from "./VueInput.vue";
import VuePagination from "./VuePagination.vue";
import "../assets/css/flex.css";
import "../assets/iconfont/iconfont.css";
const trim = require("lodash.trim");

const wordWrapList = ["normal", "break-word"];
const whiteSpaceList = ["nowrap", "normal", "pre", "pre-wrap", "pre-line"];
const textOverflowList = ["clip", "ellipsis"];

export default {
  name: "VueTableDynamic",
  data() {
    return {
      tableData: {},
      searchValue: "",
      activatedSort: {},
      totalPages: 0,
      pageSize: 0,
    };
  },
  props: {
    params: {
      type: Object,
      default: () => {
        return {};
      },
    },
  },
  computed: {
    sourceData() {
      if (this.params && Array.isArray(this.params.data)) {
        return this.params.data;
      }
      return [];
    },
    tableBorder() {
      return !!(this.params && this.params.border);
    },
    rowStripe() {
      return !!(this.params && this.params.stripe);
    },
    highlightConfig() {
      if (
        this.params &&
        this.params.highlight &&
        typeof this.params.highlight === "object"
      ) {
        return this.params.highlight;
      }
      return {};
    },
    highlightedColor() {
      if (
        this.params &&
        this.params.highlightedColor &&
        typeof this.params.highlightedColor === "string"
      ) {
        return this.params.highlightedColor;
      }
      return "#EBEBEF";
    },
    wordWrap() {
      if (
        this.params &&
        this.params.wordWrap &&
        wordWrapList.includes(this.params.wordWrap)
      ) {
        return this.params.wordWrap;
      }
      return wordWrapList[0];
    },
    whiteSpace() {
      if (
        this.params &&
        this.params.whiteSpace &&
        whiteSpaceList.includes(this.params.whiteSpace)
      ) {
        return this.params.whiteSpace;
      }
      return whiteSpaceList[1];
    },
    textOverflow() {
      if (
        this.params &&
        this.params.textOverflow &&
        textOverflowList.includes(this.params.textOverflow)
      ) {
        return this.params.textOverflow;
      }
      return textOverflowList[0];
    },
    headerInfirstRow() {
      return !!(this.params && this.params.header === "row");
    },
    headerInfirstColumn() {
      return !!(this.params && this.params.header === "column");
    },
    showCheck() {
      return !!(this.params && this.params.showCheck);
    },
    enableSearch() {
      return !!(this.params && this.params.enableSearch);
    },
    enableTools() {
      return this.enableSearch;
    },
    minWidth() {
      if (
        this.params &&
        typeof this.params.minWidth === "number" &&
        this.params.minWidth > 0
      ) {
        return this.params.minWidth;
      }
      return 300;
    },
    headerHeight() {
      if (
        this.params &&
        typeof this.params.headerHeight === "number" &&
        this.params.headerHeight >= 24
      ) {
        return this.params.headerHeight;
      }
      return 30;
    },
    rowHeight() {
      if (
        this.params &&
        typeof this.params.rowHeight === "number" &&
        this.params.rowHeight >= 24
      ) {
        return this.params.rowHeight;
      }
      return "auto";
    },
    height() {
      if (
        this.params &&
        typeof this.params.height === "number" &&
        this.params.height > this.rowHeight
      ) {
        return this.params.height - this.rowHeight + "px";
      }
      return "auto";
    },
    columnWidth() {
      if (this.params && unemptyArray(this.params.columnWidth)) {
        let obj = {};
        this.params.columnWidth.forEach((c) => {
          if (c && typeof c.column === "number" && c.column >= 0) {
            if (typeof c.width === "number" && c.width >= 0) {
              obj[c.column] = c.width + "px";
            } else if (
              typeof c.width === "string" &&
              /^(\d+\.?\d+?)%$/.test(c.width)
            ) {
              obj[c.column] = c.width;
            }
          }
        });
        return obj;
      }
      return {};
    },
    editConfig() {
      if (
        this.params &&
        this.params.edit &&
        typeof this.params.edit === "object"
      ) {
        return this.params.edit;
      }
      return {};
    },
    filterConfig() {
      if (this.params && unemptyArray(this.params.filter)) {
        let filterObj = {};
        this.params.filter.forEach((f) => {
          if (
            f &&
            typeof f.column === "number" &&
            f.column >= 0 &&
            typeof f.method === "function" &&
            unemptyArray(f.content)
          ) {
            if (
              f.content.every((c) => {
                return (
                  c &&
                  typeof c.text === "string" &&
                  typeof c.value !== "undefined"
                );
              })
            ) {
              let content = f.content.map((c) => {
                return { ...c, checked: false, key: unique(`content-`) };
              });
              filterObj[f.column] = { ...f, content, key: unique(`filter-`) };
            }
          }
        });
        return filterObj;
      }
      return {};
    },
    pagination() {
      return !!(this.params && this.params.pagination);
    },
    pageSizeConfig() {
      if (
        this.params &&
        typeof this.params.pageSize === "number" &&
        this.params.pageSize > 0
      ) {
        return this.params.pageSize;
      }
      return 10;
    },
    pageSizes() {
      if (Array.isArray(this.params.pageSizes)) {
        return this.params.pageSizes;
      }
      return [10, 20, 50, 100];
    },
  },
  watch: {
    params: {
      handler(value) {
        this.searchValue = "";
        this.activatedSort = {};
      },
      deep: true,
      immediate: true,
    },
    sourceData: {
      handler() {
        this.initData();
      },
      deep: true,
      immediate: true,
    },
    searchValue(value) {
      if (!this.enableSearch) return;
      this.search(value);
    },
    headerInfirstRow(value) {
      if (value && this.tableData && this.tableData.rows.length) {
        this.tableData.rows[0].checked = false;
        this.tableData.rows[0].show = true;
      }
    },
    enableSearch(newVal, oldVal) {
      if (oldVal && !newVal) {
        this.clearSearch();
      }
    },
    pageSizeConfig: {
      handler(v) {
        if (
          v > 0 &&
          this.pageSize !== v &&
          this.params &&
          this.params.pagination
        ) {
          this.pageSize = v;
        }
      },
      immediate: true,
    },
  },
  beforeDestroy() {
    this.tableData = {};
    this.activatedSort = {};
  },
  methods: {
    /**
     * @function Table
     */
    initData() {
      if (this.params && is2DMatrix(this.sourceData)) {
        let table = {
          key: unique(`table-`),
          checked: false,
          rows: [],
          filteredRows: {},
        };
        for (let i = 0; i < this.sourceData.length; i++) {
          let tableRow = {
            key: unique(`table-`),
            checked: false,
            show: true,
            filtered: false,
            inPage: false,
            index: i,
          };
          tableRow.cells = this.sourceData[i].map((item) => {
            return { data: item, key: unique(`table-`), checked: false };
          });
          table.rows.push(tableRow);
        }
        this.tableData = table;
        this.$nextTick(this.updatePagination);
      }
    },
    /**
     * @function
     */
    updatePagination() {
      if (!this.pagination) return;
      if (
        !(
          this.tableData &&
          this.tableData.rows &&
          this.tableData.rows.length > 0
        )
      )
        return;
      const rowNum = this.getActivatedRowNum();
      if (rowNum === this.totalPages) {
        if (this.$refs && this.$refs.tablePagination) {
          this.$refs.tablePagination.initPages(this.totalPages);
        }
      } else {
        this.totalPages = rowNum;
      }
    },
    /**
     * @function
     * @param {Number} page
     */
    onPageChange(page) {
      if (!this.pagination) return;
      if (
        !(
          this.tableData &&
          this.tableData.rows &&
          this.tableData.rows.length > 0
        )
      )
        return;
      let start = (page - 1) * this.pageSize;
      let end = start + this.pageSize;

      let rows = this.tableData.rows.filter((row, index) => {
        if (index === 0 && this.headerInfirstRow) return false;
        return row.show && !row.filtered;
      });
      rows.forEach((row, index) => {
        row.inPage = !!(index >= start && index < end);
      });
    },
    /**
     * @function
     */
    onPageSizeChange(size) {
      if (!this.pagination) return;
      this.pageSize = size;
    },
    /**
     * @function
     * @param {Number} tagetPage
     */
    toPage(tagetPage) {
      if (!this.pagination) return;
      if (!(typeof tagetPage === "number" && tagetPage > 0)) return;

      if (this.$refs && this.$refs.tablePagination) {
        this.$refs.tablePagination.toPage(tagetPage);
      }
    },
    /**
     * @function Cell
     */
    getCellStyle(rowIndex, columnIndex) {
      let style = {};
      if (this.isHighlighted(rowIndex, columnIndex)) {
        style.backgroundColor = this.highlightedColor;
      }

      if (columnIndex === 6) {
        return {
          ...style,
          flexGrow: 3,
          flexShrink: 3,
          flexBasis: "30%",
        };
      } else if (this.columnWidth[columnIndex]) {
        return {
          ...style,
          flexGrow: 0,
          flexShrink: 0,
          flexBasis: this.columnWidth[columnIndex],
        };
      } else {
        return {
          ...style,
          flexGrow: 1,
          flexShrink: 1,
          flexBasis: "0%",
        };
      }
    },

    isEditable(rowIndex, columnIndex) {
      if (
        !(
          this.editConfig &&
          (this.editConfig.row ||
            this.editConfig.column ||
            this.editConfig.cell)
        )
      )
        return false;
      if (this.headerInfirstRow && rowIndex === 0) return false;
      if (this.headerInfirstColumn && columnIndex === 0) return false;

      if (
        this.editConfig.row === "all" ||
        this.editConfig.column === "all" ||
        this.editConfig.cell === "all"
      )
        return true;

      if (
        Array.isArray(this.editConfig.row) &&
        (this.editConfig.row.includes(rowIndex) ||
          this.editConfig.row.includes(rowIndex - this.sourceData.length))
      ) {
        return true;
      }

      if (
        Array.isArray(this.editConfig.column) &&
        (this.editConfig.column.includes(columnIndex) ||
          this.editConfig.column.includes(
            columnIndex - this.sourceData[0].length
          ))
      ) {
        return true;
      }

      if (
        Array.isArray(this.editConfig.cell) &&
        this.editConfig.cell.length > 0
      ) {
        return this.editConfig.cell.some((item) => {
          return (
            Array.isArray(item) &&
            item.length >= 2 &&
            (item[0] === rowIndex ||
              item[0] === rowIndex - this.sourceData.length) &&
            (item[1] === columnIndex ||
              item[1] === columnIndex - this.sourceData[0].length)
          );
        });
      }

      return false;
    },
    isHighlighted(rowIndex, columnIndex) {
      if (
        !(
          this.highlightConfig &&
          (this.highlightConfig.row ||
            this.highlightConfig.column ||
            this.highlightConfig.cell)
        )
      )
        return false;

      if (
        Array.isArray(this.highlightConfig.row) &&
        (this.highlightConfig.row.includes(rowIndex) ||
          this.highlightConfig.row.includes(rowIndex - this.sourceData.length))
      ) {
        return true;
      }

      if (
        Array.isArray(this.highlightConfig.column) &&
        (this.highlightConfig.column.includes(columnIndex) ||
          this.highlightConfig.column.includes(
            columnIndex - this.sourceData[0].length
          ))
      ) {
        return true;
      }

      if (
        Array.isArray(this.highlightConfig.cell) &&
        this.highlightConfig.cell.length > 0
      ) {
        return this.highlightConfig.cell.some((item) => {
          return (
            Array.isArray(item) &&
            item.length >= 2 &&
            (item[0] === rowIndex ||
              item[0] === rowIndex - this.sourceData.length) &&
            (item[1] === columnIndex ||
              item[1] === columnIndex - this.sourceData[0].length)
          );
        });
      }

      return false;
    },
    /**
     * @function Cell
     * @param {Object} tableCell
     * @param {Number} rowIndex
     * @param {Number} columnIndex
     */
    onCellBlur(tableCell, rowIndex, columnIndex) {
      if (!this.isEditable(rowIndex, columnIndex)) return;

      let cellEle = document.querySelector(`#${tableCell.key}`);
      if (cellEle && tableCell.data !== trim(cellEle.innerHTML)) {
        tableCell.data = trim(cellEle.innerHTML);
        this.$emit("cell-change", rowIndex, columnIndex, tableCell.data);
      }
    },
    onCellKeyEnter(e) {},
    /**
     * @function
     * @param {Number} columnIndex
     * @param {Array} checked
     * @param {Object} config
     */
    onFilter(columnIndex, checked, config) {
      if (!(this.tableData && this.tableData.rows)) return;

      let filteredArr = [];
      this.tableData.rows.forEach((row) => {
        if (row && row.cells && row.cells[columnIndex]) {
          let matched = checked.some((item) => {
            return config.method(item.value, row.cells[columnIndex]);
          });
          matched ? "" : filteredArr.push(row.index);
        }
      });
      this.tableData.filteredRows[columnIndex] = filteredArr;

      this.updateFilteredRows();
    },
    /**
     * @function
     */
    updateFilteredRows() {
      this.tableData.rows.forEach((row) => {
        row.filtered = Object.keys(this.tableData.filteredRows).some((key) => {
          return this.tableData.filteredRows[key].includes(row.index);
        });
        row.filtered = !!row.filtered;
      });
      this.$nextTick(this.updatePagination);
    },
    /**
     * @function
     * @param {Number} columnIndex
     */
    clearFilter(columnIndex) {
      if (typeof columnIndex === "number") {
        delete this.tableData.filteredRows[columnIndex];
        if (this.filterConfig && this.filterConfig[columnIndex]) {
          this.filterConfig[columnIndex].content.forEach((c) => {
            c.checked = false;
          });
        }
      } else {
        this.tableData.filteredRows = {};
        Object.keys(this.filterConfig).forEach((key) => {
          this.filterConfig[key].content.forEach((c) => {
            c.checked = false;
          });
        });
      }

      this.updateFilteredRows();
    },
    /**
     * @function
     * @param {String} searchValue
     */
    search(searchValue) {
      if (!(this.tableData && this.tableData.rows)) return;

      searchValue = String(searchValue);
      this.tableData.rows.forEach((row) => {
        if (row && row.cells) {
          if (!searchValue) {
            return (row.show = true);
          }
          let matched = row.cells.some((cell) => {
            return String(cell.data)
              .toLocaleLowerCase()
              .includes(searchValue.toLocaleLowerCase());
          });
          row.show = !!matched;
        }
      });
      this.$nextTick(this.updatePagination);
    },
    /**
     * @function
     */
    clearSearch() {
      if (!(this.tableData && this.tableData.rows)) return;
      this.tableData.rows.forEach((row) => {
        row ? (row.show = true) : "";
      });
    },
    /**
     * @function
     * @param {Array} rowIndexs
     */
    getData(rowIndexs) {
      let matrix = [];
      if (this.tableData && unemptyArray(this.tableData.rows)) {
        let tmpRows = {};
        this.tableData.rows.forEach((row, index) => {
          if (Array.isArray(rowIndexs)) {
            rowIndexs.includes(row.index) ? (tmpRows[row.index] = row) : "";
          } else {
            tmpRows[row.index] = row;
          }
        });
        for (let i = 0; i < this.tableData.rows.length; i++) {
          let rowData = this.getRowDataFromTableRow(tmpRows[i]);
          rowData.length > 0 ? matrix.push(rowData) : "";
        }
      }
      return matrix;
    },
    /**
     * @function
     * @param {Number} rowIndex
     * @param {Boolean} isCurrent
     */
    getRowData(rowIndex, isCurrent = false) {
      if (this.tableData && unemptyArray(this.tableData.rows)) {
        let row;
        if (isCurrent) {
          row = this.tableData.rows[rowIndex];
        } else {
          row = this.tableData.rows.find((r) => {
            return r.index === rowIndex;
          });
        }
        return this.getRowDataFromTableRow(row);
      }
      return [];
    },
    /**
     * @function
     * @param {Number} rowIndex
     * @param {Number} columnIndex
     * @param {Boolean} isCurrent
     */
    getCellData(rowIndex, columnIndex, isCurrent = false) {
      if (this.tableData && unemptyArray(this.tableData.rows)) {
        let row;
        if (isCurrent) {
          row = this.tableData.rows[rowIndex];
        } else {
          row = this.tableData.rows.find((r) => {
            return r.index === rowIndex;
          });
        }
        if (!(row && unemptyArray(row.cells))) return "";

        let cell = row.cells[columnIndex];
        if (cell && typeof cell.data !== "undefined") return cell.data;
        return "";
      }
      return "";
    },
    /**
     * @function
     * @param {Number} tableRow
     */
    getRowDataFromTableRow(tableRow) {
      let rowData = [];
      if (!(tableRow && unemptyArray(tableRow.cells))) return rowData;

      for (let i = 0; i < tableRow.cells.length; i++) {
        let cellData = tableRow.cells[i].data || "";
        rowData.push(cellData);
      }
      return rowData;
    },
    /**
     * @function
     */
    getActivatedRowNum(includeWhenHeaderInfirstRow = false) {
      if (this.tableData && unemptyArray(this.tableData.rows)) {
        let num = 0;
        this.tableData.rows.forEach((row, index) => {
          if (index === 0 && this.headerInfirstRow) {
            if (!includeWhenHeaderInfirstRow) return;
            return num++;
          }
          if (row.show && !row.filtered) num++;
        });
        return num;
      }

      return 0;
    },
  },
  components: { VueInput, VuePagination },
};
</script>

<style lang="scss" scoped>
$borderColor: #dcdfe6;

.v-table-dynamic {
  width: 100%;
  display: block;
  box-sizing: border-box;
  font-family: Helvetica, Arial, "Microsoft YaHei";
  font-size: 13px;
  color: rgb(162, 157, 170);
}
.v-table {
  box-sizing: border-box;
  border: none;
  margin-top: 3em;
  width: 100%;
}

.v-table.v-show-border {
  border-top: none;
}

.v-table-row {
  box-sizing: border-box;
  border: none;
  border-bottom: 1px solid rgb(31, 39, 57);
  background-color: #2c3446;
  min-height: 5em !important;
}
.v-table-row.is-header {
  overflow: hidden;
}
.v-table-row.is-header,
.table-cell.is-header {
  font-weight: 600;
}
.v-table-row.is-striped {
  background-color: #323c50;
}
.v-table-row.v-show-border {
  border-right: 1px solid rgb(31, 39, 57);
}

.v-table-row:hover {
  background-color: rgba(125, 125, 125, 0.5);
}

.table-cell {
  box-sizing: border-box;
  height: 100%;
  padding: 1.5em 1em;
  overflow: hidden;
  -webkit-flex: 1;
  -ms-flex: 1;
  flex: 1;
  border: none;
}

td:nth-child(1),
td:nth-child(2) {
  color: rgb(248, 99, 121);
  font-weight: bold;
  text-transform: uppercase;
}

td a,
td a:visited,
td a:focus {
  text-decoration: none;
  color: rgb(248, 99, 121);
}

.table-check.v-show-border,
.table-cell.v-show-border {
  border-left: 1px solid rgb(31, 39, 57);
}

.table-cell-content {
  box-sizing: border-box;
  min-width: 12px;
  min-height: 10px;
  outline: 0;
  text-align: left;
}

.table-cell-content.fill-width {
  width: 100%;
}

.table-cell-content {
  position: relative;
  overflow: hidden;
}

.table-header-cell {
  box-sizing: border-box;
  height: 100%;
  padding: 1em 8px;
  overflow: hidden;
  -webkit-flex: 1;
  -ms-flex: 1;
  flex: 1;
  border: none;
  background-color: rgb(31, 39, 57);
  text-transform: uppercase;
  text-align: left;
}

.table-header-cell.v-show-border {
  border-left: none;
}

.table-header-cell-content {
  box-sizing: border-box;
  min-width: 12px;
  min-height: 10px;
  outline: 0;
  text-align: left;
  font-weight: bold;
  font-size: 1em;
  color: rgb(77, 195, 250);
}
.table-header-cell-content.fill-width {
  width: 100%;
}

.table-header-cell-content {
  position: relative;
  overflow: hidden;
  color: rgb(77, 195, 250);
}

.v-table-tools {
  padding: 8px 0px;
  margin-left: 1em;
}

.tools-search {
  width: 280px;
  margin-right: 8px;
}

.table-pagination {
  padding: 8px;
  background-color: rgba(125, 125, 125, 0.5);
  margin-top: 1em;
  margin-bottom: 1em;
}
</style>
