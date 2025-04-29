<script lang="ts">
  import { onMount } from "svelte";

  const SPEED = 200.0;
  const ARRIVAL_RADIUS = 150.0;

  let canvas: HTMLCanvasElement;
  let bounds: DOMRect;
  let startTime: number;
  let lastFrameTime: number;
  let mousePos = $state({ x: 10, y: 10 });
  let showPythagoras = $state(true);

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
      ctx.strokeStyle = "black";
      ctx.moveTo(position.x, position.y);
      ctx.lineTo(position.x + x * 50, position.y + y * 50);
      ctx.stroke();
    }

    function drawLengthLine() {
      if (showPythagoras) {
        let dx = mousePos.x - circle.position.x;
        let dy = mousePos.y - circle.position.y;
        let length = Math.sqrt(dx ** 2 + dy ** 2);
        ctx.beginPath();
        ctx.fillStyle = "orange";
        ctx.strokeStyle = "orange";
        ctx.moveTo(circle.position.x, circle.position.y);
        ctx.lineTo(mousePos.x, mousePos.y);
        ctx.stroke();
        ctx.fillText(
          `length: ${length.toFixed(2)}`,
          mousePos.x + 10 - dx / 2,
          mousePos.y + 10 - dy / 2,
        );
      }
    }

    function drawDxLine() {
      if (showPythagoras) {
        let dx = mousePos.x - circle.position.x;
        ctx.beginPath();
        ctx.fillStyle = "blue";
        ctx.strokeStyle = "blue";
        ctx.moveTo(circle.position.x, circle.position.y);
        ctx.lineTo(mousePos.x, circle.position.y);
        ctx.stroke();
        ctx.fillText(
          `dx: ${dx.toFixed(2)}`,
          mousePos.x - dx / 2,
          circle.position.y - 10,
        );
      }
    }

    function drawDyLine() {
      if (showPythagoras) {
        let dy = mousePos.y - circle.position.y;
        ctx.beginPath();
        ctx.fillStyle = "green";
        ctx.strokeStyle = "green";
        ctx.moveTo(mousePos.x, circle.position.y);
        ctx.lineTo(mousePos.x, mousePos.y);
        ctx.stroke();
        ctx.fillText(
          `dy: ${dy.toFixed(2)}`,
          mousePos.x + 10,
          mousePos.y - dy / 2,
        );
      }
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
      let dx = mousePos.x - circle.position.x;
      let dy = mousePos.y - circle.position.y;
      let dirX = dx;
      let dirY = dy;
      let length = Math.sqrt(dirX ** 2 + dirY ** 2);
      dirX /= length;
      dirY /= length;

      // Fun visual of normalised vector
      drawDirectionVector(dirX, dirY);

      // Calculate distance to target
      const distance = Math.sqrt(dx ** 2 + dy ** 2);
      let speed = SPEED;
      if (distance < ARRIVAL_RADIUS) {
        let speedBoost = distance < 15 ? 10.0 : 1.0;
        speed = SPEED * (distance / ARRIVAL_RADIUS) * speedBoost;
      }

      circle.position.x += dirX * speed * deltaTime;
      circle.position.y += dirY * speed * deltaTime;

      // Draw debug lines for geometry visual
      drawLengthLine();
      drawDxLine();
      drawDyLine();

      // Draw Circle
      ctx.beginPath();
      ctx.fillStyle = "red";
      ctx.arc(
        circle.position.x,
        circle.position.y,
        circle.radius,
        0,
        Math.PI * 2,
      );
      ctx.fill();

      ctx.fillText(`Speed: ${speed.toFixed(2)}`, 0, 10);
      ctx.fillText(`dx: ${(dx / length).toFixed(2)}`, 0, 20);
      ctx.fillText(`dy: ${(dy / length).toFixed(2)}`, 0, 30);

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
  <label for="showPath">Show Pythagoras?:</label>
  <input id="showPath" type="checkbox" bind:checked={showPythagoras} />
  <button
    onclick={() =>
      (circle.position = { x: canvas.width / 2, y: canvas.height / 2 })}
    >Reset Circle To Middle</button
  >
</div>
