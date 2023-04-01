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
    setSelectClass()
  }
}, 100)

function mergeCells() {
  let isFirst = true
  for (let i = selectStratCell.row; i <= selectEndCell.row; i++) {
    for (let j = selectStratCell.col; j <= selectEndCell.col; j++) {
      if (isFirst) {
        data[i][j].we = data[i][j].we + Math.abs(selectStratCell.col - selectEndCell.col)
        data[i][j].he = data[i][j].he + Math.abs(selectStratCell.row - selectEndCell.row)
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
  for (let i = selectStratCell.row; i <= selectEndCell.row; i++) {
    for (let j = selectStratCell.col; j <= selectEndCell.col; j++) {
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
