<template>
  <div class="group flex w-full rounded" @click="focus"
    :class="{
      // Add top margin for headings
      'mt-12 group-first:mt-0': block.type === BlockType.H1,
      'mt-4 group-first:mt-0': block.type === BlockType.H2,
    }">
    <div class="h-full py-1.5 px-2 pl-4 text-center cursor-pointer transition-all duration-150 text-neutral-300 flex">
      <Tooltip value="<span class='text-neutral-400'><span class='text-white'>Click</span> to delete block</span>">
        <!-- <v-icon name="hi-trash" @click="emit('deleteBlock')" -->
        <v-icon name="hi-trash" @click="test"
          class="w-6 h-6 hover:bg-neutral-100 hover:text-neutral-400 p-0.5 rounded group-hover:opacity-100 opacity-0" />
      </Tooltip>
      <Tooltip value="<span class='text-neutral-400'><span class='text-white'>Click</span> to add block below</span>">
        <v-icon name="hi-plus" @click="emit('newBlock')"
          class="w-6 h-6 hover:bg-neutral-100 hover:text-neutral-400 p-0.5 rounded group-hover:opacity-100 opacity-0" />
      </Tooltip>
      <BlockMenu ref="menu"
        :block="block"
        @setBlockType="type => emit('setBlockType', type)"
        @clearSearch="clearSearch"
        />
    </div>
    <div class="w-full relative" :class="{ 'px-4 sm:px-0': block.type !== BlockType.Divider }">
      <!-- Actual content -->
      <editor v-if="props.block.type === BlockType.Text" ref="content"
        v-model="props.block.details.value" @keydown.capture="keyDownHandler"
        class="py-1.5" />
      <div v-else ref="content"
        :contenteditable="![BlockType.Divider].includes(block.type)" spellcheck="false"
        @blur="block.details.value=content.innerHTML"
        @keydown="keyDownHandler"
        @keyup="keyUpHandler"
        class="focus:outline-none focus-visible:outline-none w-full"
        :class="{
          'py-1.5': block.type === BlockType.Text,
          'py-1.5 text-4xl font-semibold': block.type === BlockType.H1,
          'py-1.5 text-3xl font-medium': block.type === BlockType.H2,
          'py-0 h-[1px] bg-neutral-300 mt-[1.2rem]': block.type === BlockType.Divider,
        }"
        :block-type="block.type"
        :data-ph="placeholder">
        {{ [BlockType.Divider].includes(block.type) ? '' : block.details.value }}
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Placeholder text styling */
[contenteditable='true']:empty:focus:before{
  content:attr(data-ph);
  color:#BBBBBB;
}
</style>

<script setup lang="ts">
import { ref, computed, watch, PropType, onMounted } from 'vue'
import DOMPurify from 'dompurify'
import { Block, BlockType } from '@/utils/types'
import BlockMenu from './BlockMenu.vue'
import Editor from './elements/Editor.vue'
import Tooltip from './elements/Tooltip.vue'

const props = defineProps({
  block: {
    type: Object as PropType<Block>,
    default: {
      type: BlockType.Text,
      details: {},
    },
  },
})

const emit = defineEmits([
  'deleteBlock',
  'newBlock',
  'moveToPrevChar',
  'moveToNextChar',
  'moveToPrevLine',
  'moveToNextLine',
  'merge',
  'split',
  'setBlockType',
])

const placeholder = computed(() => {
  if (props.block.type === BlockType.H1) return 'Heading 1'
  else if (props.block.type === BlockType.H2) return 'Heading 2'
  else if (props.block.type === BlockType.Divider) return ''
  else return 'Type \'/\' for commands'
})

const firstChild = computed(() => {
  if (props.block.type === BlockType.Text) {
    if (content.value.$el.firstChild.firstChild.childNodes.length > 1) {
      return content.value.$el.firstChild.firstChild.firstChild
    } else {
      return content.value.$el.firstChild.firstChild.firstChild
    }
  } else {
    return content.value.firstChild || content.value
  }
})

const lastChild = computed(() => {
  if (props.block.type === BlockType.Text) {
    if (content.value.$el.firstChild.firstChild.childNodes.length > 1) {
      return content.value.$el.firstChild.firstChild.lastChild
    } else {
      return content.value.$el.firstChild.firstChild.firstChild
    }
  } else {
    return content.value.firstChild || content.value
  }
})

const innerContent = computed(() => {
  if (props.block.type === BlockType.Text) {
    if (content.value.$el.firstChild.firstChild.childNodes.length > 1) {
      return content.value.$el.firstChild.firstChild.firstChild
    } else {
      return content.value.$el.firstChild.firstChild.firstChild
    }
    // console.log(content.value.$el.firstChild.firstChild.childNodes.length)
    // console.log(content.value.$el.firstChild.firstChild.parentElement)
    // console.log(content.value.$el.firstChild.firstChild.firstChild)
    return content.value.$el.firstChild.firstChild.firstChild
  } else {
    return content.value.firstChild || content.value
  }
})

