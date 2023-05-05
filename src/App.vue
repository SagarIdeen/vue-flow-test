<script setup>
import { nextTick, watch } from 'vue'
import Sidebar from './components/vueflow/Sidebar.vue'
import { Panel, PanelPosition, Position, VueFlow, useVueFlow } from '@vue-flow/core'
import ToolbarNode from './components/vueflow/ToolbarNode.vue'
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

  const newArray = [];
  // MODIFYING THE EDGES ARRAY
  newValue.map((item, index) => {
    const existingObject = newArray.find(obj => obj.source_id === item.source_id);
    if (existingObject && newArray[newArray.length - 1].source_id === item.source_id) {
      newArray[newArray.length - 1].targets.push({ target_id: item.target_id })
    } else {
      newArray.push({
        label: item.label,
        source_id: item.source_id,
        targets: [{ target_id: item.target_id }]
      })
    }
  })
  // console.log(JSON.stringify(newArray));



  const sample = []
  // MODIFYING THE ARRAY BASED ON THE LEVEL
  newArray.map((item, index) => {

    if (index === 0) {
      sample.push(
        {
          level: 1,
          data: [{ id: item.source_id }]
        }
      )

      sample.push({
        level: 2,
        data: getD(item.targets)
      })
    } else {
      const targetExists = sample.some(obj => obj.data.some(t => t.id === item.source_id));
      if (targetExists) {
        item.targets.map((i) => {
          const targetExists2 = sample.some(obj => obj.data.some(t => t.id === i.target_id));
          targetExists2 ? null : sample.push({
            level: sample.length + 1,
            data: getD(item.targets)
            // data: [{ id: i.target_id }]
          })
        })
      }
    }
  })

  console.log('sample', JSON.stringify(sample));
}
const getD = (d) => {
  return d.map((i) => {
    return { id: i.target_id }
  })
}

const onDelete = () => {
  removeNodes(getSelectedNodes.value)
}

</script>


<template>
  <div class="dndflow" @drop="onDrop">
    <VueFlow fit-view-on-init class="vue-flow-basic-example" @dragover="onDragOver">

      <template #node-toolbar="nodeProps">
        <ToolbarNode :data="nodeProps.data" :label="nodeProps.label" :id="nodeProps.id" v-on:delete="onDelete" />
      </template>
      <Panel :position="PanelPosition.BottomRight">
        <div>
          <button @click="onSave">Save</button>
        </div>
      </Panel>
    </VueFlow>
    <Sidebar :dragged-nodes="getNodes" />
  </div>
</template>

