<template>
  <div @contextmenu="(e) => e.preventDefault()">
    <div class="wh100p txt" v-if="NotEdit" @click="cf">
      {{ modelValue }}
    </div>
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

defineProps({
  modelValue: String
})
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

.txt {
  display: flex;
  justify-content: center;
  align-items: center;
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
