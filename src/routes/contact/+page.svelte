<!-- src/routes/contact/+page.svelte -->
<section class="card">
  <h2>Contact</h2>
  <p>문의나 연락은 아래 이메일로 보내주세요.</p>
  <ul>
    <li>Email: <a href="mailto:kyeongsujin@gmail.com">kyeongsujin@gmail.com</a></li>
    <li>GitHub: <a href="https://github.com/kyeongsu112" target="_blank">kyeongsu112</a></li>
  </ul>
  <!-- 간단한 폼 예시 -->
  <form on:submit|preventDefault={() => alert('메시지가 전송되었습니다! (실제 전송 기능은 미구현)')}>
    <input type="text" placeholder="Your name" required />
    <input type="email" placeholder="Your email" required />
    <textarea rows="4" placeholder="Your message" required></textarea>
    <button type="submit">Send</button>
  </form>
</section>