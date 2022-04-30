<script setup>
import { ref, onMounted, getCurrentInstance, watch } from 'vue'
import { useWindowSize } from '@vueuse/core'

import { generate } from 'escodegen'


import  * as esprima from 'esprima'
import { parseScript, Syntax } from 'esprima'

//import { _ } from 'lodash'

import * as monaco from 'monaco-editor'
import editorWorker from 'monaco-editor/esm/vs/editor/editor.worker?worker'
import jsonWorker from 'monaco-editor/esm/vs/language/json/json.worker?worker'
import cssWorker from 'monaco-editor/esm/vs/language/css/css.worker?worker'
import htmlWorker from 'monaco-editor/esm/vs/language/html/html.worker?worker'
import tsWorker from 'monaco-editor/esm/vs/language/typescript/ts.worker?worker'

self.MonacoEnvironment = {
  getWorker(_, label) {
    if (label === 'json') {
      return new jsonWorker()
    }
    if (label === 'css' || label === 'scss' || label === 'less') {
      return new cssWorker()
    }
    if (label === 'html' || label === 'handlebars' || label === 'razor') {
      return new htmlWorker()
    }
    if (label === 'typescript' || label === 'javascript') {
      return new tsWorker()
    }
    return new editorWorker()
  }
}

const { width } = useWindowSize()

console.log('generate:', generate)
console.log('_:', _)



let a, b
const options = { comment: true }

console.log('esprima:', esprima)

const transform = (source) => {
    let ast = parseScript(source, options, (node, metadata) => {
        if(node.type === Syntax.CallExpression) {
            let callee = node.callee
            if(callee.type === Syntax.MemberExpression) {
                let property = callee.property
                console.log('================>', callee)
                if(property.type === Syntax.Identifier) {
                    if(property.name === 'singledatagrid') {
                        console.log('++++')
                        //property.name = 'datagrid'
                    }
                }
            }
        }
        console.log(node)
    })

    return generate(ast, {
        format: {
            semicolons: false
        },
        comment: true,
        comments: true
    })
}

const click = () => {
    b.setValue(transform(a.getValue()))
}

onMounted(() => {
    console.log('onMounted...')
    
    let options = {
        lineNumbers: true,
        language: 'javascript'
    }

    console.log($, $('div.container > div:first-child, div.container > div:last-child'), getCurrentInstance())

    a = monaco.editor.create($('div.container > div:first-child').get(0), options)
    b = monaco.editor.create($('div.container > div:last-child')[0], options)
})

console.log(width.value)

watch(width, () => {
    console.log('width:', width.value)
    //[a, b].map(e => e.layout())
    //[a, b].forEach(e => e.layout())
    a.layout()
    b.layout()
})
</script>

<template>
    <div class="container">
        <div></div>
        <div><button @click="click">transform</button></div>
        <div></div>
    </div>
</template>

<style scoped>
    .container {
        display: flex;
        height: 100vh;
    }
    .container > div {
        overflow: hidden;
    }
    .container > div:first-child,
    .container > div:last-child {
        flex: 1;
        margin: 8px;
        border: 1px solid lightgray;
    }
    .container > div:nth-child(2) {
        width: 200px;
        text-align: center;
        align-self: center;
    }
</style>