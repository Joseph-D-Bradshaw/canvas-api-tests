<script lang="ts">
  import { onMount } from "svelte";

  const SPEED = 200.0;
  const ROTATION_SPEED = 10.0;
  const ARRIVAL_RADIUS = 150.0;

  let canvas: HTMLCanvasElement;
  let bounds: DOMRect;
  let startTime: number;
  let lastFrameTime: number;
  let mousePos = $state({ x: 10, y: 10 });
  let showPythagoras = $state(true);

  const triangle = {
    position: { x: 200, y: 200 },
    width: 20,
    height: 30,
    rotationInRads: 0,
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
      let dx = mousePos.x - triangle.position.x;
      let dy = mousePos.y - triangle.position.y;
      let dirX = dx;
      let dirY = dy;
      let length = Math.sqrt(dirX ** 2 + dirY ** 2);
      dirX /= length;
      dirY /= length;

      // Fun visual of normalised vector
      drawDirectionVector(ctx, dirX, dirY);

      // Calculate distance to target
      const distance = Math.sqrt(dx ** 2 + dy ** 2);
      let speed = SPEED;
      if (distance < ARRIVAL_RADIUS) {
        let speedBoost = distance < 15 ? 10.0 : 1.0;
        speed = SPEED * (distance / ARRIVAL_RADIUS) * speedBoost;
      }

      triangle.position.x += dirX * speed * deltaTime;
      triangle.position.y += dirY * speed * deltaTime;
      triangle.rotationInRads += ROTATION_SPEED * deltaTime;

      // Draw debug lines for geometry visual
      drawLengthLine(ctx);
      drawDxLine(ctx);
      drawDyLine(ctx);

      drawTriangle(ctx);

      ctx.fillText(`Speed: ${speed.toFixed(2)}`, 0, 10);
      ctx.fillText(`dx: ${(dx / length).toFixed(2)}`, 0, 20);
      ctx.fillText(`dy: ${(dy / length).toFixed(2)}`, 0, 30);

      requestAnimationFrame(draw);
    }

    requestAnimationFrame(draw);
  });

  function drawTriangle(ctx: CanvasRenderingContext2D) {
    ctx.save();
    ctx.fillStyle = "red";
    ctx.translate(triangle.position.x, triangle.position.y);
    ctx.rotate(triangle.rotationInRads);
    ctx.beginPath();
    ctx.moveTo(0, -triangle.height / 2);
    ctx.lineTo(-triangle.width / 2, triangle.height / 2);
    ctx.lineTo(triangle.width / 2, triangle.height / 2);
    ctx.closePath();
    ctx.fill();
    ctx.restore();
  }

  function drawDxLine(ctx: CanvasRenderingContext2D) {
    if (showPythagoras) {
      let dx = mousePos.x - triangle.position.x;
      ctx.beginPath();
      ctx.fillStyle = "blue";
      ctx.strokeStyle = "blue";
      ctx.moveTo(triangle.position.x, triangle.position.y);
      ctx.lineTo(mousePos.x, triangle.position.y);
      ctx.stroke();
      ctx.fillText(
        `dx: ${dx.toFixed(2)}`,
        mousePos.x - dx / 2,
        triangle.position.y - 10,
      );
    }
  }

  function drawDyLine(ctx: CanvasRenderingContext2D) {
    if (showPythagoras) {
      let dy = mousePos.y - triangle.position.y;
      ctx.beginPath();
      ctx.fillStyle = "green";
      ctx.strokeStyle = "green";
      ctx.moveTo(mousePos.x, triangle.position.y);
      ctx.lineTo(mousePos.x, mousePos.y);
      ctx.stroke();
      ctx.fillText(
        `dy: ${dy.toFixed(2)}`,
        mousePos.x + 10,
        mousePos.y - dy / 2,
      );
    }
  }

  function drawLengthLine(ctx: CanvasRenderingContext2D) {
    if (showPythagoras) {
      let dx = mousePos.x - triangle.position.x;
      let dy = mousePos.y - triangle.position.y;
      let length = Math.sqrt(dx ** 2 + dy ** 2);
      ctx.beginPath();
      ctx.fillStyle = "orange";
      ctx.strokeStyle = "orange";
      ctx.moveTo(triangle.position.x, triangle.position.y);
      ctx.lineTo(mousePos.x, mousePos.y);
      ctx.stroke();
      ctx.fillText(
        `length: ${length.toFixed(2)}`,
        mousePos.x + 10 - dx / 2,
        mousePos.y + 10 - dy / 2,
      );
    }
  }

  function drawDirectionVector(
    ctx: CanvasRenderingContext2D,
    x: number,
    y: number,
  ) {
    const position = { x: 700, y: 100 };
    ctx.fillText("Direction Vector", position.x - 35, position.y - 80);
    ctx.fillText(
      `(x: ${x.toFixed(2)}, y: ${y.toFixed(2)})`,
      position.x - 35,
      position.y - 65,
    );
    ctx.beginPath();
    ctx.strokeStyle = "black";
    ctx.moveTo(position.x, position.y);
    ctx.lineTo(position.x + x * 50, position.y + y * 50);
    ctx.stroke();
  }
</script>

<canvas
  bind:this={canvas}
  width="800"
  height="800"
  style="border: 1px solid black;"
  {onmousemove}
></canvas>

<div>
  <label for="showPath">Show Pythagoras?:</label>
  <input id="showPath" type="checkbox" bind:checked={showPythagoras} />
  <button
    onclick={() =>
      (triangle.position = { x: canvas.width / 2, y: canvas.height / 2 })}
    >Reset Triangle To Middle</button
  >
</div>
