<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Gestión de Inventario — Autopartes</title>
  <style>
    :root{
      --bg:#f6f7f9; --card:#ffffff; --text:#0f172a; --muted:#6b7280; --border:#e5e7eb;
      --primary:#1f6feb; --danger:#ef4444; --radius:12px;
      --shadow:0 1px 2px rgba(0,0,0,.04),0 6px 24px rgba(0,0,0,.06);
    }
    *{box-sizing:border-box}
    html,body{height:100%}
    body{margin:0; font-family:system-ui,-apple-system,Segoe UI,Roboto,Ubuntu,Cantarell,Noto Sans,sans-serif; color:var(--text); background:var(--bg);}
    .app{display:grid; grid-template-columns:260px 1fr; min-height:100vh;}
    .app.no-sidebar{ grid-template-columns:1fr; }
    .sidebar{border-right:1px solid var(--border); background:#fafafa; padding:16px; display:flex; flex-direction:column; gap:16px;}
    .hidden{display:none !important}
    .brand{display:flex; align-items:center; gap:12px; padding:8px 6px}
    .brand .logo{font-size:28px}
    .nav{display:flex; flex-direction:column; gap:6px}
    .nav-btn{ text-align:left; border:1px solid var(--border); background:#fff; padding:10px 12px; border-radius:10px; cursor:pointer; }
    .nav-btn:hover{border-color:#d1d5db}
    .sidebar-foot{ margin-top:auto; display:flex; flex-direction:column; gap:8px; }
    .user-label{font-size:12px; color:var(--muted)}
    .view{padding:24px; max-width:1200px; width:100%}
    .center{display:grid; place-items:center; min-height:100vh}
    .card{background:var(--card); border:1px solid var(--border); border-radius:var(--radius); box-shadow:var(--shadow); padding:16px;}
    .card-max{width:100%; max-width:460px}
    .title{margin:0 0 8px}
    .muted{color:var(--muted)}
    .xs{font-size:12px}
    .kpis{display:grid; grid-template-columns:repeat(4,minmax(0,1fr)); gap:12px}
    .kpis .card h3{margin:0 0 6px; font-size:14px; color:var(--muted)}
    .kpis .card strong{font-size:20px}
    .form{display:flex; flex-direction:column; gap:12px}
    .field{display:flex; flex-direction:column; gap:6px}
    .input, .field input, .field select{ border:1px solid var(--border); border-radius:10px; padding:10px 12px; background:#fff; }
    .input:focus, .field input:focus, .field select:focus{outline:2px solid rgba(31,111,235,.25)}
    .btn{ border:1px solid var(--border); background:#fff; border-radius:10px; padding:10px 14px; cursor:pointer; }
    .btn-primary{background:var(--primary); color:#fff; border-color:transparent}
    .btn-ghost{background:transparent}
    .btn-danger{background:var(--danger); color:#fff; border-color:transparent}
    .toolbar{ display:flex; align-items:center; justify-content:space-between; gap:12px; margin-bottom:12px; }
    .toolbar .right{display:flex; gap:8px; flex-wrap:wrap}
    .stack{display:flex; flex-direction:column; gap:12px}
    .table-wrap{width:100%; overflow:auto}
    .table{width:100%; border-collapse:separate; border-spacing:0}
    .table th,.table td{padding:10px 12px; border-bottom:1px solid var(--border)}
    .table th{font-weight:600; text-align:left; white-space:nowrap}
    .table td.num,.table th.num{text-align:right}
    .badge{display:inline-block; padding:2px 8px; border-radius:999px; font-size:12px; border:1px solid var(--border);}
    .badge.low{background:#fff5f5; color:#b91c1c; border-color:#fecaca}
    .row-actions{display:flex; gap:8px; align-items:center}
    .grid{ display:grid; grid-template-columns:repeat(4, minmax(0, 1fr)); gap:12px; }
    .grid .field{min-width:0}
    @media (max-width:1100px){ .kpis{grid-template-columns:repeat(2,minmax(0,1fr));} }
    @media (max-width:900px){ .app{grid-template-columns:1fr} .sidebar{position:sticky; top:0; z-index:10} .grid{grid-template-columns:repeat(2, minmax(0,1fr))} }
    @media (max-width:600px){ .grid{grid-template-columns:1fr} .kpis{grid-template-columns:1fr;} }
    #cam{width:1px;height:1px;position:absolute;left:-9999px;top:-9999px}
  </style>
</head>
<body>
  <div id="app" class="app no-sidebar">
    <!-- Sidebar (aparece solo tras login) -->
    <aside id="sidebar" class="sidebar hidden" aria-hidden="true">
      <div class="brand">
        <div class="logo" aria-hidden="true">🔧</div>
        <div><strong>Autopartes</strong><small>Inventario</small></div>
      </div>
      <nav class="nav" aria-label="Navegación principal">
        <button class="nav-btn" data-route="dashboard">Dashboard</button>
        <button class="nav-btn" data-route="inventario">Inventario</button>
        <button class="nav-btn" data-route="movimientos">Movimientos</button>
        <button class="nav-btn" data-route="sugerencias">Sugerencias</button>
        <button class="nav-btn" data-route="historial">Historial precios</button>
        <button class="nav-btn" data-route="ajustes">Ajustes</button>
      </nav>
      <div class="sidebar-foot">
        <span id="userLabel" class="user-label"></span>
        <button id="logoutBtn" class="btn btn-ghost">Cerrar sesión</button>
      </div>
    </aside>
    <main id="view" class="view"></main>
  </div>

  <!-- Templates -->
  <template id="tpl-login">
    <section class="center">
      <div class="card card-max" role="dialog" aria-labelledby="loginTitle">
        <h1 id="loginTitle" class="title">Iniciar sesión</h1>
        <p class="muted">Acceso interno — Gestión de inventario de autopartes.</p>
        <form id="loginForm" class="form">
          <label class="field"><span>Correo</span>
            <input type="email" name="email" placeholder="admin@empresa.com" required autocomplete="username" />
          </label>
          <label class="field"><span>Contraseña</span>
            <input type="password" name="password" placeholder="123456" required autocomplete="current-password" />
          </label>
          <button class="btn btn-primary" type="submit">Entrar</button>
          <p class="muted xs">Demo: <code>admin@empresa.com</code> / <code>123456</code></p>
          <button id="hardResetBtn" type="button" class="btn" title="Use si la app no carga o el usuario no aparece">Reparar datos (reset demo)</button>
        </form>
      </div>
    </section>
  </template>

  <template id="tpl-inventario">
    <section class="stack">
      <header class="toolbar">
        <div class="left"><h2>Inventario</h2><p class="muted">Piezas registradas y estado de stock.</p></div>
        <div class="right">
          <select id="filterAlmacen" class="input" title="Filtrar por almacén"></select>
          <input id="searchParts" class="input" placeholder="Buscar por código, nombre o categoría…" />
          <label class="btn" for="importCSV">Importar CSV</label>
          <input id="importCSV" type="file" accept=".csv" style="display:none" />
          <button id="addPartBtn" class="btn btn-primary">Añadir pieza</button>
        </div>
      </header>
      <details id="partFormWrap" class="card">
        <summary id="partFormSummary">Nueva pieza</summary>
        <form id="partForm" class="grid">
          <input type="hidden" name="id" />
          <label class="field"><span>Código</span>
            <div style="display:flex; gap:8px">
              <input name="codigo" required placeholder="ABC-123" style="flex:1" />
              <button type="button" id="scanBtn" class="btn">Escanear</button>
            </div>
            <video id="cam" playsinline></video>
          </label>
          <label class="field"><span>Nombre</span><input name="nombre" required placeholder="Filtro de aceite" /></label>
          <label class="field"><span>Categoría</span><input name="categoria" placeholder="Motor, Suspensión, Frenos…" /></label>
          <label class="field"><span>Ubicación</span><input name="ubicacion" placeholder="Estante A3" /></label>
          <label class="field"><span>Almacén</span><input name="almacen" placeholder="Central, Sucursal 1…" /></label>
          <label class="field"><span>Proveedor</span><input name="proveedor" placeholder="Proveedor principal" /></label>
          <label class="field"><span>Lead time (días)</span><input name="leadTimeDias" type="number" min="0" step="1" value="0" /></label>
          <label class="field"><span>Consumo diario</span><input name="consumoDiario" type="number" min="0" step="1" value="0" /></label>
          <label class="field"><span>Stock</span><input name="stock" type="number" min="0" step="1" value="0" required /></label>
          <label class="field"><span>Stock mínimo</span><input name="stockMin" type="number" min="0" step="1" value="0" required /></label>
          <label class="field"><span>Costo unitario (Bs)</span><input name="costo" type="number" min="0" step="0.01" value="0" /></label>
          <label class="field"><span>Precio venta (Bs)</span><input name="precio" type="number" min="0" step="0.01" value="0" /></label>
          <div class="row-actions"><button class="btn btn-primary" type="submit">Guardar</button><button type="button" id="cancelEdit" class="btn btn-ghost">Cancelar</button></div>
        </form>
      </details>
      <div class="card">
        <div class="table-wrap">
          <table class="table" id="partsTable">
            <thead><tr>
              <th>Código</th><th>Nombre</th><th>Categoría</th><th>Almacén</th><th>Ubicación</th>
              <th class="num">Stock</th><th class="num">Mín</th><th class="num">Costo (Bs)</th><th class="num">Precio (Bs)</th><th></th>
            </tr></thead>
            <tbody id="partsTbody"></tbody>
          </table>
        </div>
      </div>
    </section>
  </template>

  <template id="tpl-movimientos">
    <section class="stack">
      <header class="toolbar">
        <div class="left"><h2>Movimientos</h2><p class="muted">Entradas y salidas que afectan el stock.</p></div>
        <div class="right">
          <select id="moveAlmacenFilter" class="input" title="Filtrar piezas por almacén"></select>
          <button id="undoLast" class="btn">Deshacer último</button>
          <button id="exportCSV" class="btn">Exportar inventario (CSV)</button>
        </div>
      </header>
      <div class="card">
        <h3>Registrar movimiento</h3>
        <form id="moveForm" class="grid">
          <label class="field"><span>Pieza</span><select name="piezaId" id="movePart" required></select></label>
          <label class="field"><span>Tipo</span><select name="tipo" required><option value="entrada">Entrada</option><option value="salida">Salida</option></select></label>
          <label class="field"><span>Cantidad</span><input name="cantidad" type="number" min="1" step="1" required /></label>
          <label class="field"><span>Referencia</span><input name="referencia" placeholder="Factura, orden, motivo…" /></label>
          <div class="row-actions"><button type="submit" class="btn btn-primary">Aplicar</button></div>
        </form>
      </div>
      <div class="card">
        <div class="table-wrap">
          <table class="table" id="movesTable">
            <thead><tr>
              <th>Fecha</th><th>Código</th><th>Nombre</th><th>Almacén</th><th>Tipo</th>
              <th class="num">Cantidad</th><th>Referencia</th><th>Usuario</th>
            </tr></thead>
            <tbody id="movesTbody"></tbody>
          </table>
        </div>
      </div>
    </section>
  </template>

  <template id="tpl-sugerencias">
    <section class="stack">
      <header class="toolbar">
        <div class="left"><h2>Sugerencias de compra</h2><p class="muted">Basado en consumo, lead time y stock mínimo.</p></div>
        <div class="right"><select id="sugAlmacenFilter" class="input" title="Filtrar por almacén"></select></div>
      </header>
      <div class="card">
        <div class="table-wrap">
          <table class="table" id="sugTable">
            <thead><tr>
              <th>Código</th><th>Nombre</th><th>Proveedor</th><th>Almacén</th>
              <th class="num">Stock</th><th class="num">Mín</th><th class="num">CD</th><th class="num">LT</th>
              <th class="num">Punto pedido</th><th class="num">Sugerido</th>
            </tr></thead>
            <tbody id="sugTbody"></tbody>
          </table>
        </div>
      </div>
    </section>
  </template>

  <template id="tpl-ajustes">
    <section class="stack">
      <header class="toolbar">
        <div class="left"><h2>Ajustes</h2><p class="muted">Backup y restauración.</p></div>
        <div class="right">
          <button id="backupBtn" class="btn">Backup JSON</button>
          <label class="btn" for="restoreFile">Restaurar JSON</label>
          <input id="restoreFile" type="file" accept="application/json" style="display:none"/>
          <button id="resetBtn" class="btn btn-danger">Resetear demo</button>
        </div>
      </header>
      <div class="card"><p class="muted">Usuario único con acceso total.</p></div>
    </section>
  </template>

  <template id="tpl-historial">
    <section class="stack">
      <header class="toolbar">
        <div class="left"><h2>Historial de precios</h2><p class="muted">Cambios de costo y precio por pieza.</p></div>
        <div class="right"></div>
      </header>
      <div class="card">
        <div class="table-wrap">
          <table class="table" id="histTable">
            <thead><tr>
              <th>Fecha</th><th>Código</th><th>Nombre</th>
              <th class="num">Costo (antes ➝ después)</th><th class="num">Precio (antes ➝ después)</th><th>Usuario</th>
            </tr></thead>
            <tbody id="histTbody"></tbody>
          </table>
        </div>
      </div>
    </section>
  </template>

  <script>
    /* ===== Helpers de almacenamiento seguro ===== */
    function safeRead(key, fallback){
      try{
        const raw = localStorage.getItem(key);
        if(raw === null || raw === undefined || raw === '') return fallback;
        return JSON.parse(raw);
      }catch(e){
        console.warn('Datos corruptos en', key, '— usando fallback.');
        return fallback;
      }
    }
    function safeWrite(key, value){
      try{ localStorage.setItem(key, JSON.stringify(value)); }
      catch(e){ console.error('No se pudo guardar', key, e); }
    }

    // ========= data.js =========
    const DB = (() => {
      const KEYS = { USERS:'users', PARTS:'parts', MOVES:'movements', SEEDED:'seededFlag', PRICEH:'priceHistory' };
      const rid = () => 'id_' + Date.now().toString(36) + Math.random().toString(36).slice(2,8);

      function seedIfEmpty(){
        // Si no hay seed, sembrar demo completa
        if(!localStorage.getItem(KEYS.SEEDED)){
          const users = [{email:'admin@empresa.com', name:'Admin', password:'123456'}];
          const parts = [
            {id:rid(), codigo:'FILT-ACE-001', nombre:'Filtro de aceite', categoria:'Motor', ubicacion:'A3', almacen:'Central', proveedor:'Genérico',  leadTimeDias:5, consumoDiario:2, stock:18, stockMin:5, costo:20.5,  precio:35},
            {id:rid(), codigo:'PAST-FRE-002', nombre:'Pastillas de freno', categoria:'Frenos', ubicacion:'B1', almacen:'Central', proveedor:'Fren-Tech', leadTimeDias:3, consumoDiario:3, stock:6,  stockMin:8, costo:15,   precio:28},
            {id:rid(), codigo:'AMOR-DEL-003', nombre:'Amortiguador delantero', categoria:'Suspensión', ubicacion:'C2', almacen:'Sucursal 1', proveedor:'Suspenso', leadTimeDias:7, consumoDiario:1, stock:3,  stockMin:2, costo:120, precio:180},
          ];
          safeWrite(KEYS.USERS, users);
          safeWrite(KEYS.PARTS, parts);
          safeWrite(KEYS.MOVES, []);
          safeWrite(KEYS.PRICEH, []);
          localStorage.setItem(KEYS.SEEDED, '1');
        }else{
          // Asegurar que exista el usuario demo aunque haya seed viejo
          const users = safeRead(KEYS.USERS, []);
          if(!users.find(u => (u.email||'').toLowerCase() === 'admin@empresa.com')){
            users.push({email:'admin@empresa.com', name:'Admin', password:'123456'});
            safeWrite(KEYS.USERS, users);
          }
          // Asegurar arrays válidos
          if(!Array.isArray(safeRead(KEYS.PARTS, []))) safeWrite(KEYS.PARTS, []);
          if(!Array.isArray(safeRead(KEYS.MOVES, []))) safeWrite(KEYS.MOVES, []);
          if(!Array.isArray(safeRead(KEYS.PRICEH, []))) safeWrite(KEYS.PRICEH, []);
        }
      }

      const getUsers = () => safeRead(KEYS.USERS, []);
      const listParts = () => safeRead(KEYS.PARTS, []);
      const saveParts = (parts) => safeWrite(KEYS.PARTS, parts);
      function upsertPart(part){
        const arr = listParts();
        if(part.id){
          const i = arr.findIndex(p => p.id === part.id);
          if(i >= 0) arr[i] = part; else arr.push(part);
        } else { part.id = rid(); arr.push(part); }
        saveParts(arr); return part;
      }
      const deletePart = (id) => saveParts(listParts().filter(p => p.id !== id));
      const listMoves = () => safeRead(KEYS.MOVES, []);
      const saveMoves = (moves) => safeWrite(KEYS.MOVES, moves);
      function addMove(move){ const m=listMoves(); m.unshift(move); saveMoves(m); }
      function applyMovement({piezaId, tipo, cantidad, referencia, usuarioEmail}){
        const parts = listParts();
        const p = parts.find(x => x.id === piezaId);
        if(!p) throw new Error('Pieza no encontrada');
        cantidad = Number(cantidad);
        if(tipo === 'salida'){ if(p.stock < cantidad) throw new Error('Stock insuficiente para la salida'); p.stock -= cantidad; }
        else { p.stock += cantidad; }
        saveParts(parts);
        addMove({ id:'mov_'+Date.now(), fecha:new Date().toISOString(), piezaId, codigo:p.codigo, nombre:p.nombre, almacen:p.almacen||'', tipo, cantidad, referencia: referencia || '', usuario: usuarioEmail || '' });
      }
      const listPriceHistory = () => safeRead(KEYS.PRICEH, []);
      function addPriceHistory(entry){ const arr = listPriceHistory(); arr.unshift(entry); safeWrite(KEYS.PRICEH, arr); }

      function hardReset(){
        localStorage.removeItem(KEYS.USERS);
        localStorage.removeItem(KEYS.PARTS);
        localStorage.removeItem(KEYS.MOVES);
        localStorage.removeItem(KEYS.PRICEH);
        localStorage.removeItem(KEYS.SEEDED);
        seedIfEmpty();
      }

      return { seedIfEmpty, getUsers, listParts, saveParts, upsertPart, deletePart, listMoves, saveMoves, addMove, applyMovement, listPriceHistory, addPriceHistory, hardReset };
    })();

    // ========= auth.js =========
    const Auth = (() => {
      const isAuthed = () => !!localStorage.getItem('authedUser');
      const currentUser = () => safeRead('authedUser', null);
      const logout = () => localStorage.removeItem('authedUser');
      function login(email, password){
        const users = DB.getUsers();
        const user = users.find(u => (u.email||'').toLowerCase() === String(email||'').toLowerCase());
        if(!user) return {ok:false, error:'Usuario no encontrado'};
        if(String(user.password) !== String(password)) return {ok:false, error:'Contraseña incorrecta'};
        safeWrite('authedUser', {email:user.email, name:user.name});
        return {ok:true};
      }
      return { isAuthed, currentUser, logout, login };
    })();

    // ========= app.js =========
    const app = document.getElementById('app');
    const view = document.getElementById('view');
    const sidebar = document.getElementById('sidebar');
    const userLabel = document.getElementById('userLabel');
    const logoutBtn = document.getElementById('logoutBtn');

    const PUBLIC_ROUTES = ['login'];

    init();

    function init(){
      DB.seedIfEmpty();
      if(logoutBtn){
        logoutBtn.addEventListener('click', () => { Auth.logout(); location.hash = '#/login'; route(); });
      }
      if(!Auth.isAuthed()) location.hash = '#/login';
      route();
      window.addEventListener('hashchange', route);
    }

    function ensureAuth(path){
      const authed = Auth.isAuthed();
      if(!authed && !PUBLIC_ROUTES.includes(path)){
        if(location.hash !== '#/login') location.hash = '#/login';
        return false;
      }
      return true;
    }

    function setNavActive(path){
      document.querySelectorAll('.nav-btn').forEach(btn=>{
        btn.classList.toggle('btn-primary', btn.dataset.route === path);
        btn.onclick = ()=>{ location.hash = '#/' + btn.dataset.route; };
      });
    }

    function route(){
      const hash = location.hash || '#/login';
      const path = (hash.startsWith('#/')) ? hash.slice(2) : 'login';
      const authed = Auth.isAuthed();

      // Layout/Sidebar
      if(authed){
        sidebar.classList.remove('hidden'); sidebar.setAttribute('aria-hidden','false'); app.classList.remove('no-sidebar');
        const u = Auth.currentUser(); userLabel.textContent = u ? `Conectado como: ${u.email}` : '';
      }else{
        sidebar.classList.add('hidden'); sidebar.setAttribute('aria-hidden','true'); app.classList.add('no-sidebar');
      }

      if(!ensureAuth(path)){ renderLogin(); return; }

      switch(path){
        case 'login': renderLogin(); break;
        case 'dashboard': renderDashboard(); break;
        case 'inventario': renderInventario(); break;
        case 'movimientos': renderMovimientos(); break;
        case 'sugerencias': renderSugerencias(); break;
        case 'ajustes': renderAjustes(); break;
        case 'historial': renderHistorial(); break;
        default: location.hash = authed ? '#/dashboard' : '#/login'; return;
      }
      if(authed) setNavActive(path);
    }

    /* --------- Vistas --------- */
    function renderLogin(){
      const tpl = document.getElementById('tpl-login').content.cloneNode(true);
      view.innerHTML = ''; view.appendChild(tpl);

      const form = document.getElementById('loginForm');
      const hardBtn = document.getElementById('hardResetBtn');

      form.addEventListener('submit', (e)=>{
        e.preventDefault();
        const fd = new FormData(form);
        const email = fd.get('email'); const password = fd.get('password');
        const res = Auth.login(email, password);
        if(!res.ok){ alert(res.error || 'Error al iniciar sesión'); return; }
        location.hash = '#/dashboard'; route();
      });

      hardBtn.addEventListener('click', ()=>{
        if(confirm('Esto reseteará los datos locales (demo). ¿Continuar?')){
          DB.hardReset();
          alert('Datos reparados. Intente iniciar sesión nuevamente: admin@empresa.com / 123456');
          location.hash = '#/login'; route();
        }
      });
    }

    function renderDashboard(){
      const parts = DB.listParts();
      const moves = DB.listMoves();
      const valor = parts.reduce((s,p)=> s + Number(p.costo||0)*Number(p.stock||0), 0);
      const bajos = parts.filter(p => Number(p.stock)<Number(p.stockMin)).length;
      const hoy = new Date().toDateString();
      const movHoy = moves.filter(m => new Date(m.fecha).toDateString()===hoy).length;

      view.innerHTML = `
        <section class="stack">
          <div class="kpis">
            <div class="card"><h3>Valor inventario</h3><strong>${fmtMoney(valor)}</strong></div>
            <div class="card"><h3>Ítems bajo mínimo</h3><strong>${bajos}</strong></div>
            <div class="card"><h3>Total SKUs</h3><strong>${parts.length}</strong></div>
            <div class="card"><h3>Movimientos hoy</h3><strong>${movHoy}</strong></div>
          </div>
          <div class="card"><p class="muted">Use el menú para navegar. Revise <strong>Sugerencias</strong> para reposición y <strong>Historial precios</strong> para auditoría.</p></div>
        </section>`;
    }

    function renderInventario(){
      const tpl = document.getElementById('tpl-inventario').content.cloneNode(true);
      view.innerHTML = ''; view.appendChild(tpl);

      const tbody = document.getElementById('partsTbody');
      const search = document.getElementById('searchParts');
      const addBtn = document.getElementById('addPartBtn');
      const formWrap = document.getElementById('partFormWrap');
      const form = document.getElementById('partForm');
      const summary = document.getElementById('partFormSummary');
      const cancelBtn = document.getElementById('cancelEdit');
      const importCSV = document.getElementById('importCSV');
      const filterAlm = document.getElementById('filterAlmacen');

      let parts = DB.listParts();

      function buildAlmacenOptions(){
        const set = new Set(['(Todos)']); parts.forEach(p=>{ if(p.almacen && p.almacen.trim()) set.add(p.almacen.trim()); });
        filterAlm.innerHTML = Array.from(set).map(a=>`<option value="${escapeHtml(a)}">${escapeHtml(a)}</option>`).join('');
      }
      buildAlmacenOptions();

      function matchesFilters(p){
        const q = (search.value||'').toLowerCase().trim();
        const alm = filterAlm.value || '(Todos)';
        const tokens = [p.codigo, p.nombre, p.categoria, p.proveedor, p.almacen].map(x=>String(x||'').toLowerCase());
        const passQ = !q || tokens.some(t=>t.includes(q));
        const passA = alm === '(Todos)' || String(p.almacen||'').toLowerCase() === alm.toLowerCase();
        return passQ && passA;
      }

      function paint(list){
        tbody.innerHTML = '';
        list.filter(matchesFilters).forEach(p=>{
          const low = Number(p.stock) < Number(p.stockMin);
          const tr = document.createElement('tr');
          tr.innerHTML = `
            <td>${escapeHtml(p.codigo)}</td>
            <td>${escapeHtml(p.nombre)} ${low ? '<span class="badge low">Bajo stock</span>' : ''}</td>
            <td>${escapeHtml(p.categoria||'')}</td>
            <td>${escapeHtml(p.almacen||'')}</td>
            <td>${escapeHtml(p.ubicacion||'')}</td>
            <td class="num">${p.stock}</td>
            <td class="num">${p.stockMin}</td>
            <td class="num">${fmtMoney(p.costo)}</td>
            <td class="num">${fmtMoney(p.precio)}</td>
            <td class="num">
              <button class="btn btn-ghost" data-act="edit" data-id="${p.id}">Editar</button>
              <button class="btn btn-danger" data-act="del" data-id="${p.id}">Eliminar</button>
            </td>`;
          tbody.appendChild(tr);
        });
      }

      function resetForm(){ form.reset(); form.querySelector('input[name="id"]').value = ''; summary.textContent = 'Nueva pieza'; formWrap.open = false; }

      paint(parts);
      search.addEventListener('input', ()=> paint(parts));
      filterAlm.addEventListener('change', ()=> paint(parts));
      addBtn.addEventListener('click', ()=>{ resetForm(); formWrap.open = true; });
      cancelBtn.addEventListener('click', resetForm);
      document.getElementById('scanBtn').addEventListener('click', startScan);

      form.addEventListener('submit', (e)=>{
        e.preventDefault();
        const fd = new FormData(form);
        const part = {
          id: fd.get('id') || undefined,
          codigo: String(fd.get('codigo')).trim(),
          nombre: String(fd.get('nombre')).trim(),
          categoria: String(fd.get('categoria')||'').trim(),
          ubicacion: String(fd.get('ubicacion')||'').trim(),
          almacen: String(fd.get('almacen')||'').trim(),
          proveedor: String(fd.get('proveedor')||'').trim(),
          leadTimeDias: Number(fd.get('leadTimeDias')||0),
          consumoDiario: Number(fd.get('consumoDiario')||0),
          stock: Number(fd.get('stock')||0),
          stockMin: Number(fd.get('stockMin')||0),
          costo: Number(fd.get('costo')||0),
          precio: Number(fd.get('precio')||0),
        };
        if(!part.codigo || !part.nombre){ alert('Código y nombre son obligatorios'); return; }

        let prev = null; if(part.id){ prev = parts.find(x=>x.id===part.id); }
        DB.upsertPart(part);

        if(prev && (Number(prev.costo)!==Number(part.costo) || Number(prev.precio)!==Number(part.precio))){
          const u = Auth.currentUser();
          DB.addPriceHistory({
            id:'ph_'+Date.now(), fecha:new Date().toISOString(), piezaId: part.id,
            codigo: prev.codigo, nombre: prev.nombre,
            costoAntes: Number(prev.costo)||0, costoDespues: Number(part.costo)||0,
            precioAntes: Number(prev.precio)||0, precioDespues: Number(part.precio)||0,
            usuario: u ? u.email : ''
          });
        }

        parts = DB.listParts(); buildAlmacenOptions(); paint(parts); resetForm();
      });

      tbody.addEventListener('click', (e)=>{
        const btn = e.target.closest('button[data-act]'); if(!btn) return;
        const id = btn.dataset.id; const act = btn.dataset.act;
        if(act === 'edit'){
          const p = parts.find(x => x.id === id); if(!p) return;
          formWrap.open = true; summary.textContent = `Editar: ${p.nombre}`;
          form.querySelector('input[name="id"]').value = p.id;
          form.querySelector('input[name="codigo"]').value = p.codigo;
          form.querySelector('input[name="nombre"]').value = p.nombre;
          form.querySelector('input[name="categoria"]').value = p.categoria || '';
          form.querySelector('input[name="ubicacion"]').value = p.ubicacion || '';
          form.querySelector('input[name="almacen"]').value = p.almacen || '';
          form.querySelector('input[name="proveedor"]').value = p.proveedor || '';
          form.querySelector('input[name="leadTimeDias"]').value = p.leadTimeDias || 0;
          form.querySelector('input[name="consumoDiario"]').value = p.consumoDiario || 0;
          form.querySelector('input[name="stock"]').value = p.stock;
          form.querySelector('input[name="stockMin"]').value = p.stockMin;
          form.querySelector('input[name="costo"]').value = p.costo;
          form.querySelector('input[name="precio"]').value = p.precio;
        }
        if(act === 'del'){
          if(confirm('¿Eliminar esta pieza? Esta acción no se puede deshacer.')){
            DB.deletePart(id); parts = DB.listParts(); paint(parts);
          }
        }
      });

      importCSV.addEventListener('change', ()=>{
        const f = importCSV.files[0]; if(!f) return;
        const reader = new FileReader();
        reader.onload = ()=>{
          const lines = String(reader.result||'').split(/\r?\n/).filter(Boolean);
          if(!lines.length){ alert('CSV vacío'); return; }
          const head = lines.shift().split(',').map(h=>h.trim());
          const arr = DB.listParts();
          for(const line of lines){
            const cells = parseCSVLine(line);
            const obj = Object.fromEntries(head.map((h,i)=>[h, (cells[i]??'').trim()]));
            arr.push({
              id: 'id_'+Math.random().toString(36).slice(2,8),
              codigo: obj.codigo, nombre: obj.nombre, categoria: obj.categoria,
              ubicacion: obj.ubicacion, almacen: obj.almacen,
              proveedor: obj.proveedor, leadTimeDias: Number(obj.leadTimeDias||0),
              consumoDiario: Number(obj.consumoDiario||0),
              stock: Number(obj.stock||0), stockMin: Number(obj.stockMin||0),
              costo: Number(obj.costo_Bs||obj.costo||0),
              precio: Number(obj.precio_Bs||obj.precio||0)
            });
          }
          DB.saveParts(arr); parts = DB.listParts(); buildAlmacenOptions(); paint(parts);
          alert('Importación completa'); importCSV.value = '';
        };
        reader.readAsText(f);
      });
    }

    function renderMovimientos(){
      const tpl = document.getElementById('tpl-movimientos').content.cloneNode(true);
      view.innerHTML = ''; view.appendChild(tpl);

      const moveForm = document.getElementById('moveForm');
      const partSel = document.getElementById('movePart');
      const tbody = document.getElementById('movesTbody');
      const exportBtn = document.getElementById('exportCSV');
      const undoBtn = document.getElementById('undoLast');
      const almFilter = document.getElementById('moveAlmacenFilter');

      let parts = DB.listParts();
      let moves = DB.listMoves();

      function buildAlmacenOptions(){
        const set = new Set(['(Todos)']); parts.forEach(p=>{ if(p.almacen && p.almacen.trim()) set.add(p.almacen.trim()); });
        almFilter.innerHTML = Array.from(set).map(a=>`<option value="${escapeHtml(a)}">${escapeHtml(a)}</option>`).join('');
      }
      buildAlmacenOptions();

      function fillPartOptions(){
        const alm = almFilter.value || '(Todos)';
        const list = parts.filter(p => alm === '(Todos)' || String(p.almacen||'').toLowerCase()===alm.toLowerCase());
        partSel.innerHTML = list.map(p=>(
          `<option value="${p.id}">${escapeHtml(p.codigo)} — ${escapeHtml(p.nombre)} (${escapeHtml(p.almacen||'')})</option>`
        )).join('');
      }
      fillPartOptions();

      function paint(){
        moves = DB.listMoves();
        tbody.innerHTML = '';
        moves.forEach(m=>{
          const d = new Date(m.fecha);
          const tr = document.createElement('tr');
          tr.innerHTML = `
            <td>${fmtDate(d)}</td>
            <td>${escapeHtml(m.codigo)}</td>
            <td>${escapeHtml(m.nombre)}</td>
            <td>${escapeHtml(m.almacen||'')}</td>
            <td>${m.tipo === 'entrada' ? 'Entrada' : 'Salida'}</td>
            <td class="num">${m.cantidad}</td>
            <td>${escapeHtml(m.referencia||'')}</td>
            <td>${escapeHtml(m.usuario||'')}</td>`;
          tbody.appendChild(tr);
        });
      }
      paint();

      almFilter.addEventListener('change', fillPartOptions);

      moveForm.addEventListener('submit', (e)=>{
        e.preventDefault();
        const fd = new FormData(moveForm);
        const piezaId = fd.get('piezaId'); const tipo = fd.get('tipo'); const cantidad = Number(fd.get('cantidad')); const referencia = String(fd.get('referencia')||'').trim();
        if(!piezaId || !tipo || !(cantidad>0)){ alert('Complete los campos del movimiento'); return; }
        try{
          const u = Auth.currentUser();
          DB.applyMovement({piezaId, tipo, cantidad, referencia, usuarioEmail: u?u.email:''});
          parts = DB.listParts(); paint(); alert('Movimiento registrado'); moveForm.reset();
        }catch(err){ alert(err.message || 'Error al aplicar movimiento'); }
      });

      exportBtn.addEventListener('click', ()=>{
        const csv = partsToCSV(DB.listParts());
        const blob = new Blob([csv], {type:'text/csv;charset=utf-8;'});
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        const stamp = new Date().toISOString().slice(0,19).replace(/[:T]/g,'-');
        a.href = url; a.download = `inventario_${stamp}.csv`;
        document.body.appendChild(a); a.click(); document.body.removeChild(a); URL.revokeObjectURL(url);
      });

      undoBtn.addEventListener('click', ()=>{
        const moves = DB.listMoves(); const last = moves.shift();
        if(!last){ alert('Sin movimientos'); return; }
        const parts = DB.listParts(); const p = parts.find(x=>x.id===last.piezaId);
        if(p){
          if(last.tipo==='entrada') p.stock = Math.max(0, Number(p.stock)-Number(last.cantidad));
          else p.stock = Number(p.stock)+Number(last.cantidad);
          DB.saveParts(parts);
        }
        DB.saveMoves(moves); paint(); alert('Movimiento deshecho');
      });
    }

    function renderSugerencias(){
      const tpl = document.getElementById('tpl-sugerencias').content.cloneNode(true);
      view.innerHTML = ''; view.appendChild(tpl);

      const tbody = document.getElementById('sugTbody');
      const almFilter = document.getElementById('sugAlmacenFilter');
      let parts = DB.listParts();

      function buildAlmacenOptions(){
        const set = new Set(['(Todos)']); parts.forEach(p=>{ if(p.almacen && p.almacen.trim()) set.add(p.almacen.trim()); });
        almFilter.innerHTML = Array.from(set).map(a=>`<option value="${escapeHtml(a)}">${escapeHtml(a)}</option>`).join('');
      }
      buildAlmacenOptions();

      function calcularSugerencias(){
        const alm = almFilter.value || '(Todos)';
        const base = parts.filter(p => alm === '(Todos)' || String(p.almacen||'').toLowerCase()===alm.toLowerCase());
        return base.map(p=>{
          const cd = Number(p.consumoDiario||0);
          const lt = Number(p.leadTimeDias||0);
          const min = Number(p.stockMin||0);
          const stk = Number(p.stock||0);
          const rop = cd*lt + min;
          const sug = Math.max(0, rop - stk);
          return {...p, rop, sugerido:sug};
        }).filter(x=>x.sugerido>0);
      }

      function paint(){
        const list = calcularSugerencias();
        tbody.innerHTML = '';
        list.forEach(p=>{
          const tr = document.createElement('tr');
          tr.innerHTML = `
            <td>${escapeHtml(p.codigo)}</td>
            <td>${escapeHtml(p.nombre)}</td>
            <td>${escapeHtml(p.proveedor||'')}</td>
            <td>${escapeHtml(p.almacen||'')}</td>
            <td class="num">${p.stock}</td>
            <td class="num">${p.stockMin}</td>
            <td class="num">${p.consumoDiario||0}</td>
            <td class="num">${p.leadTimeDias||0}</td>
            <td class="num">${p.rop}</td>
            <td class="num">${p.sugerido}</td>`;
          tbody.appendChild(tr);
        });
      }
      almFilter.addEventListener('change', paint); paint();
    }

    function renderAjustes(){
      const tpl = document.getElementById('tpl-ajustes').content.cloneNode(true);
      view.innerHTML = ''; view.appendChild(tpl);

      const backupBtn = document.getElementById('backupBtn');
      const restoreFile = document.getElementById('restoreFile');
      const resetBtn = document.getElementById('resetBtn');

      backupBtn.addEventListener('click', ()=>{
        const dump = { users: DB.getUsers(), parts: DB.listParts(), moves: DB.listMoves(), priceHistory: DB.listPriceHistory() };
        const blob = new Blob([JSON.stringify(dump,null,2)], {type:'application/json'});
        const a = document.createElement('a'); a.href = URL.createObjectURL(blob); a.download = 'backup_inventario.json'; a.click();
      });

      restoreFile.addEventListener('change', async ()=>{
        const f = restoreFile.files[0]; if(!f) return;
        try{
          const data = JSON.parse(await f.text());
          safeWrite('users', data.users||[]);
          safeWrite('parts', data.parts||[]);
          safeWrite('movements', data.moves||[]);
          safeWrite('priceHistory', data.priceHistory||[]);
          alert('Restauración completa'); location.hash = '#/dashboard'; route();
        }catch(e){ alert('Archivo inválido'); }
        finally{ restoreFile.value = ''; }
      });

      resetBtn.addEventListener('click', ()=>{
        if(!confirm('Esto borrará los datos actuales y cargará la demo. ¿Continuar?')) return;
        DB.hardReset(); alert('Demo restablecida'); location.hash = '#/dashboard'; route();
      });
    }

    function renderHistorial(){
      const tpl = document.getElementById('tpl-historial').content.cloneNode(true);
      view.innerHTML = ''; view.appendChild(tpl);

      const tbody = document.getElementById('histTbody');
      const hist = DB.listPriceHistory();
      tbody.innerHTML = '';
      hist.forEach(h=>{
        const d = new Date(h.fecha);
        const tr = document.createElement('tr');
        tr.innerHTML = `
          <td>${fmtDate(d)}</td>
          <td>${escapeHtml(h.codigo)}</td>
          <td>${escapeHtml(h.nombre)}</td>
          <td class="num">${fmtMoney(h.costoAntes)} ➝ ${fmtMoney(h.costoDespues)}</td>
          <td class="num">${fmtMoney(h.precioAntes)} ➝ ${fmtMoney(h.precioDespues)}</td>
          <td>${escapeHtml(h.usuario||'')}</td>`;
        tbody.appendChild(tr);
      });
    }

    /* --------- Utilidades --------- */
    function fmtMoney(n){ n = Number(n||0); return n.toLocaleString('es-BO', {style:'currency', currency:'BOB'}); }
    function fmtDate(d){ return d.toLocaleString(undefined, {year:'numeric', month:'2-digit', day:'2-digit', hour:'2-digit', minute:'2-digit'}); }
    function escapeHtml(s){
      return String(s).replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
    }
    function partsToCSV(parts){
      const head = ['id','codigo','nombre','categoria','almacen','proveedor','leadTimeDias','consumoDiario','ubicacion','stock','stockMin','costo_Bs','precio_Bs'];
      const rows = parts.map(p=>{
        const row = { id:p.id, codigo:p.codigo, nombre:p.nombre, categoria:p.categoria, almacen:p.almacen, proveedor:p.proveedor, leadTimeDias:p.leadTimeDias, consumoDiario:p.consumoDiario, ubicacion:p.ubicacion, stock:p.stock, stockMin:p.stockMin, costo_Bs:p.costo, precio_Bs:p.precio };
        return head.map(k=>String(row[k] ?? '')).map(csvCell).join(',');
      });
      return head.join(',') + '\n' + rows.join('\n');
    }
    function csvCell(v){ const needsQuote = /[",\n]/.test(v); const esc = v.replace(/"/g,'""'); return needsQuote ? `"${esc}"` : esc; }
    function parseCSVLine(s){
      const out=[]; let cur=''; let q=false;
      for(let i=0;i<s.length;i++){
        const c=s[i];
        if(c==='"'){ if(q && s[i+1]==='"'){cur+='"'; i++;} else {q=!q;} continue; }
        if(c===',' && !q){ out.push(cur); cur=''; continue; }
        cur+=c;
      } out.push(cur); return out;
    }

    /* --------- Escáner (opcional) --------- */
    async function startScan(){
      try{
        if(!('BarcodeDetector' in window)){ alert('Escáner no soportado; ingrese el código manualmente.'); return; }
        const video = document.getElementById('cam');
        const stream = await navigator.mediaDevices.getUserMedia({ video:{ facingMode:'environment' }});
        video.srcObject = stream; await video.play();
        const det = new BarcodeDetector({formats:['code_128','ean_13','ean_8','qr_code','upc_a','upc_e']});
        const codeInput = document.querySelector('input[name="codigo"]');
        let running = true;
        const stop = ()=>{ running=false; if(video.srcObject){ video.srcObject.getTracks().forEach(t=>t.stop()); } video.srcObject=null; };
        const tick = async ()=>{
          if(!running) return;
          try{ const codes = await det.detect(video);
            if(codes.length){ codeInput.value = codes[0].rawValue || ''; stop(); alert('Código detectado: ' + codeInput.value); return; }
          }catch(e){}
          requestAnimationFrame(tick);
        };
        tick();
      }catch(err){ alert('No se pudo abrir la cámara'); }
    }
  </script>
</body>
</html>