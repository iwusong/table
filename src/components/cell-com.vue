<template>
  <div class="cell-div" @click="cf" >
    <span class="txt" v-if="NotEdit">
      {{ modelValue }}
    </span>
    <input
      v-else
      class="wh100p"
      :value="modelValue"
      @input="$emit('update:modelValue', $event.target.value)"
      ref="input"
      @blur="blurFun"
    />
  </div>
</template>

<script setup lang="ts">
import { nextTick, ref } from 'vue'

defineProps<{
  modelValue: string
}>()

defineEmits(['update:modelValue'])
let input = ref()

function cf() {
  NotEdit.value = !NotEdit.value
  nextTick(() => {
    input.value.focus()
    // eslint-disable-next-line no-debugger
  })
}

function blurFun() {
  NotEdit.value = !NotEdit.value
}

let NotEdit = ref(true)



</script>

<style scoped>
.wh100p {
  width: 100%;
  height: 100%;
}
.cell-div {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
.txt {
  line-height: 50px;
}

div,
input {
  box-sizing: border-box;
}

input:focus {
  background-color: #ecebeb;
  border: none;
  outline: none;

  box-shadow: 3px 3px 8px 2px;
}
</style>
