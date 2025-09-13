<script>
  let { data } = $props();      // { item, slug }
  let memo = $state('');

  // timetable용 상태
  let timetable = $state([]);
  // 오전/오후, 시, 분 선택
  let newSubject = {
    subject: '', room: '', day: 1,
    startAmPm: 'AM', startHour: 9, startMin: 0,
    endAmPm: 'AM', endHour: 10, endMin: 0,
    color: '#90caf9'
  };
  const days = ['월', '화', '수', '목', '금'];
  const hours = Array.from({length:12},(_,i)=>i+1); // 1~12
  const minutes = [0, 15, 30, 45];
  const ampm = ['AM', 'PM'];

  // 갤러리용 상태
  let galleryImages = $state([]);

  // 15분 단위 타임슬롯 생성 (9:00~20:00)
  function pad(n) { return n < 10 ? '0'+n : n; }
  const timeSlots = [];
  for(let h=9; h<=20; h++) {
    for(let m=0; m<60; m+=15) {
      timeSlots.push(`${pad(h)}:${pad(m)}`);
    }
  }

  function to24h(amPm, hour) {
    if (amPm === 'AM') return hour === 12 ? 0 : hour;
    return hour === 12 ? 12 : hour + 12;
  }
  function toTimeString(amPm, hour, min) {
    return `${pad(to24h(amPm, hour))}:${pad(min)}`;
  }
  function timeToMinutes(t) {
    if (!/^\d{2}:\d{2}$/.test(t)) return NaN;
    const [h, m] = t.split(':').map(Number);
    return h * 60 + m;
  }
  function slotToMinutes(slot) {
    const [h, m] = slot.split(':').map(Number);
    return h*60 + m;
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
    const start = toTimeString(newSubject.startAmPm, newSubject.startHour, newSubject.startMin);
    const end = toTimeString(newSubject.endAmPm, newSubject.endHour, newSubject.endMin);
    const startMin = timeToMinutes(start);
    const endMin = timeToMinutes(end);
    if (
      !newSubject.subject ||
      !newSubject.room ||
      isNaN(startMin) ||
      isNaN(endMin) ||
      startMin < 9*60 ||
      endMin > 20*60+45 ||
      startMin >= endMin
    ) {
      alert('시간을 올바르게 입력하세요 (9:00~20:45, 시작 < 끝)');
      return;
    }
    timetable = [
      ...timetable,
      {
        subject: newSubject.subject,
        room: newSubject.room,
        day: newSubject.day,
        start, end,
        color: newSubject.color
      }
    ];
    saveTimetable();
    newSubject = {
      subject: '', room: '', day: 1,
      startAmPm: 'AM', startHour: 9, startMin: 0,
      endAmPm: 'AM', endHour: 10, endMin: 0,
      color: '#90caf9'
    };
  }

  function removeSubject(idx) {
    timetable = timetable.filter((_, i) => i !== idx);
    saveTimetable();
  }

  // 15분 셀에 해당 시간에 포함되는 과목 찾기
  function subjectsAt15min(day, slot) {
    const cellStart = slotToMinutes(slot);
    const cellEnd = cellStart + 15;
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
  <div class="card" style="overflow-x:auto; max-width:100vw;">
    <table border="1" style="width:100%; text-align:center; font-size:0.95em;">
      <thead>
        <tr>
          <th>시간/요일</th>
          {#each days as d}<th>{d}</th>{/each}
        </tr>
      </thead>
      <tbody>
        {#each timeSlots as slot}
          <tr>
            <td>{slot}</td>
            {#each [1,2,3,4,5] as d}
              <td>
                {#each subjectsAt15min(d, slot) as s, idx}
                  <div style="background:{s.color};border-radius:6px;padding:2px 4px;margin:2px 0;">
                    <b>{s.subject}</b><br />
                    <small>{s.room}</small>
                    <small>{s.start}~{s.end}</small>
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
      <select bind:value={newSubject.startAmPm}>
        {#each ampm as ap}<option value={ap}>{ap}</option>{/each}
      </select>
      <select bind:value={newSubject.startHour}>
        {#each hours as h}<option value={h}>{h}</option>{/each}
      </select>
      :
      <select bind:value={newSubject.startMin}>
        {#each minutes as m}<option value={m}>{m.toString().padStart(2,'0')}</option>{/each}
      </select>
    </label>
    ~
    <label>
      끝 시간
      <select bind:value={newSubject.endAmPm}>
        {#each ampm as ap}<option value={ap}>{ap}</option>{/each}
      </select>
      <select bind:value={newSubject.endHour}>
        {#each hours as h}<option value={h}>{h}</option>{/each}
      </select>
      :
      <select bind:value={newSubject.endMin}>
        {#each minutes as m}<option value={m}>{m.toString().padStart(2,'0')}</option>{/each}
      </select>
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
  {#if data.slug === 'gallery'}
    <div class="card">
      <h3>이미지 업로드</h3>
      <input type="file" accept="image/*" multiple on:change={onImageUpload} class="main-btn" />
      <div class="gallery-grid">
        {#each galleryImages as img, idx}
          <div class="gallery-thumb">
            <img src={img} alt="gallery" />
            <button class="main-btn small" on:click={() => removeImage(idx)}>삭제</button>
          </div>
        {/each}
        {#if galleryImages.length === 0}
          <p style="opacity:.6">아직 업로드된 이미지가 없습니다.</p>
        {/if}
      </div>
      <p style="opacity:.6">이미지는 브라우저에만 저장됩니다.</p>
    </div>
  {/if}

  <style>
  .gallery-grid {
    display: flex;
    flex-wrap: wrap;
    gap: 16px;
    margin-top: 1rem;
  }
  .gallery-thumb {
    position: relative;
    width: 140px;
    height: 140px;
    border-radius: 10px;
    overflow: hidden;
    box-shadow: 0 2px 8px #0002;
    background: #fff;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }
  .gallery-thumb img {
    width: 100%;
    height: 100px;
    object-fit: cover;
    border-radius: 10px 10px 0 0;
    border-bottom: 1px solid #eee;
  }
  .gallery-thumb .main-btn.small {
    font-size: 0.9em;
    padding: 0.3em 1em;
    margin: 8px 0 0 0;
  }
  </style>

{#if data.slug === 'memo'}
  <textarea rows="6" bind:value={memo} class="card" style="width:100%"></textarea>
  <button on:click={saveMemo}>메모 저장</button>
  <p style="opacity:.6">브라우저 로컬에만 저장됩니다.</p>
{/if}