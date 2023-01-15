<script setup lang="ts">
import {computed, ref} from "vue";

const code = ref('')
const output = computed(() => {
  let pointer = 0
  const memory: number[] = [0]
  const ret: number[] = []
  const loopStartPos: number[] = []
  let skippingLoop = false
  const startTime = Date.now()

  // start
  for (let i = 0; i < code.value.length;) {
    if (skippingLoop) {
      if (code.value[i] === ']') {
        skippingLoop = false
      }
      i++
      continue
    }
    switch (code.value[i]) {
      case '+':
        // add 1 to current pointed value
        memory[pointer]++
        break
      case '-':
        // minus 1 to current pointed value
        memory[pointer]--
        break
      case '>':
        pointer++
        // alloc new memory
        if (memory.length - 1 < pointer) {
          memory.push(0)
        }
        break
      case '<':
        pointer--
        if (pointer < 0) {
          pointer = 0
        }
        break
      case '[':
        if (memory[pointer] != 0) {
          // enter loop
          loopStartPos.push(i)
        } else {
          // skip loop
          skippingLoop = true
        }
        break
      case ']':
        if (memory[pointer] != 0) {
          // back to loop start position
          i = loopStartPos[loopStartPos.length - 1]
        } else {
          loopStartPos.pop()
        }
        break
      case '.':
        ret.push(memory[pointer])
        break
      default:
        return {
          result: `Unexpected symbol: ${code.value[i]}, at: ${i}`,
          memory: [] as number[],
          error: true,
          time: Date.now() - startTime
        }
    }
    i++

    // timeout
    if (Date.now() - startTime > 5000) {
      return {
        result: `Timeout exceeded`,
        memory: [] as number[],
        error: true,
        time: Date.now() - startTime
      }
    }
  }

  return {
    result: new TextDecoder().decode(new Uint8Array(ret)),
    memory,
    error: false,
    time: Date.now() - startTime
  }
})
</script>

<template>
  <div class="flex justify-center">
    <div class="w-1200px">
      <h1>Reactive brainfuck interpreter</h1>
      <div>Input:</div>
      <textarea class="w-full border-black border-1px" v-model="code"/>
      <div>Output: <span :class="output.error ? 'text-[#f00]' : ''">{{ output.result }}</span></div>
      <div>Time: {{ output.time }}ms</div>
      <div v-if="output.memory.length">
        <div>Memory:</div>
        <div v-for="(value, pos) in output.memory">{{ pos }}: {{ value }}</div>
      </div>
    </div>
  </div>
</template>
