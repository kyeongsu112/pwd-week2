<!-- src/routes/+page.svelte -->
<script>
  let name = '';
  let welcome = name ? `Hello, ${name}!` : 'Welcome!';

  function randomize() {
    const msgs = ['웹 개발 재밌다!', 'SvelteKit 금방 익힘', 'Vercel로 바로 배포!'];
    alert(msgs[Math.floor(Math.random() * msgs.length)]);
  }
</script>

<h1>{welcome}</h1>

<label>
  Your name: <input placeholder="type your name" bind:value={name} />
</label>
<p>미리보기: <strong>{name || '(입력 대기)'}</strong></p>

<button on:click={randomize}>랜덤 메시지</button>

<section class="card">
  <h2>소개</h2>
  <p>안녕하세요! 아주대학교 학생의 미니 포트폴리오 사이트입니다.<br>
  SvelteKit과 Vercel을 활용하여 빠르게 배포할 수 있습니다.</p>
</section>