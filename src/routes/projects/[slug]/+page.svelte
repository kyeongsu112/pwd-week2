<script>
  let { data } = $props();      // { item, slug }
  let memo = $state('');

  // timetable용 상태
  let timetable = $state([]);
  let newSubject = { subject: '', room: '', day: 1, start: '', end: '', color: '#90caf9' };
  const days = ['월', '화', '수', '목', '금'];

  // 갤러리용 상태
  let galleryImages = $state([]);

  // 시간 문자열(HH:MM) → 분 변환
  function timeToMinutes(t) {
    if (!/^\d{2}:\d{2}$/.test(t)) return NaN;
    const [h, m] = t.split(':').map(Number);
    return h * 60 + m;
  }

  // 로컬스토리지에서 데이터 불러오기
  $effect(() => {
    if (data.slug === 'memo') {
      const saved = localStorage.getItem('memo');
      if (saved) memo = saved;
    }
    if (data.slug === 'timetable') {
      const saved = localStorage.getItem('timetable');
      if (saved) timetable = JSON.parse(saved);
    }
    if (data.slug === 'gallery') {
      const saved = localStorage.getItem('galleryImages');
      if (saved) galleryImages = JSON.parse(saved);
    }
  });

  function saveMemo() {
    localStorage.setItem('memo', memo);
    alert('저장 완료!');
  }

  function saveTimetable() {
    localStorage.setItem('timetable', JSON.stringify(timetable));
  }

  function addSubject() {
    // 입력값 검증
    const startMin = timeToMinutes(newSubject.start);
    const endMin = timeToMinutes(newSubject.end);
    if (
      !newSubject.subject ||
      !newSubject.room ||
      !newSubject.start ||
      !newSubject.end ||
      isNaN(startMin) ||
      isNaN(endMin) ||
      startMin < 0 ||
      endMin > 20 * 60 ||
      startMin >= endMin
    ) {
      alert('시간을 올바르게 입력하세요 (예: 09:00, 13:30, 시작 < 끝, 00:00~20:00)');
      return;
    }
    timetable = [
      ...timetable,
      { ...newSubject, start: newSubject.start, end: newSubject.end }
    ];
    saveTimetable();
    newSubject = { subject: '', room: '', day: 1, start: '', end: '', color: '#90caf9' };
  }

  function removeSubject(idx) {
    timetable = timetable.filter((_, i) => i !== idx);
    saveTimetable();
  }

  // 시간표 셀에 해당 시간에 포함되는 과목 찾기 (분 단위)
  function subjectsAt(day, hour) {
    const cellStart = hour * 60;
    const cellEnd = (hour + 1) * 60;
    return timetable.filter(s => {
      if (s.day !== day) return false;
      const sStart = timeToMinutes(s.start);
      const sEnd = timeToMinutes(s.end);
      return sStart < cellEnd && sEnd > cellStart;
    });
  }

  // 갤러리 이미지 업로드/삭제
  function onImageUpload(event) {
    const files = Array.from(event.target.files);
    files.forEach(file => {
      const reader = new FileReader();
      reader.onload = e => {
        galleryImages = [...galleryImages, e.target.result];
        localStorage.setItem('galleryImages', JSON.stringify(galleryImages));
      };
      reader.readAsDataURL(file);
    });
    event.target.value = '';
  }
  function removeImage(idx) {
    galleryImages = galleryImages.filter((_, i) => i !== idx);
    localStorage.setItem('galleryImages', JSON.stringify(galleryImages));
  }
</script>

<h2>{data.item.title}</h2>
<p>{data.item.body}</p>

{#if data.slug === 'timetable'}
  <div class="card" style="overflow-x:auto;">
    <table border="1" style="width:100%; text-align:center;">
      <thead>
        <tr>
          <th>시간/요일</th>
          {#each days as d}<th>{d}</th>{/each}
        </tr>
      </thead>
      <tbody>
        {#each Array(12).fill(0).map((_,i)=>i+9) as hour} <!-- 9~20시 -->
          <tr>
            <td>{hour}:00</td>
            {#each [1,2,3,4,5] as d}
              <td>
                {#each subjectsAt(d, hour) as s, idx}
                  <div style="background:{s.color};border-radius:6px;padding:2px 4px;margin:2px 0;">
                    <b>{s.subject}</b><br />
                    <small>{s.room}</small>
                    <small>
                      {s.start}~{s.end}
                    </small>
                    <button on:click={() => removeSubject(timetable.findIndex(x=>x===s))} style="margin-left:4px;font-size:10px;">삭제</button>
                  </div>
                {/each}
              </td>
            {/each}
          </tr>
        {/each}
      </tbody>
    </table>
  </div>
  <form class="card" style="margin-top:1rem;" on:submit|preventDefault={addSubject}>
    <h3>과목 추가</h3>
    <input placeholder="과목명" bind:value={newSubject.subject} required />
    <input placeholder="강의실" bind:value={newSubject.room} required />
    <select bind:value={newSubject.day}>
      {#each [1,2,3,4,5] as d}
        <option value={d}>{days[d-1]}</option>
      {/each}
    </select>
    <label style="margin-left:8px;">
      시작 시간
      <input type="time" bind:value={newSubject.start} required style="width:100px;" />
    </label>
    ~
    <label>
      끝 시간
      <input type="time" bind:value={newSubject.end} required style="width:100px;" />
    </label>
    <input type="color" bind:value={newSubject.color} style="width:40px; margin-left:8px;" />
    <button type="submit">추가</button>
  </form>
  <p style="opacity:.6">브라우저 로컬에 저장됩니다.</p>
{/if}

{#if data.slug === 'gallery'}
  <div class="card">
    <h3>이미지 업로드</h3>
    <input type="file" accept="image/*" multiple on:change={onImageUpload} />
    <div style="display:flex; flex-wrap:wrap; gap:12px; margin-top:1rem;">
      {#each galleryImages as img, idx}
        <div style="position:relative;">
          <img src={img} alt="gallery" style="width:120px; height:120px; object-fit:cover; border-radius:8px; border:1px solid #ccc;" />
          <button on:click={() => removeImage(idx)} style="position:absolute;top:2px;right:2px;font-size:12px;">삭제</button>
        </div>
      {/each}
      {#if galleryImages.length === 0}
        <p style="opacity:.6">아직 업로드된 이미지가 없습니다.</p>
      {/if}
    </div>
    <p style="opacity:.6">이미지는 브라우저에만 저장됩니다.</p>
  </div>
{/if}

{#if data.slug === 'memo'}
  <textarea rows="6" bind:value={memo} class="card" style="width:100%"></textarea>
  <button on:click={saveMemo}>메모 저장</button>
  <p style="opacity:.6">브라우저 로컬에만 저장됩니다.</p>
{/if}