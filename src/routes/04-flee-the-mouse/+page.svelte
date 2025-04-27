<script lang="ts">
  import { onMount } from "svelte";

  const SPEED = 200.0;

  let canvas: HTMLCanvasElement;
  let bounds: DOMRect;
  let startTime: number;
  let lastFrameTime: number;
  let mousePos = $state({ x: 10, y: 10 });
  let showPath = $state(false);

  const circle = {
    position: { x: 200, y: 200 },
    radius: 15,
  };

  function onmousemove(
    e: MouseEvent & { currentTarget: EventTarget & HTMLCanvasElement },
  ) {
    mousePos = {
      x: e.clientX - bounds.left,
      y: e.clientY - bounds.top,
    };
  }

  onMount(() => {
    const ctx = canvas.getContext("2d") as CanvasRenderingContext2D;
    ctx.fillStyle = "red";
    bounds = canvas.getBoundingClientRect();

    function drawDirectionVector(x: number, y: number) {
      // Draw Direction Vector (top right for visualising it)
      const position = { x: 700, y: 100 };
      ctx.fillText("Direction Vector", position.x - 35, position.y - 80);
      ctx.fillText(
        `(x: ${x.toFixed(2)}, y: ${y.toFixed(2)})`,
        position.x - 35,
        position.y - 65,
      );
      ctx.beginPath();
      ctx.moveTo(position.x, position.y);
      ctx.lineTo(position.x + x * 50, position.y + y * 50);
      ctx.stroke();
    }

    function draw(timestamp: number) {
      if (startTime === undefined) {
        startTime = timestamp;
        lastFrameTime = timestamp;
      }
      const deltaTime = (timestamp - lastFrameTime) / 1000;
      lastFrameTime = timestamp;

      // Clear Screen
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // Calc Direction (and normalise)
      let dirX = circle.position.x - mousePos.x;
      let dirY = circle.position.y - mousePos.y;
      let length = Math.sqrt(dirX ** 2 + dirY ** 2);
      dirX /= length;
      dirY /= length;

      // Fun visual of normalised vector
      drawDirectionVector(dirX, dirY);

      circle.position.x += dirX * SPEED * deltaTime;
      circle.position.y += dirY * SPEED * deltaTime;

      // Draw Circle
      ctx.beginPath();
      ctx.arc(
        circle.position.x,
        circle.position.y,
        circle.radius,
        0,
        Math.PI * 2,
      );
      ctx.fill();

      // Draw Line between Circle and Mouse
      if (showPath) {
        ctx.beginPath();
        ctx.moveTo(circle.position.x, circle.position.y);
        ctx.lineTo(mousePos.x, mousePos.y);
        ctx.stroke();
      }

      ctx.fillText(`${deltaTime}`, 0, 10);

      requestAnimationFrame(draw);
    }

    requestAnimationFrame(draw);
  });
</script>

<canvas
  bind:this={canvas}
  width="800"
  height="800"
  style="border: 1px solid black;"
  {onmousemove}
></canvas>

<div>
  <label for="showPath">Show Path?:</label>
  <input id="showPath" type="checkbox" bind:checked={showPath} />
  <button
    onclick={() =>
      (circle.position = { x: canvas.width / 2, y: canvas.height / 2 })}
    >Reset Circle To Middle</button
  >
</div>
