<script setup lang="ts">
import { ref } from 'vue'

const resultMsg = ref('')

const { available } = await (window as any).ai.languageModel.capabilities()

if (available === 'readily') {
  const session = await (window as any).ai.languageModel.create()

  const result = await session.prompt('Write me a poem')
  // console.log('result', result)
  resultMsg.value = result
} else {
  resultMsg.value = `Your browser doesn't support the Prompt API. 
  If you're on Chrome, join the Early Preview Program to enable it.
  `
}
</script>

<template>
  <div>
    <p>{{ resultMsg }}</p>
  </div>
</template>

<style scoped></style>
