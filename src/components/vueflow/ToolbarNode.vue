<script setup>
import { Handle, Position } from '@vue-flow/core'
import { NodeToolbar } from '@vue-flow/node-toolbar'

const props = defineProps(['data', 'label', 'id'])

const emit = defineEmits(['delete'])

const defaultNodeStyle = {
    border: '1px solid #000',
    background: '#fff',
    color: 'black',
    borderRadius: '5px',
    padding: '8px 40px'
}


const onDelete = () => {
    emit('delete')
}
</script>

<template>
    <NodeToolbar style="display: flex; gap: 0.5rem; align-items: center" :is-visible="data.toolbarVisible"
        :position="Position.Right">
        <button @click="onDelete">Delete</button>
    </NodeToolbar>

    <div :style="defaultNodeStyle">
        {{ label }}
    </div>

    <Handle type="target" :position="Position.Top" />
    <Handle type="source" :position="Position.Bottom" />
</template>
