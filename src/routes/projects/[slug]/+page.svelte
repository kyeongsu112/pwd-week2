<script>
  let { data } = $props();      // { item, slug }
  let memo = $state('');

  // timetable용 상태
  let timetable = $state([]);
  let newSubject = { subject: '', room: '', day: 1, start: 9, end: 10, color: '#90caf9' };
  const days = ['월', '화', '수', '목', '금'];

  $effect(() => {
    if (data.slug === 'memo') {
      const saved = localStorage.getItem('memo');
      if (saved) memo = saved;
    }
    if (data.slug === 'timetable') {
      const saved = localStorage.getItem('timetable');
      if (saved) timetable = JSON.parse(saved);
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
    timetable = [...timetable, { ...newSubject }];
    saveTimetable();
    newSubject = { subject: '', room: '', day: 1, start: 9, end: 10, color: '#90caf9' };
  }

  function removeSubject(idx) {
    timetable = timetable.filter((_, i) => i !== idx);
    saveTimetable();
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
        {#each Array(9).fill(0).map((_,i)=>i+9) as hour}
          <tr>
            <td>{hour}:00</td>
            {#each [1,2,3,4,5] as d}
              <td>
                {#each timetable.filter(s => s.day === d && s.start <= hour && s.end > hour) as s, idx}
                  <div style="background:{s.color};border-radius:6px;padding:2px 4px;margin:2px 0;">
                    <b>{s.subject}</b><br />
                    <small>{s.room}</small>
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
    <input type="number" min="9" max="18" bind:value={newSubject.start} required style="width:60px" />시 ~
    <input type="number" min="10" max="19" bind:value={newSubject.end} required style="width:60px" />시
    <input type="color" bind:value={newSubject.color} style="width:40px;" />
    <button type="submit">추가</button>
  </form>
  <p style="opacity:.6">브라우저 로컬에 저장됩니다.</p>
{/if}

{#if data.slug === 'memo'}
  <textarea rows="6" bind:value={memo} class="card" style="width:100%"></textarea>
  <button on:click={saveMemo}>메모 저장</button>
  <p style="opacity:.6">브라우저 로컬에만 저장됩니다.</p>
{/if}