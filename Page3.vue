<template>
  <div class="wrapper">
    <canvas ref="canvas"></canvas>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, ref } from 'vue';
const canvas = ref<HTMLCanvasElement>();
const color = 'rgb(200, 200, 200)';
const random = (min: number, max: number) => {
  return Math.floor(Math.random() * (max + 1 - min)) + min;
};
const width = ref<number>(window.innerWidth);
const height = ref<number>(window.innerHeight);
const radio = ref<number>(3);
const xr = computed(() => [radio.value / 2, width.value - radio.value / 2]);
const yr = computed(() => [radio.value / 2, height.value - radio.value / 2]);

class Point {
  private _x: number;
  private _y: number;
  private _xs: number;
  private _ys: number;
  private _lastDrawTime: number;
  constructor(private ctx: CanvasRenderingContext2D) {
    this._x = random(xr.value[0], xr.value[1]);
    this._y = random(yr.value[0], yr.value[1]);
    this._xs = random(-50, 50);
    this._ys = random(-50, 50);
    this._lastDrawTime = -1;
  }

  private calcXY() {
    const duration = Math.min((Date.now() - this._lastDrawTime) / 1000, 0.1);
    const xDis = this._xs * duration;
    const yDis = this._ys * duration;
    this._x = this._x + xDis;
    this._y = this._y + yDis;
  }

  public draw() {
    if (this._lastDrawTime != -1) {
      this.calcXY();
      if (this._x < xr.value[0]) {
        this._xs = 0 - this._xs;
        this._x = xr.value[0];
      } else if (this._x > xr.value[1]) {
        this._xs = 0 - this._xs;
        this._x = xr.value[1];
      }
      if (this._y < yr.value[0]) {
        this._ys = 0 - this._ys;
        this._y = yr.value[0];
      }
      if (this._y > yr.value[1]) {
        this._ys = 0 - this._ys;
        this._y = yr.value[1];
      }
    }
    this.ctx.beginPath();
    this.ctx.arc(this._x, this._y, radio.value, 0, Math.PI * 2);
    this.ctx.fillStyle = color;
    this.ctx.fill();
    this._lastDrawTime = Date.now();
  }
  get x(): number {
    return this._x;
  }
  get y(): number {
    return this._y;
  }
  get xs(): number {
    return this._xs;
  }
  get ys(): number {
    return this._ys;
  }
}

class Graph {
  private points: Point[];
  constructor(
    private ctx: CanvasRenderingContext2D,
    pointNumber = 60,
    private maxDis = 300,
  ) {
    this.points = Array(pointNumber)
      .fill(0)
      .map(() => new Point(ctx));
  }

  private draw() {
    this.ctx.clearRect(0, 0, width.value, height.value);
    for (let i = 0; i < this.points.length; i++) {
      const p1 = this.points[i];
      p1.draw();
      for (let j = i + 1; j < this.points.length; j++) {
        const p2 = this.points[j];
        const d = Math.sqrt((p1.x - p2.x) ** 2 + (p1.y - p2.y) ** 2);
        if (d > this.maxDis) {
          continue;
        }
        this.ctx.beginPath();
        this.ctx.moveTo(p1.x, p1.y);
        this.ctx.lineTo(p2.x, p2.y);
        this.ctx.strokeStyle = `rgb(200, 200, 200, ${1 - d / this.maxDis})`;
        this.ctx.stroke();
      }
    }
    requestAnimationFrame(() => this.draw());
  }

  public start() {
    requestAnimationFrame(() => this.draw());
  }
}

onMounted(() => {
  if (!canvas.value) {
    return;
  }
  canvas.value.width = width.value;
  canvas.value.height = height.value;
  const ctx = canvas.value!.getContext('2d') as CanvasRenderingContext2D;

  const graph = new Graph(ctx, 40);
  graph.start();
});
</script>

<style scoped lang="css">
.wrapper {
  background-color: #000000;
}
</style>
