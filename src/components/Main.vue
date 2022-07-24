<template>
  <div class="grid grid-cols-2">
    <div class="h-screen overflow-y-auto">
      <pre class="whitespace-pre-wrap p-10">
      {{ JSON.stringify(blocks, false, 2) }}
      </pre>
    </div>
    <div class="max-w-prose mx-auto my-24">
      <h1 id="title" class="px-4 sm:px-0 focus:outline-none focus-visible:outline-none text-5xl font-bold mb-12" contenteditable="true" spellcheck="false">
        {{ title || '' }}
      </h1>
      <!-- <h1 id="title" class="px-4 sm:px-0 focus:outline-none focus-visible:outline-none text-5xl font-bold mb-12" contenteditable="true" spellcheck="false"
        :class="store.title ? '' : 'empty'"
        @blur="updateTitle($event)"
        @keydown.enter="$event.preventDefault(); appendBlock(-1);"
        @keydown.down="moveToBlocks"
        data-ph="Untitled">
        {{ store.title || '' }}
      </h1> -->
      <draggable tag="div" :list="blocks"
        v-bind="dragOptions" class="-ml-24 space-y-2 pb-4">
        <transition-group type="transition" name="flip-list">
          <div class="list-group-item relative group flex rounded-lg"
            v-for="block, i in blocks" :key="i">
            <Block :block="block"
              :ref="el => blockElements[i] = el"
              @deleteBlock="blocks.splice(i, 1)"
              @newBlock="insertBlock(i)"
              @moveToPrevChar="blockElements[i-1] ? blockElements[i-1].moveToEnd() : null"
              @moveToNextChar="blockElements[i+1] ? blockElements[i+1].moveToStart() : null"
              @moveToPrevLine="blockElements[i-1] ? blockElements[i-1].moveToLastLine() : null"
              @moveToNextLine="blockElements[i+1] ? blockElements[i+1].moveToFirstLine() : null"
              @merge="merge(i)"
              @split="split(i)"
              @setBlockType="type => setBlockType(i, type)"
              />
          </div>
        </transition-group>
      </draggable>
      <div @click="viewBlocks" class="bg-black text-white rounded px-3 py-1 w-max font-bold">Log</div>
    </div>
  </div>
</template>

<style>
#title.empty[contenteditable='true']:empty:before{
  content:attr(data-ph);
  color:#BBBBBB;
}
/* .flip-list-move {
  transition: transform 0.5s;
} */
.no-move {
  transition: transform 0s;
}
.ghost {
  opacity: 1;
  background: #F5F5F5;
}
.list-group {
  min-height: 20px;
}
</style>

<script setup lang="ts">
import { ref, onBeforeUpdate } from 'vue'
import { VueDraggableNext as draggable } from 'vue-draggable-next'
import { BlockType } from '@/utils/types'
import Block from './Block.vue'

const dragOptions = {
  animation: 150,
  group: 'description',
  disabled: false,
  ghostClass: 'ghost',
}

const title = ref('Lotion')
const blocks = ref([{
  type: BlockType.H1,
  details: {
    value: 'Test'
  },
}, {
  type: BlockType.Divider,
  details: {},
}, {
  type: BlockType.Text,
  details: {
    value: '123'
  },
}, {
  type: BlockType.Text,
  details: {
    value: '456'
  },
}, {
  type: BlockType.Text,
  details: {
    value: '"No" is a song by American singer-songwriter Meghan Trainor from her second major label studio album Thank You (2016). Ricky Reed (pictured) produced the song and wrote it with Trainor and Jacob Kasher Hindlin; Epic Records released it as the album\'s lead single on March 4, 2016. A dance-pop song inspired by 1990s music and R&B, "No" has lyrics about sexual consent and women\'s empowerment which encourage them to reject unwanted advances from men.'
  },
}, {
  type: BlockType.Text,
  details: {
    value: '123'
  },
}, ])

onBeforeUpdate(() => {
  blockElements.value = []
})

const blockElements = ref<Block[]>([])

function insertBlock (blockIdx: number) {
  blocks.value.splice(blockIdx + 1, 0, {
    type: BlockType.Text,
    details: {
      value: '',
    },
  })
  setTimeout(() => blockElements.value[blockIdx+1].moveToStart())
}

function setBlockType (blockIdx: number, type: BlockType) {
  blocks.value[blockIdx].type = type
  if (type === BlockType.Divider) insertBlock(blockIdx)
}

function merge (blockIdx: number) {
  if (![BlockType.Text, BlockType.H1, BlockType.H2].includes(blocks.value[blockIdx-1].type)) {
    blocks.value.splice(blockIdx-1, 1)
    setTimeout(() => {
      blockElements.value[blockIdx-1].moveToStart()
    })
    return
  }
  const prevBlockContentLength = blocks.value[blockIdx-1].details.value.length
  blocks.value[blockIdx-1].details.value += blockElements.value[blockIdx].content.innerText
  setTimeout(() => {
    blockElements.value[blockIdx-1].setCaretPos(prevBlockContentLength)
    blocks.value.splice(blockIdx, 1)
  })
}

function split (blockIdx: number) {
  const caretPos = blockElements.value[blockIdx].getCaretPos()
  insertBlock(blockIdx)
  blocks.value[blockIdx+1].details.value = blocks.value[blockIdx].details.value.slice(caretPos)
  blocks.value[blockIdx].details.value = blocks.value[blockIdx].details.value.slice(0, caretPos)
}

function viewBlocks () {
  console.log(blocks.value)
}
</script>