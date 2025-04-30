<script lang="ts">
  import { page } from "$app/state";

  let { children } = $props();

  let routes = [
    "/01-mouse-movement",
    "/02-key-movement",
    "/03-seek-the-mouse",
    "/04-flee-the-mouse",
    "/05-arrival-behaviour",
    "/06-rotation",
    "/07-rotation-with-atan2",
    "/08-from-memory-test",
    "/09-drag-and-drop",
  ];
</script>

<nav>
  {#each routes as route}
    <a class:active={route === page.url.pathname} href={route}
      >{route.replaceAll("-", " ").replace("/", "")}</a
    >
  {/each}
</nav>

{@render children?.()}

<style>
  nav {
    display: flex;
    flex-wrap: wrap;
    gap: 0.4rem 1rem;
    padding: 1rem 0.8rem;
  }

  nav a {
    text-decoration: none;
    font-family: sans-serif;
    color: blue;
    border: 1px solid blue;
    border-radius: 5px;
    padding: 0.5rem 0.8rem;
    text-transform: capitalize;
  }

  .active {
    font-weight: bold;
  }
</style>
