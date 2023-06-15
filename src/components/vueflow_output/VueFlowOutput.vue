<script setup>
import { Background } from '@vue-flow/background'
import { Panel, PanelPosition, VueFlow, useVueFlow } from '@vue-flow/core'
import { ref, onMounted, reactive } from 'vue'




const gettingArray = [
    { "level": 1, "ids": [{ "id": "0" }] },
    { "level": 2, "ids": [{ "id": "1" }] },
    { "level": 3, "ids": [{ "id": "3" }, { "id": "2" }, { "id": "12" }] },
    { "level": 4, "ids": [{ "id": "6" }] },
    { "level": 5, "ids": [{ "id": "4" }] }
]

const { onPaneReady } = useVueFlow()


const nodeData = ref([]);
const edgesData = ref([])

const elements = ref(null)




onMounted(() => {

    console.log('mounted');

    gettingArray.forEach((obj, i) => {
        obj.ids.forEach((item, j) => {
            nodeData.value.push(
                {
                    id: item.id,
                    label: `${item.id} node`,
                    position: { x: 250 + (j * 200), y: 50 + (70 * i) }
                }
            )

            if (i > 0) {

                gettingArray[i - 1].ids.forEach(k => {
                    edgesData.value.push({
                        id: `e-${i}-${j}`,
                        source: k.id,
                        target: item.id
                    })
                })
            }
        })
    })

    console.log('nodes :', JSON.stringify(nodeData.value));
    console.log('edges :', JSON.stringify(edgesData.value));
    console.log('combined :', JSON.stringify(nodeData.value.concat(edgesData.value)));

    elements.value = nodeData.value.concat(edgesData.value)
})




</script>

<template>
    <VueFlow v-model="elements" :default-viewport="{ zoom: 1.5 }">

        <Background />
    </VueFlow>
</template>
