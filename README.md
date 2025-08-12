<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>향상된 용돈 관리 (웹 버전)</title>
  <link rel="preconnect" href="https://cdn.jsdelivr.net" />
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    :root {
      --bg: #0b1220;
      --panel: #111a2b;
      --muted: #7f8ba7;
      --text: #eaf1ff;
      --accent: #6ea8ff;
      --accent-2: #55d0a2;
      --danger: #ff6b6b;
      --warning: #f1c40f;
    }
    * { box-sizing: border-box; }
    body {
      margin: 0; padding: 24px; font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial, "Apple Color Emoji", "Segoe UI Emoji";
      background: radial-gradient(1200px 800px at 20% -10%, #1a2744 0%, var(--bg) 55%);
      color: var(--text);
    }
    h1 { font-size: 28px; margin: 0 0 12px; }
    h2 { font-size: 20px; margin: 18px 0 10px; color: #d9e6ff; }
    .grid { display: grid; grid-template-columns: repeat(12, 1fr); gap: 16px; }
    .card {
      grid-column: span 12; background: linear-gradient(180deg, #121c31 0%, var(--panel) 100%);
      border: 1px solid #22304d; border-radius: 16px; padding: 16px; box-shadow: 0 10px 30px rgba(0,0,0,.35);
    }
    @media (min-width: 980px) {
      .span-4 { grid-column: span 4; }
      .span-6 { grid-column: span 6; }
      .span-8 { grid-column: span 8; }
      .span-12 { grid-column: span 12; }
    }
    label { display: block; font-size: 13px; color: var(--muted); margin-top: 10px; }
    input[type="text"], input[type="number"], input[type="date"], select {
      width: 100%; padding: 10px 12px; margin-top: 6px; border-radius: 12px; border: 1px solid #253452;
      background: #0e1629; color: var(--text);
    }
    button { cursor: pointer; border: none; padding: 10px 14px; border-radius: 12px; font-weight: 600; }
    .btn { background: #1a2b4a; color: #e5efff; border: 1px solid #26395f; }
    .btn:hover { filter: brightness(1.08); }
    .btn-primary { background: var(--accent); color: #0a1630; border: none; }
    .btn-ghost { background: transparent; border: 1px solid #2b3c5e; color: #cfe0ff; }
    .btn-danger { background: var(--danger); color: #1a0d0d; }
    .row { display: grid; grid-template-columns: repeat(12, 1fr); gap: 10px; }
    .row > .col-3 { grid-column: span 3; }
    .row > .col-4 { grid-column: span 4; }
    .row > .col-6 { grid-column: span 6; }
    .row > .col-12 { grid-column: span 12; }
    .muted { color: var(--muted); font-size: 13px; }
    .stat {
      display: grid; grid-template-columns: 1fr auto; align-items: center; padding: 10px 14px; border-radius: 12px; background: #0f1a30; border: 1px solid #22304d;
    }
    .stat .value { font-size: 20px; font-weight: 800; letter-spacing: .2px; }
    .pill { display:inline-flex; align-items:center; gap:6px; font-size:12px; padding:4px 8px; border-radius: 999px; border:1px solid #2b3c5e; color:#cfe0ff; }
    table { width: 100%; border-collapse: collapse; }
    th, td { padding: 10px 8px; border-bottom: 1px solid #243151; text-align: left; font-size: 14px; }
    th { color: #bcd4ff; font-weight: 700; }
    tr:hover td { background: #0f182b; }
    .status-ok { color: var(--accent-2); font-weight: 700; }
    .status-over { color: var(--danger); font-weight: 700; }
    .flex { display:flex; gap: 8px; flex-wrap: wrap; }
    .right { text-align:right; }
    .chip { background:#0e1730; border:1px solid #243151; padding:6px 10px; border-radius:999px; font-size:12px; color:#cddfff; }
    .hint { font-size: 12px; color: #9bb1d8; }
    .divider { height: 1px; background: #22304d; margin: 12px 0; }
    .danger { color: var(--danger); }
    .good { color: var(--accent-2); }
    .warning { color: var(--warning); }
    canvas { background: #0b1327; border: 1px solid #22304d; border-radius: 12px; padding: 8px; }
  </style>
</head>
<body>
  <header class="grid" style="align-items:end; margin-bottom: 10px;">
    <div class="card span-8">
      <h1>향상된 용돈 관리 (웹)</h1>
      <div class="muted">예산 제약, 한계효용, 가중치 기반 재배분까지 — 파이썬 CLI를 웹으로 완전 이식</div>
      <div class="divider"></div>
      <div class="grid" style="gap:12px; grid-template-columns: repeat(4, 1fr);">
        <div class="stat"><div>총 예산</div><div class="value" id="statBudget">-</div></div>
        <div class="stat"><div>총 지출</div><div class="value" id="statSpent">-</div></div>
        <div class="stat"><div>잔여 예산</div><div class="value" id="statRemain">-</div></div>
        <div class="stat"><div>저축 제외 잔여</div><div class="value" id="statRemainAfterSave">-</div></div>
      </div>
    </div>
    <div class="card span-4">
      <h2>빠른 동작</h2>
      <div class="flex">
        <button class="btn" id="btnExportCSV">기록 CSV 내보내기</button>
        <button class="btn" id="btnExportMeta">메타 CSV 내보내기</button>
        <button class="btn-ghost" id="btnReset">전체 초기화</button>
      </div>
      <div class="hint" style="margin-top:8px;">브라우저 LocalStorage에 저장됩니다. 초기화 시 모든 데이터가 삭제됩니다.</div>
    </div>
  </header>

  <main class="grid">
    <!-- 설정 -->
    <section class="card span-6" id="settings">
      <h2>1) 기본 설정</h2>
      <div class="row">
        <div class="col-6">
          <label>월 예산 (원)</label>
          <input type="number" id="inputBudget" min="0" placeholder="예: 200000" />
        </div>
        <div class="col-6">
          <label>저축 목표 (원)</label>
          <input type="number" id="inputSaveGoal" min="0" placeholder="예: 50000" />
        </div>
        <div class="col-12">
          <label>카테고리 & 가중치 (1~5)</label>
          <div id="weightsBox"></div>
          <div class="flex" style="margin-top:8px;">
            <input type="text" id="newCatName" placeholder="새 카테고리 이름" style="max-width: 260px;" />
            <input type="number" id="newCatWeight" placeholder="가중치(1~5)" min="1" max="5" step="0.5" style="max-width: 160px;" />
            <button class="btn" id="btnAddCat">카테고리 추가</button>
          </div>
        </div>
        <div class="col-12" style="display:flex; gap:8px; margin-top:10px;">
          <button class="btn-primary" id="btnSaveMeta">설정 저장</button>
          <button class="btn" id="btnLoadDefaults">기본값 불러오기</button>
        </div>
      </div>
    </section>

    <!-- 기록 추가 -->
    <section class="card span-6" id="addRecord">
      <h2>2) 기록 추가</h2>
      <div class="row">
        <div class="col-4">
          <label>날짜</label>
          <input type="date" id="recDate" />
        </div>
        <div class="col-4">
          <label>카테고리</label>
          <select id="recCategory"></select>
        </div>
        <div class="col-4">
          <label>금액(원)</label>
          <input type="number" id="recAmount" min="0" />
        </div>
        <div class="col-12">
          <label>설명</label>
          <input type="text" id="recDesc" placeholder="예: 편의점 아이스크림" />
        </div>
        <div class="col-6">
          <label>만족도 (1~5)</label>
          <input type="number" id="recSat" min="1" max="5" step="0.1" />
        </div>
        <div class="col-6" style="display:flex; align-items:end;">
          <button class="btn-primary" id="btnAddRecord" style="width:100%">기록 저장</button>
        </div>
      </div>
      <div id="addMsg" class="hint" style="margin-top:8px;"></div>
    </section>

    <!-- 기록/잔액 보기 -->
    <section class="card span-12" id="records">
      <h2>3) 전체 기록 및 잔액</h2>
      <div class="flex" style="justify-content: space-between; align-items:center; margin-bottom:8px;">
        <div class="flex" style="gap:6px; align-items:center;">
          <span class="chip">정렬: 날짜 오름차순</span>
          <label class="pill">필터
            <select id="filterCategory" style="margin-left:6px; background:transparent; border:none; color:#e1ecff;">
              <option value="">전체</option>
            </select>
          </label>
        </div>
        <div class="muted" id="recCount">-건</div>
      </div>
      <div style="overflow:auto;">
        <table>
          <thead>
            <tr>
              <th style="min-width:120px;">날짜</th>
              <th style="min-width:90px;">카테고리</th>
              <th>설명</th>
              <th class="right" style="min-width:120px;">금액(원)</th>
              <th class="right" style="min-width:120px;">만족도</th>
            </tr>
          </thead>
          <tbody id="recordsTbody"></tbody>
        </table>
      </div>
    </section>

    <!-- 한계효용 곡선 -->
    <section class="card span-6" id="marginal">
      <h2>4) 한계효용 곡선</h2>
      <div class="row">
        <div class="col-8">
          <label>카테고리 선택</label>
          <select id="muCategory"></select>
        </div>
        <div class="col-4" style="display:flex; align-items:end;">
          <button class="btn" id="btnDrawMU" style="width:100%">그래프 업데이트</button>
        </div>
        <div class="col-12">
          <canvas id="muChart" height="170"></canvas>
          <div class="hint" id="muHint" style="margin-top:8px;">같은 카테고리에서 최소 2건 이상의 만족도 기록이 있어야 합니다.</div>
        </div>
      </div>
    </section>

    <!-- 예산 재배분 제안 -->
    <section class="card span-6" id="realloc">
      <h2>5) 예산 재배분 제안</h2>
      <div class="muted" style="margin-bottom:8px;">카테고리 가중치 비율 x 총 예산 = 추천 예산. 실제 사용과 비교하여 초과/여유를 표시합니다.</div>
      <div style="overflow:auto;">
        <table>
          <thead>
            <tr>
              <th>카테고리</th>
              <th class="right">추천(원)</th>
              <th class="right">사용(원)</th>
              <th class="right">차이(원)</th>
              <th>상태</th>
            </tr>
          </thead>
          <tbody id="reallocTbody"></tbody>
        </table>
      </div>
      <div class="hint" id="reallocMsg" style="margin-top:8px;"></div>
    </section>
  </main>

  <script>
    const STORAGE = {
      meta: 'allowance_meta_v2',
      records: 'allowance_records_v2'
    };

    // ====== 상태 ======
    let meta = null;  // { budget:number, saveGoal:number, categories:string[], weights:{[cat]:number} }
    let records = []; // [{date:"YYYY-MM-DD", category, description, amount:number, satisfaction:number}]

    // ====== 유틸 ======
    const won = (n) => (isNaN(n) ? '-' : Number(n).toLocaleString('ko-KR'));
    const clamp = (v, min, max) => Math.max(min, Math.min(max, v));

    function saveMeta() {
      localStorage.setItem(STORAGE.meta, JSON.stringify(meta));
    }
    function saveRecords() {
      localStorage.setItem(STORAGE.records, JSON.stringify(records));
    }
    function loadMeta() {
      const raw = localStorage.getItem(STORAGE.meta);
      if (!raw) return null;
      try { return JSON.parse(raw); } catch { return null; }
    }
    function loadRecords() {
      const raw = localStorage.getItem(STORAGE.records);
      if (!raw) return [];
      try { return JSON.parse(raw); } catch { return []; }
    }

    function loadDefaults() {
      meta = {
        budget: 200000,
        saveGoal: 50000,
        categories: ['식비','교통비','간식','문화','의류','기타'],
        weights: { '식비':5, '교통비':3, '간식':2, '문화':1, '의류':2, '기타':1 }
      };
      saveMeta();
      uiFromMeta();
      updateAll();
    }

    // ====== UI 바인딩 ======
    const inputBudget = document.getElementById('inputBudget');
    const inputSaveGoal = document.getElementById('inputSaveGoal');
    const weightsBox = document.getElementById('weightsBox');
    const recCategory = document.getElementById('recCategory');
    const filterCategory = document.getElementById('filterCategory');
    const muCategory = document.getElementById('muCategory');
    const addMsg = document.getElementById('addMsg');

    function uiFromMeta() {
      // 숫자 입력
      inputBudget.value = meta?.budget ?? '';
      inputSaveGoal.value = meta?.saveGoal ?? '';

      // 가중치 행 렌더
      weightsBox.innerHTML = '';
      (meta?.categories || []).forEach(cat => {
        const row = document.createElement('div');
        row.className = 'row';
        row.style.marginTop = '6px';
        row.innerHTML = `
          <div class="col-6"><input type="text" value="${cat}" data-cat-name /></div>
          <div class="col-4"><input type="number" min="1" max="5" step="0.1" value="${meta.weights[cat] ?? 1}" data-cat-weight /></div>
          <div class="col-2"><button class="btn-ghost" data-remove>삭제</button></div>
        `;
        row.querySelector('[data-remove]').addEventListener('click', () => {
          row.remove();
          // 저장은 사용자가 '설정 저장'을 눌렀을 때 적용
        });
        weightsBox.appendChild(row);
      });

      // 카테고리 선택 목록들
      function setOptions(el) {
        el.innerHTML = '';
        (meta?.categories || []).forEach(c => {
          const opt = document.createElement('option');
          opt.value = c; opt.textContent = c; el.appendChild(opt);
        });
      }
      setOptions(recCategory);
      // 필터는 '전체' 포함
      filterCategory.innerHTML = '<option value="">전체</option>';
      (meta?.categories || []).forEach(c => {
        const opt = document.createElement('option');
        opt.value = c; opt.textContent = c; filterCategory.appendChild(opt);
      });
      setOptions(muCategory);
    }

    function metaFromUI() {
      const cats = [];
      const weights = {};
      weightsBox.querySelectorAll('div.row').forEach(row => {
        const name = row.querySelector('[data-cat-name]').value.trim();
        const w = parseFloat(row.querySelector('[data-cat-weight]').value);
        if (name && !isNaN(w)) { cats.push(name); weights[name] = clamp(w,1,5); }
      });
      const budget = parseFloat(inputBudget.value);
      const saveGoal = parseFloat(inputSaveGoal.value);
      meta = { budget: isNaN(budget)?0:budget, saveGoal: isNaN(saveGoal)?0:saveGoal, categories: cats, weights };
      saveMeta();
      uiFromMeta();
      updateAll();
    }

    // ====== 통계/표시 업데이트 ======
    const statBudget = document.getElementById('statBudget');
    const statSpent = document.getElementById('statSpent');
    const statRemain = document.getElementById('statRemain');
    const statRemainAfterSave = document.getElementById('statRemainAfterSave');
    const recordsTbody = document.getElementById('recordsTbody');
    const recCount = document.getElementById('recCount');

    function computeSpentTotal(list = records) {
      return list.reduce((s, r) => s + (parseFloat(r.amount)||0), 0);
    }

    function computeSpentByCategory(list = records) {
      const map = {};
      list.forEach(r => { map[r.category] = (map[r.category]||0) + (parseFloat(r.amount)||0); });
      return map;
    }

    function updateHeaderStats() {
      const spent = computeSpentTotal();
      const rem = (meta?.budget||0) - spent;
      const remAfter = rem - (meta?.saveGoal||0);
      statBudget.textContent = won(meta?.budget||0) + '원';
      statSpent.textContent = won(spent) + '원';
      statRemain.textContent = won(rem) + '원';
      statRemainAfterSave.textContent = won(remAfter) + '원';
    }

    function renderRecords() {
      const filter = filterCategory.value;
      const sorted = [...records].sort((a,b)=> new Date(a.date) - new Date(b.date));
      const view = filter ? sorted.filter(r=>r.category===filter) : sorted;
      recordsTbody.innerHTML = '';
      view.forEach(r => {
        const tr = document.createElement('tr');
        tr.innerHTML = `
          <td>${r.date}</td>
          <td>${r.category}</td>
          <td>${r.description||''}</td>
          <td class="right">${won(r.amount)}원</td>
          <td class="right">${r.satisfaction}</td>
        `;
        recordsTbody.appendChild(tr);
      });
      recCount.textContent = `${view.length}건`;
    }

    // ====== 기록 추가 ======
    document.getElementById('btnAddRecord').addEventListener('click', () => {
      const date = document.getElementById('recDate').value || new Date().toISOString().slice(0,10);
      const category = document.getElementById('recCategory').value;
      const amount = parseFloat(document.getElementById('recAmount').value);
      const description = document.getElementById('recDesc').value.trim();
      let sat = parseFloat(document.getElementById('recSat').value);

      if (!category) { addMsg.textContent = '카테고리를 선택하세요.'; return; }
      if (isNaN(amount) || amount < 0) { addMsg.textContent = '금액을 올바르게 입력하세요.'; return; }
      if (isNaN(sat)) { addMsg.textContent = '만족도를 입력하세요.'; return; }
      sat = clamp(sat, 1, 5);

      records.push({ date, category, description, amount, satisfaction: sat });
      saveRecords();
      addMsg.textContent = '기록이 추가되었습니다.';
      // 입력 비우기
      document.getElementById('recAmount').value = '';
      document.getElementById('recDesc').value = '';
      document.getElementById('recSat').value = '';

      updateAll();
    });

    filterCategory.addEventListener('change', renderRecords);

    // ====== 한계효용 차트 ======
    let muChart = null;
    function drawMU() {
      const cat = muCategory.value;
      const subset = [...records]
        .filter(r => r.category === cat)
        .sort((a,b)=> new Date(a.date) - new Date(b.date));
      const sats = subset.map(r => parseFloat(r.satisfaction)).filter(v=>!isNaN(v));
      const marginals = [];
      for (let i=1;i<sats.length;i++) marginals.push(Number((sats[i]-sats[i-1]).toFixed(3)));

      const ctx = document.getElementById('muChart').getContext('2d');
      if (muChart) { muChart.destroy(); }

      if (marginals.length < 1) {
        document.getElementById('muHint').textContent = '충분한 기록이 없습니다. 동일 카테고리 만족도 2건 이상 필요.';
        muChart = new Chart(ctx, {
          type: 'line', data: { labels: [], datasets: [{ label: '한계효용', data: [] }] }, options: { plugins:{ legend:{ display:false } } }
        });
        return;
      }
      document.getElementById('muHint').textContent = '양수→만족 증가, 음수→한계효용 체감.';
      const labels = Array.from({length:marginals.length}, (_,i)=> String(i+2));
      muChart = new Chart(ctx, {
        type: 'line',
        data: {
          labels,
          datasets: [{ label: '한계효용(만족도 변화)', data: marginals, tension: 0.25, pointRadius: 4 }]
        },
        options: {
          scales: {
            x: { title: { display:true, text: '구매 횟수(2번째부터)' } },
            y: { title: { display:true, text: '만족도 증감' } }
          },
          plugins: { legend: { display: false } }
        }
      });
    }
    document.getElementById('btnDrawMU').addEventListener('click', drawMU);

    // ====== 예산 재배분 ======
    const reallocTbody = document.getElementById('reallocTbody');
    const reallocMsg = document.getElementById('reallocMsg');
    function renderReallocation() {
      const spentBy = computeSpentByCategory();
      const w = meta?.weights || {}; const cats = meta?.categories || [];
      const totalW = cats.reduce((s,c)=> s + (parseFloat(w[c])||0), 0);
      reallocTbody.innerHTML = '';
      let overMaxCat = null; let overMaxAmt = -Infinity;

      cats.forEach(cat => {
        const share = totalW>0 ? (w[cat] / totalW) : 0;
        const recAmt = (meta?.budget||0) * share;
        const used = spentBy[cat] || 0;
        const diff = used - recAmt;
        const over = diff > 0;
        if (diff > overMaxAmt) { overMaxAmt = diff; overMaxCat = cat; }
        const tr = document.createElement('tr');
        tr.innerHTML = `
          <td>${cat}</td>
          <td class="right">${won(recAmt)}원</td>
          <td class="right">${won(used)}원</td>
          <td class="right ${over? 'danger':'good'}">${over?'+':''}${won(diff)}원</td>
          <td>${over ? '<span class="status-over">초과</span>' : '<span class="status-ok">여유</span>'}</td>
        `;
        reallocTbody.appendChild(tr);
      });
      if (overMaxAmt > 0 && overMaxCat) {
        reallocMsg.innerHTML = `예산 초과: '<b>${overMaxCat}</b>'에서 <b>${won(overMaxAmt)}</b>원 감축을 권장합니다.`;
      } else {
        reallocMsg.textContent = '모든 카테고리가 추천 범위 내에 있습니다.';
      }
    }

    // ====== CSV 내보내기 ======
    function download(filename, text) {
      const a = document.createElement('a');
      a.setAttribute('href', 'data:text/plain;charset=utf-8,' + encodeURIComponent(text));
      a.setAttribute('download', filename);
      document.body.appendChild(a); a.click(); document.body.removeChild(a);
    }

    function exportRecordsCSV() {
      const header = 'Date,Category,Description,Amount,Satisfaction
';
      const lines = records.map(r => {
        const desc = '"' + (r.description||'').replace(/"/g,'""') + '"';
        return `${r.date},${r.category},${desc},${r.amount},${r.satisfaction}`;
      });
      download('allowance_records_v2.csv', header + lines.join('
'));
    }

    function exportMetaCSV() {
      // 파이썬 Series(meta, name='Value').to_csv 형태와 유사하게 생성
      const cats = (meta?.categories||[]).join(',');
      const weights = (meta?.categories||[]).map(c=>`${c}:${meta.weights[c]}`).join(';');
      const rows = [
        'index,Value',
        `Budget,${meta?.budget||0}`,
        `SaveGoal,${meta?.saveGoal||0}`,
        `"Categories","${cats.replace(/"/g,'""')}"`,
        `"Weights","${weights.replace(/"/g,'""')}"`
      ];
      download('allowance_meta.csv', rows.join('
'));
    }

    document.getElementById('btnExportCSV').addEventListener('click', exportRecordsCSV);
    document.getElementById('btnExportMeta').addEventListener('click', exportMetaCSV);

    // ====== 초기화 ======
    document.getElementById('btnReset').addEventListener('click', () => {
      if (!confirm('정말 모든 데이터를 초기화할까요? 되돌릴 수 없습니다.')) return;
      localStorage.removeItem(STORAGE.meta);
      localStorage.removeItem(STORAGE.records);
      meta = null; records = [];
      init();
    });

    // ====== 설정 이벤트 ======
    document.getElementById('btnSaveMeta').addEventListener('click', metaFromUI);
    document.getElementById('btnLoadDefaults').addEventListener('click', loadDefaults);
    document.getElementById('btnAddCat').addEventListener('click', () => {
      const name = document.getElementById('newCatName').value.trim();
      const w = parseFloat(document.getElementById('newCatWeight').value);
      if (!name) return alert('카테고리 이름을 입력하세요.');
      if (isNaN(w) || w < 1 || w > 5) return alert('가중치는 1~5 범위로 입력하세요.');
      // UI에 즉시 행 추가 (저장은 '설정 저장' 시 반영)
      const row = document.createElement('div');
      row.className = 'row';
      row.style.marginTop = '6px';
      row.innerHTML = `
        <div class="col-6"><input type="text" value="${name}" data-cat-name /></div>
        <div class="col-4"><input type="number" min="1" max="5" step="0.1" value="${w}" data-cat-weight /></div>
        <div class="col-2"><button class="btn-ghost" data-remove>삭제</button></div>
      `;
      row.querySelector('[data-remove]').addEventListener('click', () => row.remove());
      weightsBox.appendChild(row);
      document.getElementById('newCatName').value = '';
      document.getElementById('newCatWeight').value = '';
    });

    // ====== 전체 업데이트 ======
    function updateAll() {
      updateHeaderStats();
      renderRecords();
      renderReallocation();
      drawMU();
    }

    // ====== 초기 구동 ======
    function init() {
      meta = loadMeta();
      records = loadRecords();

      // 날짜 기본값 오늘
      const today = new Date().toISOString().slice(0,10);
      document.getElementById('recDate').value = today;

      if (!meta) {
        loadDefaults(); // 기본값으로 시작 (원한다면 비워두고 직접 입력 가능)
      } else {
        uiFromMeta();
        updateAll();
      }
    }

    init();
  </script>
</body>
</html>
