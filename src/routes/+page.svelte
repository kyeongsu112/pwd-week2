<!-- src/routes/+page.svelte -->
<script>
  let name = $state('');
  let welcome = $derived(name ? `Hello, ${name}!` : 'Welcome!');

  // 이름을 localStorage에 저장하고 불러오기
  $effect(() => {
    const saved = localStorage.getItem('homeName');
    if (saved) name = saved;
  });
  $effect(() => {
    localStorage.setItem('homeName', name);
  }, [name]);

  function randomize() {
    const msgs = [
      '오늘도 멋진 하루 보내세요!',
      '디지털미디어학과에서 새로운 도전을!',
      '코딩은 즐거움입니다 :)',
      '작은 변화가 큰 성장을 만듭니다.',
      '포트폴리오 방문을 환영합니다!'
    ];
    alert(msgs[Math.floor(Math.random() * msgs.length)]);
  }
</script>

<!-- 상단 네비게이션은 layout에서 렌더링되므로 홈에서는 nav를 제거합니다. -->

<section class="card" style="text-align:center;">
  <h1>{welcome}</h1>
  <label>
    이름을 입력해보세요:
    <input placeholder="이름" bind:value={name} style="margin-left:0.5em;" />
  </label>
  <p>미리보기: <strong>{name || '(입력 대기)'}</strong></p>
  <button on:click={randomize} class="main-btn">랜덤 응원 메시지</button>
  <div style="margin:1.5rem 0;">
    <a href="/projects" class="main-btn" style="margin:0 0.5em;">프로젝트 보기</a>
    <a href="/about" class="main-btn" style="margin:0 0.5em;">자기소개</a>
    <a href="/contact" class="main-btn" style="margin:0 0.5em;">연락하기</a>
  </div>
</section>

<section class="card">
  <h2>기술 스택</h2>
  <p>JavaScript, Svelte, HTML, CSS, Vercel, GitHub 등</p>
</section>

<section class="card">
  <h2>최근 프로젝트</h2>
  <ul>
    <li><a href="/projects/timetable">Timetable Helper</a></li>
    <li><a href="/projects/gallery">Image Gallery</a></li>
    <li><a href="/projects/memo">Memo Pad</a></li>
  </ul>
</section>

<style>
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