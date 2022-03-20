<script setup lang="ts">
import { $ref, $computed } from 'vue/macros';
import { onMounted } from 'vue';

interface Point {
  x: number;
  y: number;
}
interface Branch {
  start: Point;
  length: number;
  angel: number;
}
const el = $ref<HTMLCanvasElement>();
const WIDTH = 600;
const HEIGHT = 600;
const DEPTH = 3;
const ctx = $computed(() => el?.getContext('2d'));
const pendingFrame = [];
function init() {
  // lineTo({ x: 30, y: 50 }, { x: 150, y: 100 });
  const startBranch: Branch = {
    start: { x: WIDTH / 2, y: HEIGHT },
    length: 40,
    angel: -Math.PI / 2,
  };
  step(startBranch);
}

function step(b: Branch, depth: number = 0) {
  const end = getEndPoint(b);
  drawBranch(b);
  if (depth < DEPTH || Math.random() < 0.5) {
    pendingFrame.push(() =>
      step(
        {
          start: end,
          length: b.length + (Math.random() * 10 - 5),
          angel: b.angel - 0.3 * Math.random(),
        },
        depth + 1
      )
    );
  }
  if (depth < DEPTH || Math.random() < 0.5) {
    pendingFrame.push(() =>
      step(
        {
          start: end,
          length: b.length + (Math.random() * 10 - 5),
          angel: b.angel + 0.3 * Math.random(),
        },
        depth + 1
      )
    );
  }
}

function drawFrame() {
  const frames = [...pendingFrame];
  pendingFrame.length = 0;
  frames.forEach((frame) => frame());
}

function startFrame() {
  requestAnimationFrame(() => {
    drawFrame();
    startFrame();
  });
}

startFrame();
function drawBranch(b: Branch) {
  const endPoint = getEndPoint(b);
  lineTo(b.start, endPoint);
}

function getEndPoint(b: Branch) {
  return {
    x: b.start.x + b.length * Math.cos(b.angel),
    y: b.start.y + b.length * Math.sin(b.angel),
  };
}

function lineTo(p1: Point, p2: Point) {
  ctx.beginPath();
  ctx.moveTo(p1.x, p1.y);
  ctx.lineTo(p2.x, p2.y);
  ctx.stroke();
}

onMounted(() => {
  init();
});
</script>

<template>
  <canvas ref="el" width="600" height="600" style="border: 1px solid grey"></canvas>
</template>

<style></style>
