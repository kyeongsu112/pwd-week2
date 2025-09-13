<!-- src/routes/+layout.svelte -->
<script>
  // v5: <slot> 대신 children 스니펫을 받아 {@render children()}로 렌더
  let { children } = $props();
</script>

<svelte:head>
  <title>AJOU Mini Portfolio</title>
  <meta name="description" content="SvelteKit + Vercel mini portfolio" />
</svelte:head>

<nav>
<nav class="main-nav">
  <a class="nav-btn" href="/">Home</a>
  <a class="nav-btn" href="/about">About</a>
  <a class="nav-btn" href="/projects">Projects</a>
  <a class="nav-btn" href="/contact">Contact</a>
</nav>

<main>
  {@render children()}
</main>

<style global>
<style global>
  @import '../app.css';
  .main-nav {
    display: flex;
    justify-content: center;
    gap: 1.5rem;
    margin: 2rem 0 2.5rem 0;
  }
  .nav-btn {
    display: inline-block;
    background: var(--brand, #5b3df5);
    color: #fff;
    font-size: 1.15rem;
    font-weight: 600;
    padding: 0.7em 2em;
    border-radius: 2em;
    text-decoration: none;
    box-shadow: 0 2px 8px #0001;
    transition: background 0.2s, transform 0.1s;
  }
  .nav-btn:hover, .main-btn:hover {
    background: #3d28b0;
    transform: translateY(-2px) scale(1.04);
  }
  .main-btn {
    display: inline-block;
    background: var(--brand, #5b3df5);
    color: #fff;
    font-size: 1rem;
    font-weight: 500;
    padding: 0.6em 1.5em;
    border-radius: 1.5em;
    text-decoration: none;
    margin: 0.5em 0;
    border: none;
    cursor: pointer;
    transition: background 0.2s, transform 0.1s;
  }
</style>