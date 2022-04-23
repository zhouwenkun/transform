<script setup>
import { ref, onMounted } from 'vue'
import { generate } from 'escodegen'
import { parseScript, Syntax } from 'esprima'

const options = { comment: true }
const source = ref('$().singlegrid("hello babel.") //test \n /*teststst*/')
const target = ref('')

const transform = () => {
    let ast = parseScript(source.value, options, (node, metadata) => {
        if(node.type === Syntax.CallExpression) {
            let callee = node.callee
            if(callee.type === Syntax.MemberExpression) {
                let property = callee.property
                if(property.type === Syntax.Identifier) {
                    if(property.name === 'singlegrid') {
                        console.log('================>')
                        property.name = 'datagrid'
                    }
                }
            }
        }
        console.log(node)
    })

    target.value = generate(ast, {
        format: {
            semicolons: false
        },
        comment: true
    })
}

onMounted(transform)

</script>

<template>
    <div class="container">
        <div><textarea v-model="source"></textarea></div>
        <div><button @click="transform">transform</button></div>
        <div><textarea v-model="target"></textarea></div>
    </div>
</template>

<style scoped>
    .container {
        display: flex;
        height: 480px;
    }

    .container > div {
        padding: 10px;
    }

    .container > div:first-child,
    .container > div:last-child {
        flex: 1;
    }

    .container > div:nth-child(2) {
        width: 100px;
    }

    textarea {
        width: 100%;
        height: 100%;
    }
</style>