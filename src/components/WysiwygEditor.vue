<template>
  <div class="">
    <IconsSprite></IconsSprite>
    <div class="toolbar">
      <button class="button _bg" @click="undo" :disabled="!canUndo">
        <svg class="button-icon">
          <use xlink:href="#back"></use>
        </svg>
      </button>
      <button class="button _bg" @click="redo" :disabled="!canRedo">
        <svg class="button-icon _reverse">
          <use xlink:href="#back"></use>
        </svg>
      </button>
      <button class="button _bg" @click="formatText('h1')">
        <svg class="button-icon">
          <use xlink:href="#h1"></use>
        </svg>
      </button>
      <button class="button _bg" @click="formatText('p')">
        <svg class="button-icon">
          <use xlink:href="#p"></use>
        </svg>
      </button>
      <button class="button _bg" @click="addImage">
        <svg class="button-icon">
          <use xlink:href="#img"></use>
        </svg>
      </button>
      <button class="button" @click="copyHtml">{{ isHtmlCopied ? 'HTML скопирован!' : 'Скопировать HTML' }}</button>
      <button class="button" @click="replaceHtml()">Заменить HTML из буфера обмена</button>
    </div>
    <div class="container editor" contenteditable="true" ref="editor" @input="saveState" v-html="htmlContent">
    </div>
  </div>
</template>

<style scoped lang="scss">
  .toolbar {
    width: 100%;

    display: flex;
    justify-content: center;
    gap: 12px;

    margin-bottom: 1rem;

    .button {
      min-width: 42px;
      height: 38px;

      font-family: var(--font-family);
      font-weight: 400;
      font-size: 15px;
      line-height: 153%;
      color: #639eff;

      background-color: transparent;
      border: none;
      cursor: pointer;

      &[disabled] {
        opacity: 0.5;
      }

      &._bg {
        border-radius: 4px;
        background: #282828;
      }

      &-icon {
        width: 20px;
        height: 20px;

        &._reverse {
          transform: scaleX(-1);
        }
      }
    }
  }
</style>

<script setup>
  import { ref, computed, onMounted } from "vue"
  import IconsSprite from "./IconsSprite.vue"



  const props = defineProps({
    html: {
      type: String,
      required: false
    }
  })

  let editor = ref()
  let history = ref([props.html])
  let historyIndex = ref(0)

  let htmlContent = ref(props.html)


  onMounted(() => {
    editor.value.addEventListener('paste', function (event) {
      event.preventDefault()

      const text = event.clipboardData.getData('text/plain')

      document.execCommand('insertText', false, text)
    })
  })



  let canUndo = computed(() => {
    return historyIndex.value > 0
  })

  let canRedo = computed(() => {
    return historyIndex.value < history.value.length - 1
  })



  function formatText(newTag) {
    const selection = window.getSelection()

    if (selection.rangeCount === 0) {
      console.log('Нет выделения')
      return
    }

    const range = selection.getRangeAt(0)
    const selectedNode = range.startContainer

    if (selectedNode.nodeType !== Node.TEXT_NODE) {
      console.log('Курсор не находится внутри текстового блока')
      return
    }

    const parent = selectedNode.parentNode
    const innerHtml = parent.innerHTML

    const newElement = document.createElement(newTag)
    newElement.innerHTML = innerHtml

    parent.parentNode.replaceChild(newElement, parent)

    saveState()
  }
  function addImage() {
    const url = prompt('Введите URL изображения')
    if (url) {
      document.execCommand('insertImage', false, url)
      saveState()
    }
  }
  function saveState() {
    const html = editor.value.innerHTML
    if (historyIndex.value === history.value.length - 1) {
      history.value.push(html)
    } else {
      history.value = history.value.slice(0, historyIndex.value + 1)
      history.value.push(html)
    }
    historyIndex.value++
    console.log('history.value: ', history.value)
  }
  function undo() {
    if (canUndo.value) {
      historyIndex.value--
      editor.value.innerHTML = history.value[historyIndex.value]
    }
  }
  function redo() {
    if (canRedo.value) {
      historyIndex.value++
      editor.value.innerHTML = history.value[historyIndex.value]
    }
  }

  let isHtmlCopied = ref(false)
  function copyHtml() {
    const html = editor.value.innerHTML
    navigator.clipboard.writeText(html).then(() => {
      console.log('HTML скопирован в буфер обмена!')
      isHtmlCopied.value = true
      setTimeout(() => {
        isHtmlCopied.value = false
      }, 1500)
    })
  }



  async function replaceHtml() {
    try {
      htmlContent.value = ""
      const clipboardItems = await navigator.clipboard.read()
      for (const clipboardItem of clipboardItems) {
        const types = clipboardItem.types
        for (const type of types) {
          const blob = await clipboardItem.getType(type)
          const htmlText = await blob.text()
          htmlContent.value += htmlText
        }
      }
    } catch (err) {
      console.error('Ошибка при чтении буфера обмена: ', err)
    }
  }
</script>
