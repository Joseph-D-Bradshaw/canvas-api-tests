<script lang="ts">
  import { onMount } from "svelte";

  let canvas: HTMLCanvasElement;
  let ctx: CanvasRenderingContext2D;
  let bounds: DOMRect;
  let startTime: number;
  let lastFrameTime: number;

  let draggingShape = $state(false);
  let offsetX = $state(0);
  let offsetY = $state(0);

  const circle = {
    x: 60,
    y: 60,
    radius: 30,
  };

  const mouse = {
    x: 10,
    y: 10,
  };

  function onMouseDown(
    _e: MouseEvent & {
      currentTarget: EventTarget & HTMLCanvasElement;
    },
  ) {
    let dx = circle.x - mouse.x;
    let dy = circle.y - mouse.y;
    let distance = Math.sqrt(dx ** 2 + dy ** 2);
    if (distance < circle.radius) {
      draggingShape = true;
      offsetX = circle.x - mouse.x;
      offsetY = circle.y - mouse.y;
    }
  }

  function onMouseUp(
    _e: MouseEvent & {
      currentTarget: EventTarget & HTMLCanvasElement;
    },
  ) {
    draggingShape = false;
  }

  function onMouseMove(
    e: MouseEvent & {
      currentTarget: EventTarget & HTMLCanvasElement;
    },
  ) {
    mouse.x = e.clientX - bounds.left;
    mouse.y = e.clientY - bounds.top;

    if (draggingShape) {
      circle.x = mouse.x + offsetX;
      circle.y = mouse.y + offsetY;
    }
  }

  function draw(ctx: CanvasRenderingContext2D, timestamp: number) {
    if (startTime === undefined) {
      startTime = timestamp;
      lastFrameTime = timestamp;
    }
    let delta = (timestamp - lastFrameTime) / 1000;
    lastFrameTime = timestamp;

    ctx.clearRect(0, 0, canvas.width, canvas.height);

    ctx.beginPath();
    ctx.fillStyle = draggingShape ? "blue" : "red";
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

<canvas
  bind:this={canvas}
  width="800"
  height="800"
  style="border: 1px solid black;"
  onmouseup={onMouseUp}
  onmousedown={onMouseDown}
  onmousemove={onMouseMove}
></canvas>
