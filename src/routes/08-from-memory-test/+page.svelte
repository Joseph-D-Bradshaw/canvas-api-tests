<script lang="ts">
  import { onMount } from "svelte";

  const SPEED = 200.0;

  let canvas: HTMLCanvasElement;
  let ctx: CanvasRenderingContext2D;
  let bounds: DOMRect;
  let startTime: number;
  let lastFrameTime: number;

  const circle = {
    x: 10,
    y: 10,
    radius: 10,
  };

  const mouse = {
    x: 10,
    y: 10,
  };

  function updateMouse(
    e: MouseEvent & {
      currentTarget: EventTarget & HTMLCanvasElement;
    },
  ) {
    mouse.x = e.clientX - bounds.left;
    mouse.y = e.clientY - bounds.top;
  }

  function draw(ctx: CanvasRenderingContext2D, timestamp: number) {
    if (startTime === undefined) {
      startTime = timestamp;
      lastFrameTime = timestamp;
    }
    let delta = (timestamp - lastFrameTime) / 1000;
    lastFrameTime = timestamp;

    let dx = mouse.x - circle.x;
    let dy = mouse.y - circle.y;
    let distance = Math.sqrt(dx ** 2 + dy ** 2);
    if (distance > 0) {
      let direction = { x: dx / distance, y: dy / distance };

      circle.x += direction.x * SPEED * delta;
      circle.y += direction.y * SPEED * delta;
    }

    ctx.clearRect(0, 0, canvas.width, canvas.height);

    ctx.beginPath();
    ctx.fillStyle = "black";
    ctx.arc(circle.x, circle.y, circle.radius, 0, Math.PI * 2);
    ctx.fill();
    ctx.closePath();
    requestAnimationFrame((timestamp) => draw(ctx, timestamp));
  }

  onMount(() => {
    ctx = canvas.getContext("2d") as CanvasRenderingContext2D;
    bounds = canvas.getBoundingClientRect();
    requestAnimationFrame((ts) => draw(ctx, ts));
  });
</script>

A quick test from memory to ensure I remember and understand the principles of
basic vector math on the HTML canvas.

<canvas
  bind:this={canvas}
  width="800"
  height="800"
  style="border: 1px solid black;"
  onmousemove={updateMouse}
></canvas>
