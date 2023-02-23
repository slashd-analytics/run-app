<script setup>
import {ref, onMounted} from 'vue'
import Editor from '@guolao/vue-monaco-editor'
import SlashdRun from '@slashd/run'

SlashdRun.setup({restrict:true, deps:['https://unpkg.com/numbro', 'https://unpkg.com/dayjs', 'https://unpkg.com/lodash', 'https://unpkg.com/d3']})

const result = ref('Hey')
const err = ref(false)
const currSnippet = ref(null)
let monacoRef = null


const onChange = async (v) => {
  err.value = false
  try{
    const res = await SlashdRun.exe(v, {})
    result.value = res
  }catch(e){
    err.value = true
    result.value = e
  }
}

const snipIt = () => {
  monacoRef.getModel().setValue(currSnippet.value.code)
  currSnippet.value = null
}

onMounted(() => {
  currSnippet.value = snippets[0]
})

const handleEditorMount = (editor, monaco) => {
  monacoRef = editor
  snipIt()
}

const snippets = [
  {label:'Hello World', code:'return "Hello World"'},
  {label:'Random', code:'return Math.random()'},
  {label:'Numbro', code:'return numbro(1000).format({thousandSeparated:true})'},
  {label:'DayJs', code:'return dayjs().format("DD MM YYYY")'},
  {label:'LoDash', code:'return _.difference([2, 1], [2, 3])'},
  {label:'D3js', code:'return d3.scaleLinear().domain([0,100]).range(["red", "blue"])(Math.random()*100)'},
  {label:'try to get constructor', code:"(_=>_).constructor('return this')()"},
  {label:'try to make fetch', code:"(_=>_).constructor('return this')().fetch('https://jsonplaceholder.typicode.com/todos').then(res => res.json())"}
]


</script>

<template>
  <div class="body">
    <div class="bar">
      <select v-model="currSnippet" @change="snipIt">
          <option value="null" disabled>Select a Snippet</option>
          <option v-for="item in snippets" :value="item" :key="item.label">{{ item.label }}</option>
      </select>
    </div>
    <div class="up">
    <Editor 
      :onMount='handleEditorMount'
      theme='vs-light'
      :options='{lineNumbers:false, showFoldingControls:false, quickSuggestions:false, acceptSuggestionOnCommitCharacter:false, padding:0}'
      defaultLanguage="javascript"
      :defaultValue='defValue'
      :onUpdate:value='onChange'
  />
  </div>
  <div class="down">
    <textarea :class="{err}" v-model="result"></textarea>
  </div>
  </div>
</template>

<style scoped>

.body{
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
}

.bar{
  padding: .5rem;
}
.up{
  padding: 1rem;
  flex:1;
  background-color: #eee;
}
.down{
  flex:1;
  padding: 1rem;
  background-color: #ddd;
}

.down textarea{
  width: 100%;
  height: 100%;
  padding: 1rem;
  border:none;
  
}

textarea.err{
  background-color: rgb(253, 197, 197);
}
</style>
