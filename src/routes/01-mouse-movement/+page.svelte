<script lang="ts">
  import { onMount } from "svelte";

  let canvas: HTMLCanvasElement;
  let bounds: DOMRect;
  let startTime: number;
  let mousePos = $state({ x: 10, y: 10 });

  const rect = {
    position: { x: 10, y: 10 },
    width: 100,
    height: 100,
  };

  function onmousemove(
    e: MouseEvent & { currentTarget: EventTarget & HTMLCanvasElement },
  ) {
    mousePos = {
      x: e.clientX - bounds.left - rect.width / 2,
      y: e.clientY - bounds.top - rect.height / 2,
    };
  }

  onMount(() => {
    const ctx = canvas.getContext("2d") as CanvasRenderingContext2D;
    ctx.fillStyle = "red";
    bounds = canvas.getBoundingClientRect();

    function draw(timestamp: number) {
      if (startTime === undefined) startTime = timestamp;
      const elapsed = timestamp - startTime;
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      rect.position.x = mousePos.x;
      rect.position.y = mousePos.y;
      ctx.fillRect(rect.position.x, rect.position.y, rect.width, rect.height);
      ctx.fillText(`${elapsed}`, 0, 10);
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
