<template>
  <label class="nogrow flexrow">
    <input
      type="radio"
      :name="radioGroup"
      class="nogrow"
      @change="select"
      ref="radio"
    />
    <MceEditor
      v-model="state.html"
      ref="editor"
      @save="emitHtml"
      :style="{ height: state.height, 'min-height': state.height }"
    />
  </label>
</template>

<style lang="less" scoped>
input[type='radio'] {
  flex-grow: 0;
  align-self: flex-start;
  margin: var(--ironsworn-spacer-lg);
}
</style>

<script lang="ts" setup>
import { reactive, ref } from 'vue'
import MceEditor from '../mce-editor.vue'

const props = defineProps<{
  radioGroup: string
}>()

const radio = ref<HTMLElement>()
const editor = ref<typeof MceEditor>()

const state = reactive({
  html: '',
  height: '35px',
})

function select() {
  editor.value?.enableEditor()
  state.height = '300px'
}

const $emit = defineEmits<{
  (e: 'change', html: string)
}>()
function emitHtml() {
  $emit('change', state.html)
}
</script>