function test () {
  // moveToStart()
  // console.log(content.value)
  // content.value.editor.commands.focus(2)
  // console.log(content.value.editor.commands.setTextSelection)
}

const textContent = computed(() => {
  if (innerContent.value) return innerContent.value.parentElement.textContent
})

const htmlContent = computed(() => {
  if (innerContent.value) return innerContent.value.parentElement.innerHTML
})

function keyDownHandler (event:KeyboardEvent) {
  if (event.key === 'ArrowUp') {
    if (menu.value.open) {
      event.preventDefault()
    }
    // If at first line, move to previous block
    else if (atFirstLine()) {
      event.preventDefault()
      emit('moveToPrevLine')
    }
  } else if (event.key === 'ArrowDown') {
    if (menu.value.open) {
      event.preventDefault()
    }
    // If at last line, move to next block
    else if (atLastLine()) {
      event.preventDefault()
      emit('moveToNextLine')
    }
  } else if (event.key === 'ArrowLeft') {
    // If at first character, move to previous block
    if (atFirstChar()) {
      event.preventDefault()
      emit('moveToPrevChar')
    }
  } else if (event.key === 'ArrowRight') {
    // console.log(getEndCoordinates())
    // console.log(getCaretCoordinates())
    // If at last character, move to next block
    if (atLastChar()) {
      event.preventDefault()
      emit('moveToNextChar')
    }
  } else if (event.key === 'Backspace' && highlightedLength() === 0) {
    if (menu.value && menu.value.open) {
      const selection = window.getSelection()
      if (selection) {
        const offset = selection.anchorOffset
        const deletedChar = content.value.innerText.substring(offset-1, offset)
        if (deletedChar === '/') {
          menu.value.open = false
        }
      }
    } else if (atFirstChar()) {
      event.preventDefault()
      emit('merge')
    }
  } else if (event.key === 'Enter') {
    event.preventDefault()
    console.log(content.value)
    // if (!(menu.value && menu.value.open)) {
    //   emit('split')
    // }
  }
  return
  if (event.key === 'ArrowUp') {
    if (menu.value.open) {
      event.preventDefault()
    }
    // If at first line, move to previous block
    else if (atFirstLine()) {
      event.preventDefault()
      emit('moveToPrevLine')
    }
  } else if (event.key === 'ArrowDown') {
    if (menu.value.open) {
      event.preventDefault()
    }
    // If at last line, move to next block
    else if (atLastLine()) {
      event.preventDefault()
      emit('moveToNextLine')
    }
  } else if (event.key === 'ArrowLeft') {
    // If at first character, move to previous block
    if (atFirstChar()) {
      event.preventDefault()
      emit('moveToPrevChar')
    }
  } else if (event.key === 'ArrowRight') {
    // If at last character, move to next block
    if (atLastChar()) {
      event.preventDefault()
      emit('moveToNextChar')
    }
  } else if (event.key === 'Backspace' && highlightedLength() === 0) {
    if (menu.value && menu.value.open) {
      const selection = window.getSelection()
      if (selection) {
        const offset = selection.anchorOffset
        const deletedChar = content.value.innerText.substring(offset-1, offset)
        if (deletedChar === '/') {
          menu.value.open = false
        }
      }
    } else if (atFirstChar()) {
      event.preventDefault()
      emit('merge')
    }
  } else if (event.key === 'Enter') {
    event.preventDefault()
    if (!(menu.value && menu.value.open)) {
      emit('split')
    }
  }
}

function keyUpHandler (event:Event) {
  // parseMarkdown(event)
}

function isContentBlock () {
  return [BlockType.Text, BlockType.H1, BlockType.H2].includes(props.block.type)
}

const content = ref<HTMLDivElement|null>(null)
const menu = ref<BlockMenu|null>(null)

function atFirstChar () {
  const startCoord = getStartCoordinates()
  const coord = getCaretCoordinates()
  return coord.x === startCoord.x && coord.y === startCoord.y
}

function atLastChar () {
  const endCoord = getEndCoordinates()
  const coord = getCaretCoordinates()
  return coord.x === endCoord.x && coord.y === endCoord.y
}
function atFirstLine () {
  const startCoord = getStartCoordinates()
  const coord = getCaretCoordinates()
  return coord.y === startCoord.y
}

function atLastLine () {
  const endCoord = getEndCoordinates()
  const coord = getCaretCoordinates()
  return coord.y === endCoord.y
}

function highlightedLength () {
  return window.getSelection().toString().length
}

