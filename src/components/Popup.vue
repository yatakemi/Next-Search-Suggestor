<script setup lang="ts">
import { ref } from 'vue'

const retryMsg = 'Please try again.'
const resultMsg = ref<string[]>([])

let siteInner = ''

chrome.tabs.query({ currentWindow: true, active: true }).then(
  (tabs) => {
    for (const tab of tabs) {
      chrome.scripting
        .executeScript({
          target: { tabId: tab.id as number },
          func: () => {
            const mainContent = document.querySelector('body')
            if (mainContent === null) {
              return null
            }
            const textContent = mainContent.innerText

            return textContent
          },
        })
        .then((results) => {
          console.log(results[0].result)
          siteInner =
            results[0].result?.replace('  ', '').substring(0, 4000) || ''
          console.log('siteInner', siteInner)
        })
    }
  },
  (error) => {
    console.error(error)
  }
)

const { available } = await (window as any).ai.languageModel.capabilities()

if (available === 'readily') {
  try {
    const session = await (window as any).ai.languageModel.create()

    const result = await session.prompt(
      `Below is the text of the site the user visited. Use this information to suggest what users should search for next.
The answer should only return an array of search strings. Never return anything else.
sample: ["keyword1-1 keyword1-2", "keyword2-1 keyword2-2"]
answer: 
-------
    ${siteInner}`
    )
    console.log('result', result)
    const resultObj = JSON.parse(result) as string[]
    console.log('resultObj', resultObj)
    resultMsg.value = resultObj
  } catch (e) {
    console.log(e)
    resultMsg.value = [retryMsg]
  }
} else {
  resultMsg.value = [
    "Your browser doesn't support the Prompt API. If you're on Chrome, join the Early Preview Program to enable it. https://developer.chrome.com/docs/ai/built-in?hl=ja#get_an_early_preview",
  ]
}

const generateLink = (keyword: string) => {
  return 'https://www.google.com/search?q=' + keyword
}
const openNewPage = (linkUrl: string) => {
  chrome.tabs.create({ url: linkUrl })
}

const reload = () => {
  window.location.reload()
}
</script>

<template>
  <div>
    <h1 class="title">List of what you should search for next</h1>

    <button v-if="retryMsg === resultMsg[0]" @click="reload">
      {{ retryMsg }}
    </button>
    <ul v-else>
      <li v-for="item in resultMsg" :key="item">
        <a
          :href="generateLink(item)"
          @click="openNewPage(generateLink(item))"
          >{{ item }}</a
        >
      </li>
    </ul>
  </div>
</template>

<style scoped>
body {
  font-family: sans-serif;
  margin: 0;
  padding: 0;
}

.title {
  text-align: center;
  margin-bottom: 20px;
  font-size: 24px;
}

ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

li {
  margin-bottom: 10px;
}

a {
  text-decoration: none;
  color: #007bff;
  transition: color 0.3s ease-in-out;
}

a:hover {
  color: #0056b2;
}
</style>
