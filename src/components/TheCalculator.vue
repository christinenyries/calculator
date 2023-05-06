<script setup lang="ts">
import { ref, computed } from 'vue'
const display = ref('') // what the user sees

// the `display` as number
const current = computed({
  get() {
    if (display.value === '') {
      return 0
    }
    return parseFloat(display.value)
  },
  set(newValue: number) {
    display.value = newValue.toString()
  }
})
const previous = ref(0)

const operations = {
  '+/-': (a: number) => -a,
  '%': (a: number) => a / 100,
  '-': (a: number, b: number) => a - b,
  '+': (a: number, b: number) => a + b,
  x: (a: number, b: number) => a * b,
  '/': (a: number, b: number) => a / b
}
const pendingOperation = ref<((...n: number[]) => number) | null>(null)
const areOperandsMissing = ref<boolean>(false)

const clear = () => {
  display.value = ''
  pendingOperation.value = null
}

// appends to `display`
const append = (n: number | '.') => {
  if (areOperandsMissing.value) {
    display.value = ''
    areOperandsMissing.value = false
  }
  if (n === '.' && display.value.includes('.')) return
  display.value += `${n}`
}

const submit = (operation: '/' | 'x' | '-' | '+' | '+/-' | '%') => {
  switch (operation) {
    case '+/-':
      current.value = operations['+/-'](current.value)
      break
    case '%':
      current.value = operations['%'](current.value)
      break
    default:
      evaluatePending()

      // then add new pending operation
      pendingOperation.value = operations[operation]
      areOperandsMissing.value = true
      previous.value = current.value
      break
  }
}
const evaluatePending = () => {
  if (pendingOperation.value) {
    current.value = pendingOperation.value(previous.value, current.value)
    pendingOperation.value = null
  }
}
</script>

<template>
  <div class="calculator">
    <div class="display">{{ display }}</div>

    <div @click="clear">C</div>
    <div @click="submit('+/-')">+/-</div>
    <div @click="submit('%')">%</div>
    <div @click="submit('/')" class="operator">/</div>

    <div @click="append(7)">7</div>
    <div @click="append(8)">8</div>
    <div @click="append(9)">9</div>
    <div @click="submit('x')" class="operator">x</div>

    <div @click="append(4)">4</div>
    <div @click="append(5)">5</div>
    <div @click="append(6)">6</div>
    <div @click="submit('-')" class="operator">-</div>

    <div @click="append(1)">1</div>
    <div @click="append(2)">2</div>
    <div @click="append(3)">3</div>
    <div @click="submit('+')" class="operator">+</div>

    <div class="zero" @click="append(0)">0</div>
    <div @click="append('.')">.</div>
    <div @click="evaluatePending" class="operator">=</div>
  </div>
</template>

<style scoped>
.calculator {
  @apply grid w-96 grid-cols-4 place-items-stretch gap-2 text-center font-mono text-xl;
  grid-auto-rows: minmax(5rem, auto);
}

.calculator > * {
  @apply rounded-lg border border-slate-300 bg-base-300 pt-7;
}

.display {
  @apply col-span-4 bg-neutral pr-7 text-right text-neutral-content;
}

.zero {
  @apply col-span-2;
}

.operator {
  @apply bg-orange-400 font-bold text-white;
}
</style>
