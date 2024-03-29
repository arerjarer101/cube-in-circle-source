<script setup>
import { ref } from 'vue'

const circle = ref({
  left: 150,
  top: 150,
})
const rect = ref({
  height: 150,
  width: 150,
  centerX: 50,
  centerY: 50,
})
const rectVertexOffsets = ref([
  [0,0], 
  [rect.value.width-5, 0], 
  [rect.value.width-5, rect.value.height-5], 
  [0, rect.value.height-5]
])

const isDragging = ref(false)
const circleRadius = ref(200)
const mouseX = ref()
const mouseY = ref()
const circleLeft = ref(circle.value.left + 'px')
const circleTop = ref(circle.value.top + 'px')
const rectLeft = ref((circle.value.left + circleRadius.value - rect.value.width/2) + 'px')
const rectTop = ref((circle.value.top + circleRadius.value - rect.value.height/2) + 'px')
const layerOffsetX = ref(rect.value.width/2)
const layerOffsetY = ref(rect.value.width/2)
let newX = null
let newY = null
const previousValues = ref({
  width: rect.value.width,
  height: rect.value.height,
  radius: circleRadius.value
})

document.onmousemove = (e) => {
  mouseX.value = e.pageX
  mouseY.value = e.pageY
  if(isDragging.value) {
    newX = (mouseX.value - layerOffsetX.value) 
    newY = (mouseY.value - layerOffsetY.value) 

    if(!isInsideCircle(newX, newY)) return

    rectLeft.value = newX + 'px'
    rectTop.value = newY + 'px'
  }
}

function isInsideCircle(x, y) {
  let ans = true
  rectVertexOffsets.value.forEach(e => {
    if (
      (
        (x+e[0] - (circleRadius.value+circle.value.left))*(x+e[0] - (circleRadius.value+circle.value.left)) + 
        (y+e[1] - (circleRadius.value+circle.value.top))*(y+e[1] - (circleRadius.value+circle.value.top))
      ) >= circleRadius.value*circleRadius.value
    ) {
      ans = false
    }
  });

  return ans
}

document.onmouseup = () => {
  isDragging.value = false
}

function startDragging(event) {
 isDragging.value = true
 layerOffsetX.value = event.layerX
 layerOffsetY.value = event.layerY
}

function recalculateOffsets(){
  rectVertexOffsets.value[0] = [0,0]
  rectVertexOffsets.value[1] = [rect.value.width-5, 0]
  rectVertexOffsets.value[2] = [rect.value.width-5, rect.value.height-5]
  rectVertexOffsets.value[3] = [0, rect.value.height-5]
}

function onHeightInput() {
  recalculateOffsets()
  if((Math.pow(rect.value.height, 2) + Math.pow(rect.value.width, 2))/2 > circleRadius.value*circleRadius.value*2) {
    rect.value.height = previousValues.value.height
    return
  }
  previousValues.value.height = rect.value.height
}
function onWidthInput() {
  recalculateOffsets()
  if((Math.pow(rect.value.height, 2) + Math.pow(rect.value.width, 2))/2 > circleRadius.value*circleRadius.value*2) {
    rect.value.width = previousValues.value.width
    return
  }
  previousValues.value.width = rect.value.width
}
function onRadiusInput() {
  recalculateOffsets()
  if((Math.pow(rect.value.height, 2) + Math.pow(rect.value.width, 2))/2 > circleRadius.value*circleRadius.value*2) {
    rect.value.width = previousValues.value.width
    return
  }
  previousValues.value.width = rect.value.width
}
function onCoordXInput() {
  rectLeft.value = (circle.value.left + circleRadius.value - rect.value.width/2) + 'px'
  circleLeft.value = circle.value.left + 'px'
}
function onCoordYInput() {
  rectTop.value = (circle.value.top + circleRadius.value - rect.value.height/2) + 'px'
  circleTop.value = circle.value.top + 'px'
}

</script>

<template>
  <div id="playground" >
    <div id="topbar">
      <div>
        <label for="height_input">Высота:</label>
        <input id="height_input" @input="onHeightInput" v-model="rect.height" type="number">
      </div>
      <div>
        <label for="width_input">Ширина:</label>
        <input id="width_input" @input="onWidthInput" v-model="rect.width" type="number">
      </div>
      <div>
        <label for="radius_input">Радиус окружности:</label>
        <input id="radius_input" @input="onRadiusInput" v-model="circleRadius" type="number">
      </div>
      <div>
        <span>Координаты окружности</span>
      </div>
      <div id="coords">
        <div>
          <label for="coord_inputX">X:</label>
          <input id="coord_inputX" @input="onCoordXInput" v-model="circle.left" type="number">
        </div>
        <div>
          <label for="coord_inputY">Y:</label>
          <input id="coord_inputY" @input="onCoordYInput" v-model="circle.top" type="number">
        </div>
      </div>
    </div>

    <svg @mousedown="startDragging($event)" draggable id="rect" :width="rect.width" :height="rect.height" xmlns="http://www.w3.org/2000/svg">
      <rect :width="rect.width" :height="rect.height"/>
    </svg>
    
    <svg id="circle" :width="circleRadius*2+5" :height="circleRadius*2+5">
      <circle :cx="circleRadius+2.5" :cy="circleRadius+2.5" :r="circleRadius"/>
    </svg>
  </div>
  
</template>

<style scoped>
circle {
  fill:rgb(72, 255, 0);
  stroke:rgb(192, 255, 252);
  stroke-width:5;
  fill-opacity:0.5;
  stroke-opacity:0.9;
}

rect {
  fill:rgb(132, 132, 255);
  stroke:rgb(255, 77, 0);
  stroke-width:5;
  fill-opacity:0.7;
  stroke-opacity:0.9;
}

#circle {
  z-index: 500;
  position: absolute;
  top: v-bind(circleTop);
  left: v-bind(circleLeft);
}

#rect {
  cursor: grab;
  z-index: 1000;
  position: absolute;
  top: v-bind(rectTop);
  left: v-bind(rectLeft);
}

#rect:active {
  cursor: grabbing;
}

#topbar {
  font-size: 1.2rem;
  padding: 20px 5px 10px 10px;
  display: flex;
  flex-direction: row;
  gap: 20px;
}

#topbar input {
  font-size: 1.3rem;
  width: 10rem;
  text-align: center;
}

#coords{
  display: flex;
  flex-direction: column;
  gap: 5px;
}

</style>
