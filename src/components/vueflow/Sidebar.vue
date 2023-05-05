<script setup>
import { watch, toRef, ref } from 'vue'

import ToolbarNode from './ToolbarNode.vue'
import { Position, VueFlow } from '@vue-flow/core'


const roles = [
    {
        id: 1,
        label: 'R1'
    },
    {
        id: 2,
        label: 'R2'
    },
    {
        id: 3,
        label: 'R3'
    },
    {
        id: 4,
        label: 'R4'
    },
    {
        id: 5,
        label: 'R5'
    },
    {
        id: 6,
        label: 'R6'
    }
]

function onDragStart(event, nodeType, label, id) {
    if (event.dataTransfer) {
        event.dataTransfer.setData('application/vueflow', nodeType)
        event.dataTransfer.setData('label', label)
        event.dataTransfer.setData('id', id)
        event.dataTransfer.effectAllowed = 'move'
    }
}

const props = defineProps(['draggedNodes'])

const passedData = toRef(props, 'draggedNodes')

const filteredArray1 = ref()

watch(passedData, (newValue, oldValue) => {
    // console.log(JSON.stringify(newValue))
    filteredArray1.value = roles.filter(item1 => {
        return !newValue.some(item2 => item2.id.toString() === item1.id.toString())
    })

    // console.log(JSON.stringify(filteredArray1.value));
})

</script>

<template>
    <aside>
        <div class="description">You can drag these nodes to the panel.</div>
        <div v-for="role in filteredArray1 ? filteredArray1 : roles " class="nodes">
            <div class="vue-flow__node-default" :draggable="true"
                @dragstart="onDragStart($event, 'toolbar', role.label, role.id)">{{ role.label }}</div>
            <!-- <div class="vue-flow__node-input" :draggable="true" @dragstart="onDragStart($event, 'input', 'Starting')">
                Starting Node
            </div>

            <div class="vue-flow__node-default" :draggable="true" @dragstart="onDragStart($event, 'default', 'sample 1')">
                Sample 1</div>

            <div class="vue-flow__node-output" :draggable="true" @dragstart="onDragStart($event, 'output', 'final')">Final
                Node
            </div> -->
        </div>
    </aside>
</template>
