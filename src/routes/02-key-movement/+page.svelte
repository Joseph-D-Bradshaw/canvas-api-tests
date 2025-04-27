<script lang="ts">
  import { onMount } from "svelte";

  const SPEED = 200.0;

  const rect = {
    position: { x: 10, y: 10 },
    width: 100,
    height: 100,
  };

  const keys = $state({
    w: false,
    s: false,
    a: false,
    d: false,
    shift: false,
  });

  let canvas: HTMLCanvasElement;
  let bounds: DOMRect;
  let startTime: number;
  let lastFrameTime: number;
  let velocity = $state({ x: 0, y: 0 });

  function onkeydown(
    e: KeyboardEvent & {
      currentTarget: EventTarget & HTMLCanvasElement;
    },
  ) {
    const key = e.key.toLowerCase();
    if (key in keys) keys[key as keyof typeof keys] = true;
  }

  function onkeyup(
    e: KeyboardEvent & {
      currentTarget: EventTarget & HTMLCanvasElement;
    },
  ) {
    const key = e.key.toLowerCase();
    if (key in keys) keys[key as keyof typeof keys] = false;
  }

  onMount(() => {
    canvas.focus();
    const ctx = canvas.getContext("2d") as CanvasRenderingContext2D;
    ctx.fillStyle = "red";
    bounds = canvas.getBoundingClientRect();

    function draw(timestamp: number) {
      if (startTime === undefined) {
        startTime = timestamp;
        lastFrameTime = timestamp;
      }
      const elapsed = timestamp - startTime;
      const deltaTime = (timestamp - lastFrameTime) / 1000;
      lastFrameTime = timestamp;

      ctx.clearRect(0, 0, canvas.width, canvas.height);

      velocity.x = 0;
      velocity.y = 0;

      let speedMultiplier = keys.shift ? 2 : 1;

      if (keys.w) velocity.y -= SPEED * speedMultiplier;
      if (keys.s) velocity.y += SPEED * speedMultiplier;
      if (keys.a) velocity.x -= SPEED * speedMultiplier;
      if (keys.d) velocity.x += SPEED * speedMultiplier;

      rect.position.x += velocity.x * deltaTime;
      rect.position.y += velocity.y * deltaTime;

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
  tabindex="0"
  style="border: 1px solid black;"
  {onkeydown}
  {onkeyup}
></canvas>

<p>Controls: WASD and Shift</p>
