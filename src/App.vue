<script setup lang="ts">
import { nextTick, reactive, ref } from 'vue'
import CellCom from '@/components/cell-com.vue'
import { throttle } from 'lodash-es'
import type { dataItem } from '@/type'

interface selectCell {
  col: number
  row: number
}

const widthBase = 160
const HeightBase = 50
let selectStratCell = reactive<selectCell>({ row: 0, col: 0 })
let selectEndCell = reactive<selectCell>({ row: 0, col: 0 })

let data: Array<Array<dataItem>> = reactive([])
let mouseX = ref(0)
let mouseY = ref(0)
function initTable(row: number, col: number) {
  for (let i = 0; i < row; i++) {
    data.push(
      Array.from({ length: col }).map(() => {
        return {
          value: '',
          beMerged: false,
          we: 1,
          he: 1,
          isSelect: false
        }
      })
    )
  }
}

initTable(8, 6)

function rightDownListen(e: MouseEvent, row: number, col: number) {
  if (e.buttons === 2) {
    resetSelect()
    selectStratCell = {
      row,
      col
    }
    rightIsPress.value = true
    contextmenuShow.value = false
  }
}

function rightUpListen(e: MouseEvent, row: number, col: number) {
  if (e.button === 2) {
    rightIsPress.value = false
    selectEndCell = {
      row,
      col
    }
  }
}

let mouseMoveFunc = throttle((e: MouseEvent, row: number, col: number) => {
  if (rightIsPress.value) {
    selectEndCell = {
      row,
      col
    }
    resetSelect()
    setSelectClass()
    mouseX.value = e.clientX
    mouseY.value = e.clientY
  }
}, 100)

function mergeCells() {
  let isFirst = true
  let reverseSECell1 = reverseSECell()
  for (let i = reverseSECell1[0].row; i <= reverseSECell1[1].row; i++) {
    for (let j = reverseSECell1[0].col; j <= reverseSECell1[1].col; j++) {
      if (isFirst) {
        data[i][j].we =
          reverseSECell1[1].col -
          reverseSECell1[0].col +
          data[reverseSECell1[1].row][reverseSECell1[1].col].we
        data[i][j].he =
          reverseSECell1[1].row -
          reverseSECell1[0].row +
          data[reverseSECell1[1].row][reverseSECell1[1].col].he
        isFirst = false
      } else {
        data[i][j].beMerged = true
      }
    }
  }
  selectStratCell.row = 0
  selectStratCell.col = 0
  selectEndCell.col = 0
  selectEndCell.row = 0
  resetSelect()
  contextmenuShow.value = false
}

function setSelectClass() {
  let reverseSECell1 = reverseSECell()
  for (let i = reverseSECell1[0].row; i <= reverseSECell1[1].row; i++) {
    for (let j = reverseSECell1[0].col; j <= reverseSECell1[1].col; j++) {
      data[i][j].isSelect = true
    }
  }
}

function resetSelect() {
  for (const datum of data) {
    for (const dataItem of datum) {
      dataItem.isSelect = false
    }
  }
}

function reverseSECell() {
  let rows = Math.min(selectStratCell.row, selectEndCell.row)
  let rowe = Math.max(selectStratCell.row, selectEndCell.row)
  let cols = Math.min(selectStratCell.col, selectEndCell.col)
  let cole = Math.max(selectStratCell.col, selectEndCell.col)
  return [
    { row: rows, col: cols },
    { row: rowe, col: cole }
  ]
}

let rightIsPress = ref(false)
let contextmenuShow = ref(false)
function contextmenuFun(e: MouseEvent) {
  e.preventDefault()
  nextTick(() => {
    contextmenuShow.value = true
  })
}
function cellClick() {
  selectStratCell.row = 0
  selectStratCell.col = 0
  selectEndCell.col = 0
  selectEndCell.row = 0
  resetSelect()
  contextmenuShow.value = false
}
</script>

<template>
  <div style="height: 20px"></div>
  <div
    @contextmenu="contextmenuFun($event)"
    :class="{ cursorCrosshair: rightIsPress }"
    class="form-box"
  >
    <div v-for="(i, iIndex) in data" :key="i">
      <template v-for="(j, jIndex) in i" :key="j">
        <cell-com
          @click="cellClick"
          @mousedown="rightDownListen($event, iIndex, jIndex)"
          @mouseup="rightUpListen($event, iIndex, jIndex)"
          @mousemove="mouseMoveFunc($event, iIndex, jIndex)"
          v-if="!j.beMerged"
          :style="{
            width: j.we * widthBase + 'px',
            height: j.he * HeightBase + 'px',
            left: widthBase * jIndex + 'px',
            top: HeightBase * iIndex + 'px'
          }"
          class="cell"
          :class="{ 'is-select': j.isSelect }"
          v-model="j.value"
        />
      </template>
    </div>
    <div
      :style="{
        left: mouseX + 'px',
        top: mouseY + 'px'
      }"
      v-if="contextmenuShow"
      class="table-contextmenu"
    >
      <button @click="mergeCells">合并单元格</button>
      <button>取消合并</button>
      <button>权限设置</button>
      <button>样式设置</button>
    </div>
  </div>
</template>

<style scoped>
.form-box {
  width: 800px;
  position: relative;
}

.cursorCrosshair {
  cursor: cell;
}

.form-box div {
  box-sizing: border-box;
}

.cell {
  border: #181818 1px solid;
  display: inline-block;
  position: absolute;
}

.is-select {
  background-color: rgba(208, 213, 213, 0.6);
}
.table-contextmenu {
  width: 100px;
  background-color: #ffffff;
  position: fixed;
  box-shadow: 3px 3px 8px 2px #606565;
  display: flex;
  flex-direction: column;
  justify-content: center;
}
.table-contextmenu button {
  border: 0;
  cursor: pointer;
  padding: 8px 0;
  background-color: #fff;
}
.table-contextmenu button:hover {
  color: #09caee;
}
</style>
