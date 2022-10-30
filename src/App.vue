<script setup lang="ts">
import {computed, onMounted, ref} from 'vue'

const el = ref<HTMLCanvasElement>()
const ctx = computed(()=>el.value!.getContext('2d')!)
const WIDTH = 600
const HEIGHT = 600
type Point = {
  x: number,
  y: number
}
type line  = {
  start: Point,
  length: number,
  theta: number
}
onMounted(()=>{
  init()
  const startLine:line = {
    start: {x:WIDTH/2,y:HEIGHT},
    length: 10,
    theta: -Math.PI /4
  }
  step(startLine)
})
const pendingTask:Function[] = []
function step(l:line, depth = 0) {
  const end = getEndPoint(l)
  drawLine(l)
  // 向左递归划线
  if( depth <4 || Math.random()<0.5){
    pendingTask.push(()=>{
      step({
        start: end,
        length: l.length +(Math.random() * 2 - 1),
        theta: l.theta-0.2 * Math.random()
      },depth+1)
    })
  }
  // 向右递归划线
  if(depth < 4 || Math.random()>0.5){
    pendingTask.push(()=>{
      step({
        start: end,
        length: l.length +(Math.random() * 2 - 1),
        theta: l.theta+0.2 * Math.random()
      },depth+1)
    })
  }
}
function frame() {
  const task = [...pendingTask]
  pendingTask.length =  0
  task.forEach(fn=>fn())
}
// requestAnimationFrame(()=>{
//   frame()
//   requestAnimationFrame(()=>{
//     frame()
//   })
// })
let frameCount = 0
function startFrame() {
  requestAnimationFrame(()=>{
    frameCount+=1
    if(frameCount%3 === 0) frame()
    startFrame()
  })
}
startFrame()
function lineTo (p1:Point,p2:Point) {
  ctx.value.beginPath()
  ctx.value.moveTo(p1.x,p1.y)
  ctx.value.lineTo(p2.x,p2.y)
  ctx.value.stroke()
}
function drawLine(l:line) {
  lineTo(l.start, getEndPoint(l))
}
function getEndPoint(l:line) {
  return {
    x: l.start.x + l.length* Math.cos(l.theta),
    y: l.start.y + l.length* Math.sin(l.theta)
  }
}
function init () {
  ctx.value.strokeStyle='#000'
}
</script>

<template>
  <canvas ref="el" width="600" height="600" style="border: 1px solid #000"></canvas>
</template>

<style scoped>
</style>
