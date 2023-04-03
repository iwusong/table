<script setup lang="ts">
import { reactive, ref } from 'vue'
import CellCom from '@/components/cell-com.vue'
import { throttle } from 'lodash-es'

interface dataItem {
  value: string

  we: number
  he: number
  beMerged: boolean
  isSelect: boolean
}

interface selectCell {
  col: number
  row: number
}

const widthBase = 160
const HeightBase = 50
let selectStratCell: selectCell
let selectEndCell: selectCell

function asd(row: string) {
  return Array.from({ length: 5 }).map((item, index) => {
    return {
      value: row + (index + 1 + ''),
      beMerged: false,
      we: 1,
      he: 1,
      isSelect: false
    }
  })
}

function rightDownListen(e: MouseEvent, row: number, col: number) {
  if (e.buttons === 2) {
    resetSelect()
    selectStratCell = {
      row,
      col
    }
    rightIsPress.value = true
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
      col,
      row
    }
    resetSelect()
    setSelectClass()
  }
}, 100)

function mergeCells() {
  let isFirst = true
  let reverseSECell1 = reverseSECell()
  for (let i = reverseSECell1[0].row; i <= reverseSECell1[1].row; i++) {
    for (let j = reverseSECell1[0].col; j <= reverseSECell1[1].col; j++) {
      if (isFirst) {
        data[i][j].we = reverseSECell1[1].col- reverseSECell1[0].col+data[reverseSECell1[1].row][reverseSECell1[1].col].we
        data[i][j].he = reverseSECell1[1].row- reverseSECell1[0].row+data[reverseSECell1[1].row][reverseSECell1[1].col].he
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
let data: Array<Array<dataItem>> = reactive([asd('1'), asd('2'), asd('3')])
</script>

<template>
  <button>增加列</button>
  <span>---</span>
  <button @click="mergeCells">合并</button>
  <div style="height: 20px"></div>
  <div :class="{ cursorCrosshair: rightIsPress }" class="form-box">
    <div v-for="(i, iIndex) in data" :key="i">
      <template v-for="(j, jIndex) in i" :key="j">
        <cell-com
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
  </div>
</template>

<style scoped>
.form-box {
  width: 800px;
  position: relative;
}

.cursorCrosshair {
  cursor: crosshair;
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
  background-color: rgba(31, 225, 232, 0.3);
}
</style>
