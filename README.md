<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>欧洲代理商地图</title>
  <!-- Leaflet CSS -->
  <link
    rel="stylesheet"
    href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"
    integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY="
    crossorigin=""
  />
  <style>
    :root{
      --bg:#0f172a;/* slate-900 */
      --panel:#111827ee;/* gray-900 */
      --muted:#94a3b8;/* slate-400 */
      --text:#e5e7eb;/* gray-200 */
      --accent:#22c55e;/* green-500 */
      --accent-2:#3b82f6;/* blue-500 */
      --danger:#ef4444;/* red-500 */
      --card:#0b1224;/* custom */
      --border:#1f2937;/* gray-800 */
    }
    html,body{height:100%;margin:0;background:var(--bg);color:var(--text);font-family:system-ui,-apple-system,Segoe UI,Roboto,Helvetica,Arial,"Apple Color Emoji","Segoe UI Emoji"}
    #app{display:grid;grid-template-columns:340px 1fr;height:100vh}
    .sidebar{background:linear-gradient(180deg, rgba(17,24,39,.9), rgba(15,23,42,.9));border-right:1px solid var(--border);display:flex;flex-direction:column}
    .header{padding:16px 16px 8px;border-bottom:1px solid var(--border)}
    .title{font-size:18px;font-weight:700;letter-spacing:.3px}
    .subtitle{font-size:12px;color:var(--muted)}
    .toolbar{display:flex;gap:8px;flex-wrap:wrap;margin-top:12px}
    button, .file-label{background:#0b1324;border:1px solid var(--border);color:var(--text);padding:8px 10px;border-radius:12px;font-size:12px;cursor:pointer;transition:.2s;display:inline-flex;align-items:center;gap:8px}
    button:hover,.file-label:hover{border-color:#334155}
    .danger{border-color:#3a0d0d;background:#1a0b0b;color:#fecaca}
    .danger:hover{border-color:#7f1d1d}
    .accent{border-color:#06331e;background:#061b12}
    .accent:hover{border-color:#14532d}
    .search{padding:10px 12px;margin:12px 16px;border-radius:12px;border:1px solid var(--border);background:#0b1324;color:var(--text);outline:0}
    .list{padding:8px 8px 120px;overflow:auto}
    .agent{background:var(--card);border:1px solid var(--border);border-radius:14px;padding:12px;margin:8px;display:flex;flex-direction:column;gap:6px}
    .agent h4{margin:0;font-size:14px}
    .muted{color:var(--muted);font-size:12px}
    .row{display:flex;gap:8px;flex-wrap:wrap}
    .pill{font-size:11px;padding:4px 8px;border:1px solid var(--border);border-radius:999px;color:var(--muted)}
    .agent-buttons{display:flex;gap:6px;margin-top:6px}
    #map{height:100vh;width:100%}
    .leaflet-container{background:#0b1224}
    .marker-label{font-weight:700;text-shadow:0 1px 2px #000}

    /* Modal */
    .modal{position:fixed;inset:0;background:rgba(0,0,0,.5);display:none;align-items:center;justify-content:center;padding:16px;z-index:9999}
    .modal.open{display:flex}
    .modal-card{width:min(520px,100%);background:#0b1224;border:1px solid var(--border);border-radius:16px;box-shadow:0 10px 30px rgba(0,0,0,.5)}
    .modal-head{padding:14px 16px;border-bottom:1px solid var(--border);display:flex;justify-content:space-between;align-items:center}
    .modal-body{padding:16px;display:grid;gap:12px}
    .modal-body label{font-size:12px;color:var(--muted)}
    .modal-body input,.modal-body textarea{width:100%;padding:10px 12px;border-radius:12px;border:1px solid var(--border);background:#0b1324;color:var(--text)}
    .modal-foot{padding:12px 16px;border-top:1px solid var(--border);display:flex;gap:8px;justify-content:flex-end}
    .coord{font-size:12px;color:var(--muted)}

    /* Footer help */
    .help{position:absolute;left:12px;bottom:12px;background:#0b1224cc;border:1px solid var(--border);padding:8px 12px;border-radius:12px;font-size:12px;color:var(--muted);backdrop-filter:blur(4px);z-index:500}

    @media (max-width: 900px){
      #app{grid-template-columns:1fr 1fr}
      .sidebar{position:absolute;z-index:800;width:320px;max-width:90vw;height:calc(100% - 24px);margin:12px;border-radius:16px;overflow:hidden}
    }
  </style>
</head>
<body>
<div id="app">
  <aside class="sidebar">
    <div class="header">
      <div class="title">欧洲代理商地图</div>
      <div class="subtitle">点击地图添加标记，输入代理商信息。数据自动保存在浏览器本地。</div>
      <div class="toolbar">
        <button class="accent" id="btn-export" title="导出为 JSON 文件">导出 JSON</button>
        <label class="file-label" for="file-import" title="从 JSON 文件导入">
          导入 JSON<input id="file-import" type="file" accept="application/json" style="display:none" />
        </label>
        <button id="btn-clear" class="danger" title="清空所有标记">清空全部</button>
      </div>
    </div>
    <input id="search" class="search" placeholder="搜索代理商名称/备注…" />
    <div id="list" class="list"></div>
  </aside>
  <div id="map"></div>
</div>

<div class="help">提示：单击地图新增标记；拖动标记可微调位置；点击列表可定位到标记。</div>

<!-- 创建/编辑代理商 Modal -->
<div id="modal" class="modal" role="dialog" aria-modal="true">
  <div class="modal-card">
    <div class="modal-head">
      <strong id="modal-title">新增代理商</strong>
      <button id="modal-close" title="关闭">✕</button>
    </div>
    <div class="modal-body">
      <div>
        <label for="agent-name">代理商名称</label>
        <input id="agent-name" placeholder="例如：ACME Cranes s.r.o." />
      </div>
      <div>
        <label for="agent-notes">备注（可选）</label>
        <textarea id="agent-notes" rows="3" placeholder="联系人、电话、业务范围等"></textarea>
      </div>
      <div class="coord">坐标：<span id="coord-text"></span></div>
    </div>
    <div class="modal-foot">
      <button id="modal-cancel">取消</button>
      <button id="modal-save" class="accent">保存</button>
    </div>
  </div>
</div>

<!-- Leaflet JS -->
<script
  src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"
  integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo="
  crossorigin=""
></script>
<script>
(function(){
  // --- State ---
  const state = {
    map: null,
    agents: [], // {id, name, notes, lat, lng}
    markers: new Map(), // id -> marker
    editingId: null,
    pendingLatLng: null,
  };

  // --- Helpers ---
  const $ = sel => document.querySelector(sel);
  const $$ = sel => Array.from(document.querySelectorAll(sel));
  const uid = () => Math.random().toString(36).slice(2) + Date.now().toString(36);
  const storageKey = 'eu-agents-v1';

  function saveToStorage(){
    localStorage.setItem(storageKey, JSON.stringify(state.agents));
  }
  function loadFromStorage(){
    try{
      const raw = localStorage.getItem(storageKey);
      if(raw){ state.agents = JSON.parse(raw); }
    }catch(e){ console.warn('读取本地存储失败', e); }
  }

  // --- Map Init ---
  function initMap(){
    state.map = L.map('map',{ zoomControl: true, worldCopyJump: true }).setView([54, 15], 4);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      maxZoom: 19,
      attribution: '&copy; OpenStreetMap contributors'
    }).addTo(state.map);

    state.map.on('click', (e)=>openCreateModal(e.latlng));
  }

  // --- Modal ---
  const modal = $('#modal');
  const modalTitle = $('#modal-title');
  const inputName = $('#agent-name');
  const inputNotes = $('#agent-notes');
  const coordText = $('#coord-text');

  function openCreateModal(latlng){
    state.editingId = null;
    state.pendingLatLng = latlng;
    modalTitle.textContent = '新增代理商';
    inputName.value = '';
    inputNotes.value = '';
    coordText.textContent = formatLatLng(latlng);
    modal.classList.add('open');
    setTimeout(()=>inputName.focus(), 50);
  }
  function openEditModal(agent){
    state.editingId = agent.id;
    state.pendingLatLng = {lat: agent.lat, lng: agent.lng};
    modalTitle.textContent = '编辑代理商';
    inputName.value = agent.name || '';
    inputNotes.value = agent.notes || '';
    coordText.textContent = formatLatLng(agent);
    modal.classList.add('open');
    setTimeout(()=>inputName.focus(), 50);
  }
  function closeModal(){ modal.classList.remove('open'); }

  $('#modal-close').onclick = closeModal;
  $('#modal-cancel').onclick = closeModal;
  $('#modal-save').onclick = () => {
    const name = inputName.value.trim();
    if(!name){ inputName.focus(); return; }
    const notes = inputNotes.value.trim();
    if(state.editingId){
      // update
      const idx = state.agents.findIndex(a=>a.id===state.editingId);
      if(idx>-1){
        const a = state.agents[idx];
        a.name = name; a.notes = notes;
        a.lat = state.pendingLatLng.lat; a.lng = state.pendingLatLng.lng;
        render(); saveToStorage();
      }
    }else{
      // create
      const agent = { id: uid(), name, notes, lat: state.pendingLatLng.lat, lng: state.pendingLatLng.lng };
      state.agents.push(agent);
      addMarker(agent);
      renderList(); saveToStorage();
    }
    closeModal();
  };

  function formatLatLng({lat,lng}){
    return lat.toFixed(5) + ', ' + lng.toFixed(5);
  }

  // --- Markers ---
  function addMarker(agent){
    const marker = L.marker([agent.lat, agent.lng], {draggable:true});
    marker.bindTooltip(`<span class="marker-label">${escapeHtml(agent.name)}</span>`, {permanent:true, direction:'top', offset:[0,-12]}).openTooltip();
    marker.bindPopup(`<b>${escapeHtml(agent.name)}</b><br>${escapeHtml(agent.notes||'') || '无备注'}<br/><small>${formatLatLng(agent)}</small>`);
    marker.on('dragend', ()=>{
      const {lat,lng} = marker.getLatLng();
      agent.lat = lat; agent.lng = lng;
      marker.setPopupContent(`<b>${escapeHtml(agent.name)}</b><br>${escapeHtml(agent.notes||'') || '无备注'}<br/><small>${formatLatLng(agent)}</small>`);
      saveToStorage();
      renderList();
    });
    marker.on('dblclick', ()=> openEditModal(agent));
    marker.addTo(state.map);
    state.markers.set(agent.id, marker);
  }

  function clearMarkers(){
    for(const m of state.markers.values()) m.remove();
    state.markers.clear();
  }

  // --- List ---
  const listEl = $('#list');
  function renderList(filter=''){
    const q = filter.trim().toLowerCase();
    listEl.innerHTML = '';
    const frag = document.createDocumentFragment();
    state.agents
      .filter(a => !q || a.name.toLowerCase().includes(q) || (a.notes||'').toLowerCase().includes(q))
      .sort((a,b)=>a.name.localeCompare(b.name))
      .forEach(agent=>{
        const card = document.createElement('div');
        card.className = 'agent';
        card.innerHTML = `
          <h4>${escapeHtml(agent.name)}</h4>
          <div class="muted">${escapeHtml(agent.notes||'') || '—'}</div>
          <div class="row">
            <span class="pill">${agent.lat.toFixed(4)}, ${agent.lng.toFixed(4)}</span>
          </div>
          <div class="agent-buttons">
            <button data-act="locate">定位</button>
            <button data-act="edit">编辑</button>
            <button data-act="delete" class="danger">删除</button>
          </div>
        `;
        card.querySelector('[data-act="locate"]').onclick = ()=>{
          state.map.flyTo([agent.lat, agent.lng], Math.max(state.map.getZoom(), 8));
          const mk = state.markers.get(agent.id); if(mk){ mk.openPopup(); }
        };
        card.querySelector('[data-act="edit"]').onclick = ()=> openEditModal(agent);
        card.querySelector('[data-act="delete"]').onclick = ()=> deleteAgent(agent.id);
        frag.appendChild(card);
      });
    listEl.appendChild(frag);
  }

  function deleteAgent(id){
    const idx = state.agents.findIndex(a=>a.id===id);
    if(idx>-1){
      const [removed] = state.agents.splice(idx,1);
      const m = state.markers.get(removed.id); if(m){ m.remove(); state.markers.delete(removed.id); }
      saveToStorage(); renderList($('#search').value);
    }
  }

  // --- Export / Import ---
  $('#btn-export').onclick = ()=>{
    const data = JSON.stringify({
      meta: { app: 'EU Agent Map', version: 1, exportedAt: new Date().toISOString() },
      agents: state.agents
    }, null, 2);
    const blob = new Blob([data], {type:'application/json'});
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url; a.download = 'eu-agents.json'; a.click();
    URL.revokeObjectURL(url);
  };

  $('#file-import').addEventListener('change', (e)=>{
    const file = e.target.files[0]; if(!file) return;
    const reader = new FileReader();
    reader.onload = () => {
      try{
        const obj = JSON.parse(reader.result);
        if(Array.isArray(obj)){
          state.agents = obj;
        }else if(obj && Array.isArray(obj.agents)){
          state.agents = obj.agents;
        }else{
          alert('文件格式不正确'); return;
        }
        saveToStorage();
        // re-render all
        clearMarkers();
        state.agents.forEach(addMarker);
        renderList($('#search').value);
      }catch(err){ alert('导入失败：' + err.message); }
    };
    reader.readAsText(file);
    e.target.value = '';
  });

  $('#btn-clear').onclick = ()=>{
    if(confirm('确定要清空所有标记吗？该操作不可撤销。')){
      state.agents = []; saveToStorage(); clearMarkers(); renderList('');
    }
  };

  // --- Search ---
  $('#search').addEventListener('input', (e)=>{
    renderList(e.target.value);
  });

  // Utility
  function escapeHtml(str){
    return String(str||'').replace(/[&<>"]/g, s=>({"&":"&amp;","<":"&lt;",">":"&gt;","\"":"&quot;"}[s]));
  }

  // --- Boot ---
  initMap();
  loadFromStorage();
  // draw markers from storage
  state.agents.forEach(addMarker);
  renderList('');
})();
</script>
</body>
</html>
