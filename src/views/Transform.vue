<script setup>
import { ref, onMounted, watch, getCurrentInstance } from 'vue'
import { generate } from 'escodegen'
import { parseScript, Syntax } from 'esprima'
import { fromTextArea } from 'codemirror'

import 'codemirror/lib/codemirror.css'
import 'codemirror/mode/javascript/javascript.js'

let a, b
const options = { comment: true }

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
        comment: true
    })
}

const click = () => {
    b.setValue(transform(a.getValue()))
}

onMounted(() => {
    console.log('onMounted...')

    let options = {
        lineNumbers: true,
        mode: 'javascript'
    }
    let textareas = $('div.container textarea')

    console.log(textareas, textareas[0].value, textareas[1].value)

    console.log(getCurrentInstance())

    a = fromTextArea(textareas[0], options)
    b = fromTextArea(textareas[1], options)
})
</script>

<template>
    <div class="container">
        <div><textarea></textarea></div>
        <div><button @click="click">transform</button></div>
        <div><textarea></textarea></div>
    </div>
</template>

<style>
    .CodeMirror {
        width: 100%;
        height: 100%;
        border: 1px solid lightgray;
    }
</style>

<style scoped>
    .container {
        display: flex;
        height: 480px;
    }

    .container > div {
        padding: 10px;
        text-align: left;
        overflow: hidden;
    }

    .container > div:first-child,
    .container > div:last-child {
        flex: 1;
    }

    .container > div:nth-child(2) {
        width: 100px;
        text-align: center;
    }

    textarea {
        width: 100%;
        height: 100%;
    }
</style>