function moveToStart () {
  if (isContentBlock()) {
    console.log(innerContent.value)
    if (innerContent.value) {
      const selection = window.getSelection()
      const range = document.createRange()
      range.selectNodeContents(firstChild.value)
      range.collapse(true)
      selection.removeAllRanges()
      selection.addRange(range)
    } 
  } else {
    emit('moveToNextChar')
  }
}

function moveToEnd () {
  if (isContentBlock()) {
    if (innerContent.value) {
      const selection = window.getSelection()
      const range = document.createRange()
      range.selectNodeContents(lastChild.value)
      range.collapse()
      selection.removeAllRanges()
      selection.addRange(range)
    } 
  } else {
    emit('moveToPrevChar')
  }
}

async function moveToFirstLine () {
  if (isContentBlock()) {
    if (!innerContent.value) {
      moveToStart()
    } else {
      let prevCoord = getCaretCoordinates()
      console.log(prevCoord)
      let prevDist = 99999
      let caretPos = 0
      while (true) {
        setCaretPos(caretPos)
        let newCoord
        await new Promise(resolve => setTimeout(() => {
          newCoord = getCaretCoordinates()
          console.log(newCoord)
          resolve()
        }), 0)
        const newDist = Math.abs(newCoord.x - prevCoord.x)
        if (newDist > prevDist) {
          console.log('short')
          if (caretPos > 0) setCaretPos(caretPos - 1)
          break
        } else if (caretPos === innerContent.value.length || caretPos > 10) {
          console.log('end')
          // Reached end of line
          break
        } else {
          prevDist = newDist
          caretPos += 1
        }
      }
    } 
  } else {
    emit('moveToNextLine')
  }
}

function moveToLastLine () {
  if (isContentBlock()) {
    if (!innerContent.value) {
      moveToStart()
    } else {
      let prevCoord = getCaretCoordinates()
      let prevDist = 99999
      let caretPos = innerContent.value ? innerContent.value.length : 0
      while (true) {
        setCaretPos(caretPos)
        const newCoord = getCaretCoordinates()
        const newDist = Math.abs(newCoord.x - prevCoord.x)
        if (newDist > prevDist) {
          if (caretPos < innerContent.value.length) setCaretPos(caretPos + 1)
          break
        } else if (caretPos === 0) {
          // Reached start of line
          break
        } else {
          prevDist = newDist
          caretPos -= 1
        }
      }
    }
  } else {
    emit('moveToPrevLine')
  }
}

function getCaretCoordinates () {
  let x = 0, y = 0
  const selection = window.getSelection()
  if (selection.rangeCount > 0) {
    const range = selection.getRangeAt(0)
    const rect = range.getBoundingClientRect()
    return rect
  }
  return { x, y }
}

function getCaretPos () {
  if (innerContent.value) {
    const selection = window.getSelection()
    return selection.anchorOffset
  } else {
    return 0
  }
}

function setCaretPos (caretPos:number) {
  if (innerContent.value) {
    if (props.block.type === BlockType.Text) {
      content.value.editor.commands.focus(caretPos)
    } else {
      const selection = window.getSelection()
      const range = document.createRange()
      range.setStart(innerContent.value, caretPos)
      range.setEnd(innerContent.value, caretPos)
      selection.removeAllRanges()
      selection.addRange(range)
    }
  }
}

function getStartCoordinates () {
  let x = 0, y = 0
  if (firstChild.value) {
    const range = document.createRange()
    range.selectNodeContents(firstChild.value)
    range.collapse(true)
    const rect = range.getBoundingClientRect()
    x = rect.left
    y = rect.top
  }
  return { x, y }
}

function getEndCoordinates () {
  let x = 0, y = 0
  if (lastChild.value) {
    const range = document.createRange()
    range.selectNodeContents(lastChild.value)
    range.collapse()
    const rect = range.getBoundingClientRect()
    x = rect.left
    y = rect.top
  }
  return { x, y }
}

// function parseMarkdown (event:Event) {
//   if (content.value) {
//     if (content.value.innerText.match(/^#\s$/) && event.key === ' ') {
//       content.value.innerText = ''
//       emit('setBlockType', BlockType.H1)
//     } else if (content.value.innerText.match(/^##\s$/) && event.key === ' ') {
//       content.value.innerText = ''
//       emit('setBlockType', BlockType.H2)
//     } else if (content.value.innerText.match(/^---$/)) {
//       content.value.innerText = ''
//       emit('setBlockType', BlockType.Divider)
//     } else if (event.key === '/') {
//       if (menu.value) menu.value.open = true
//     }
//   }
// }

function clearSearch (startIdx: number, endIdx: number) {
  // content.value.innerText = content.value.innerText.substring(0, startIdx) + content.value.innerText.substring(endIdx)
  // setCaretPos(startIdx)
}

defineExpose({
  content,
  textContent,
  htmlContent,
  moveToStart,
  moveToEnd,
  moveToFirstLine,
  moveToLastLine,
  getCaretPos,
  setCaretPos,
})
</script>
