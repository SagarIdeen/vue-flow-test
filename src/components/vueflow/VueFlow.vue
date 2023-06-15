<script setup>
import { nextTick, watch } from 'vue'
import Sidebar from './Sidebar.vue'
import { Panel, PanelPosition, Position, VueFlow, useVueFlow } from '@vue-flow/core'
import ToolbarNode from './ToolbarNode.vue'
import { ref } from 'vue'

const defaultNodeStyle = {
    border: '1px solid #rrr',
    background: '#fff',
    color: 'black',
    borderRadius: '5px',
    padding: '8px 40px'

}



const { findNode, onConnect, addEdges, addNodes, project, vueFlowRef, getEdges, getNodes, edges, toObject, getElements, getSelectedNodes, removeNodes } = useVueFlow({
    nodes: [
        {
            id: 0,
            type: 'input',
            label: 'intiater',
            position: { x: 250, y: 25 },
            style: defaultNodeStyle
        },
    ],
})

function onDragOver(event) {
    event.preventDefault()

    if (event.dataTransfer) {
        event.dataTransfer.dropEffect = 'move'
    }
}

onConnect((params) => addEdges([params]))

function onDrop(event) {
    const type = event.dataTransfer?.getData('application/vueflow')
    const label = event.dataTransfer?.getData('label')
    const id = event.dataTransfer?.getData('id')

    const { left, top } = vueFlowRef.value.getBoundingClientRect()

    const position = project({
        x: event.clientX - left,
        y: event.clientY - top,
    })

    const newNode = {
        id: id,
        type,
        position,
        label: `${label} node`,
    }

    addNodes([newNode])

    // align node position after drop, so it's centered to the mouse
    nextTick(() => {
        const node = findNode(newNode.id)
        const stop = watch(
            () => node.dimensions,
            (dimensions) => {
                if (dimensions.width > 0 && dimensions.height > 0) {
                    node.position = { x: node.position.x - node.dimensions.width / 2, y: node.position.y - node.dimensions.height / 2 }
                    stop()
                }
            },
            { deep: true, flush: 'post' },
        )
    })
}



const onSave = async () => {
    const value = getEdges.value;
    // GETTING THE EDGES OF NODES
    const getEdgesdata = Promise.all(value.map((item, index) => {
        return {
            id: index,
            label: item.sourceNode.label,
            source_id: item.source,
            target_id: item.target
        }
    }))


    const newValue = await getEdgesdata;
    const finalData = [];
    //setting the initial level 
    const intiaterObjects = newValue.filter(obj => obj.label === "intiater");
    const targetsOfIntiater = intiaterObjects.map(obj => ({ id: obj.target_id }));

    if (intiaterObjects && newValue.length > 0) {

        finalData.push({ level: 1, ids: [{ id: intiaterObjects[0]?.source_id }] })
        finalData.push({ level: 2, ids: targetsOfIntiater })

    }


    for (let i = 0; i < newValue.length; i++) {



        const selectedObjects = newValue.filter(obj => finalData[finalData.length - 1].ids.some(idObj => idObj.id === obj.source_id.toString()));
        // console.log('mp:', JSON.stringify(selectedObjects));
        if (selectedObjects.length > 0) {
            const sameTargetId = checkSameTargetId(selectedObjects);
            // console.log('checking same id', sameTargetId);
            if (sameTargetId) {
                finalData.push({ level: finalData[finalData.length - 1].level + 1, ids: [{ id: sameTargetId }] })
            } else {
                const targetsOfIntiater = selectedObjects.map(obj => ({ id: obj.target_id }));
                finalData.push({ level: finalData[finalData.length - 1].level + 1, ids: targetsOfIntiater })
            }
        }
        i++
    }

    console.log('output', JSON.stringify(finalData));

}


function checkSameTargetId(objects) {
    const targetId = objects[0].target_id;

    if (objects.every(obj => obj.target_id === targetId)) {
        return targetId;
    }

    return null;
}



const onDelete = () => {
    removeNodes(getSelectedNodes.value)
}
const onReset = () => {

    let f = getNodes.value.filter(item1 => {
        return item1.id.toString() != "0"
    })
    removeNodes(f)

}

</script>


<template>
    <div class="dndflow" @drop="onDrop">
        <VueFlow @dragover="onDragOver" :default-viewport="{ zoom: 1.5 }">
            <template #node-toolbar="nodeProps">

                <ToolbarNode :data="nodeProps.data" :label="nodeProps.label" :id="nodeProps.id" v-on:delete="onDelete" />
            </template>
            <Panel :position="PanelPosition.BottomRight">
                <div>
                    <button @click="onSave">Save</button>
                    <button @click="onReset">Reset</button>
                </div>
            </Panel>
        </VueFlow>

        <Sidebar :dragged-nodes="getNodes" />
    </div>
</template>
