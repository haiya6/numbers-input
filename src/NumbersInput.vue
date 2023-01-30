<script setup lang="ts">
import { ref } from 'vue'
import { onKeyStroke } from '@vueuse/core'

const emit = defineEmits<{
  (event: 'input', data: { done: boolean, value?: string }): void
}>()

const count = 4
const currentIndex = ref(-1)
const inputs = ref<HTMLInputElement[]>()
const values = ref(Array.from<string>({ length: count }).fill(''))

const numberFilter = (str: string) => str.replace(/[^\d]/g, '')

const setValue = (idx: number, value: string) => {
  values.value[idx] = value
  inputs.value![idx].value = value

  const result = values.value.join('')
  emit('input', {
    done: result.length === 4,
    value: result.length === 4 ? result : undefined
  })
}

/**
 * 上一个 input 聚焦
 */
const prevInputFocus = () => {
  if (currentIndex.value <= 0)
    return
  currentIndex.value--
  inputs.value![currentIndex.value].focus()
}

/**
 * 下一个 input 聚焦
 */
const nextInputFocus = () => {
  if (currentIndex.value >= count - 1)
    return
  currentIndex.value++
  inputs.value![currentIndex.value].focus()
}

/**
 * 处理粘贴
 * @param idx 
 * @param event 
 */
const onPaste = (idx: number, event: ClipboardEvent) => {
  event.preventDefault()
  const str = numberFilter(event.clipboardData?.getData('Text') ?? '')

  for(let i = 0; i < str.length; i++) {
    setValue(idx, str[i])
    if (idx >= count - 1)
      break
    else 
      idx++
  }

  inputs.value![idx].focus()
}

/**
 * 处理 input 事件
 */
const onInput = (idx: number, event: InputEvent) =>  {
  // 键盘输入
  if (event.inputType === 'insertText' || event.inputType === 'insertCompositionText') {
    const value = numberFilter(event.data ?? '')
    setValue(idx, value)
    if (value)
      nextInputFocus()
  }
}

/**
 * 监听键盘左键切换 input
 */
onKeyStroke('ArrowLeft', event => {
  if (currentIndex.value !== -1) {
    event.preventDefault() 
    prevInputFocus()
  }
})

/**
 * 切换键盘右键切换 input
 */
onKeyStroke('ArrowRight', event => {
  if (currentIndex.value !== -1) {
    event.preventDefault()
    nextInputFocus()
  }
})

/**
 * 键盘回退案件
 */
onKeyStroke('Backspace', event => {
  if (currentIndex.value !== -1) {
    event.preventDefault()
    if (values.value[currentIndex.value]) {
      setValue(currentIndex.value, '')
    } else {
      prevInputFocus()
    }
  }
})
</script>

<template>
  <div class="numbers-input">
    <div v-for="_, idx in 4" class="box-item">
      <input 
        class="item-input"
        type="text"
        maxlength="1"
        ref="inputs"
        @focus="currentIndex = idx"
        @blur="currentIndex = -1"
        @paste="onPaste(idx, $event)"
        @input="onInput(idx, $event as InputEvent)"
      >
    </div>
  </div>
</template>

<style lang="scss" scoped>
.numbers-input {
  display: flex;
  align-items: center;

  .box-item {
    width: 60px;
    height: 60px;
    background: #F5F5F5;
    border-radius: 4px;
    border: 1px solid #E3E6EC;

    & + .box-item {
      margin-left: 20px;
    }

    .item-input {
      font-family: Fang;
      font-size: 34px;
      width: 100%;
      height: 100%;
      text-align: center;
    }
  }
}
</style>
