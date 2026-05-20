<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>TRS — 系統後台管理</title>
<link rel="stylesheet" href="../css/main.css">
<style>
body { display:flex; height:100vh; overflow:hidden; }
.page-body { margin-left:56px; flex:1; display:flex; flex-direction:column; overflow:hidden; }
.content { flex:1; overflow-y:auto; padding:16px; display:flex; flex-direction:column; gap:16px; }

/* ── KPI ROW ── */
.kpi-grid { display:grid; grid-template-columns:repeat(4,1fr); gap:12px; }
.kpi-card {
  background:var(--bg-card); border:1px solid var(--border);
  border-radius:var(--radius-lg); padding:14px 16px;
  display:flex; flex-direction:column; gap:4px;
}
.kpi-label { font-size:11px; color:var(--text-muted); text-transform:uppercase; letter-spacing:0.5px; }
.kpi-val   { font-family:var(--mono); font-size:22px; font-weight:700; }
.kpi-sub   { font-size:11px; color:var(--text-muted); }

/* ── TRADER MATRIX ── */
.matrix-grid { display:grid; grid-template-columns:repeat(3,1fr); gap:10px; }
.trader-card {
  background:var(--bg-card); border:1px solid var(--border);
  border-radius:var(--radius-lg); padding:14px;
  position:relative; overflow:hidden;
}
.trader-card.locked { border-color:rgba(255,51,51,0.4); }
.trader-card.frozen { border-color:rgba(255,51,51,0.8); background:rgba(255,51,51,0.05); }
.trader-card-header { display:flex; align-items:center; justify-content:space-between; margin-bottom:10px; }
.trader-name { font-size:13px; font-weight:600; }
.trader-actions { display:flex; gap:4px; }

.progress-row { margin-bottom:6px; }
.progress-header { display:flex; justify-content:space-between; margin-bottom:3px; }
.progress-label { font-size:10px; color:var(--text-muted); }
.progress-val   { font-size:10px; font-family:var(--mono); }
.progress-track { height:4px; background:var(--bg-tertiary); border-radius:2px; overflow:hidden; }
.progress-fill  { height:100%; border-radius:2px; transition:width 0.4s; }

.trader-stats { display:grid; grid-template-columns:1fr 1fr; gap:6px; margin-top:8px; }
.ts-item { background:var(--bg-tertiary); border-radius:6px; padding:6px 8px; }
.ts-label { font-size:9px; color:var(--text-muted); text-transform:uppercase; }
.ts-val   { font-family:var(--mono); font-size:13px; font-weight:600; margin-top:1px; }

/* ── TWO-COL ── */
.two-col { display:grid; grid-template-columns:1fr 1fr; gap:12px; }
.section-card {
  background:var(--bg-card); border:1px solid var(--border);
  border-radius:var(--radius-lg); overflow:hidden;
}
.section-header {
  display:flex; align-items:center; justify-content:space-between;
  padding:12px 14px; border-bottom:1px solid var(--border);
  background:var(--bg-secondary);
}
.section-title { font-size:11px; font-weight:600; color:var(--text-muted); text-transform:uppercase; letter-spacing:0.5px; }
.section-body { padding:12px 14px; overflow-y:auto; max-height:280px; }

/* ── ACCOUNT LIST ── */
.account-row {
  display:flex; align-items:center; gap:10px;
  padding:8px 0; border-bottom:1px solid var(--border);
}
.account-row:last-child { border-bottom:none; }
.account-name { font-size:12px; font-weight:500; flex:1; }
.account-meta { font-size:10px; color:var(--text-muted); }

/* ── RISK RULES ── */
.rule-row {
  display:flex; align-items:center;
  padding:8px 0; border-bottom:1px solid var(--border); gap:10px;
}
.rule-row:last-child { border-bottom:none; }
.rule-name { font-size:12px; flex:1; }
.rule-val  { font-family:var(--mono); font-size:12px; color:var(--accent); }
.rule-scope { font-size:10px; color:var(--text-muted); }

/* ── MODAL OVERRIDE ── */
.form-row { display:grid; grid-template-columns:120px 1fr; align-items:center; gap:8px; margin-bottom:10px; }
.form-row label { font-size:12px; color:var(--text-muted); }
</style>
</head>
<body>

<nav class="nav-sidebar">
  <a class="nav-logo" href="../index.html">⚡</a>
  <a class="nav-item" data-page="trader" href="trader.html">
    <span>📊</span><span class="nav-tooltip">交易員下單</span>
  </a>
  <a class="nav-item active" data-page="admin" href="admin.html">
    <span>🛡</span><span class="nav-tooltip">系統後台</span>
  </a>
  <a class="nav-item" data-page="risk" href="risk-alert.html">
    <span>🚨</span><span class="nav-tooltip">風險警報</span>
  </a>
  <div class="nav-divider"></div>
  <a class="nav-item nav-bottom" href="../index.html">
    <span>🚪</span><span class="nav-tooltip">登出</span>
  </a>
</nav>

<div class="page-body">
  <div class="topbar">
    <div class="topbar-title">系統後台管理</div>
    <span class="badge badge-danger" style="margin-left:8px;">超級管理員</span>
    <div class="topbar-right">
      <span id="clockDisplay" style="font-family:var(--mono);font-size:12px;color:var(--text-muted)"></span>
      <button class="btn btn-danger btn-sm" onclick="triggerGlobalFreeze()">🔴 全局熔斷</button>
      <a class="btn btn-ghost btn-sm" href="../index.html">登出</a>
    </div>
  </div>
  <div class="index-bar" id="indexBar"></div>

  <div class="content">

    <!-- KPI -->
    <div class="kpi-grid">
      <div class="kpi-card">
        <div class="kpi-label">今日總損益</div>
        <div class="kpi-val up" id="kpiTotalPnl">-4,497,299</div>
        <div class="kpi-sub">3 位交易員 · 4 個帳號</div>
      </div>
      <div class="kpi-card">
        <div class="kpi-label">當前委託筆數</div>
        <div class="kpi-val" style="color:var(--accent)">24</div>
        <div class="kpi-sub">8 筆待成交</div>
      </div>
      <div class="kpi-card">
        <div class="kpi-label">風控警報</div>
        <div class="kpi-val warn" id="kpiAlerts">2</div>
        <div class="kpi-sub">未處理警報</div>
      </div>
      <div class="kpi-card">
        <div class="kpi-label">系統狀態</div>
        <div class="kpi-val down">正常</div>
        <div class="kpi-sub" style="display:flex;align-items:center;gap:4px;"><div class="pulse-dot"></div>Phase 1 模擬下單</div>
      </div>
    </div>

    <!-- TRADER MATRIX -->
    <div>
      <div style="font-size:11px;font-weight:600;color:var(--text-muted);text-transform:uppercase;letter-spacing:0.5px;margin-bottom:10px;">交易員矩陣監控</div>
      <div class="matrix-grid" id="traderMatrix"></div>
    </div>

    <!-- TWO COL -->
    <div class="two-col">
      <!-- 帳號清單 -->
      <div class="section-card">
        <div class="section-header">
          <span class="section-title">券商帳號管理</span>
          <button class="btn btn-ghost btn-xs" onclick="openModal('addAccountModal')">+ 新增</button>
        </div>
        <div class="section-body">
          <div class="account-row">
            <span class="badge badge-down" style="font-size:10px">期貨</span>
            <div style="flex:1">
              <div class="account-name">永豐 期-台北3240627</div>
              <div class="account-meta">SinoPac · FUTURES · 額度 60,000,000</div>
            </div>
            <span class="badge badge-down" style="font-size:10px">正常</span>
            <button class="btn btn-ghost btn-xs" onclick="lockAccount('永豐期台北')">鎖定</button>
          </div>
          <div class="account-row">
            <span class="badge badge-accent" style="font-size:10px">股票</span>
            <div style="flex:1">
              <div class="account-name">永豐 證-敦南0388939</div>
              <div class="account-meta">SinoPac · STOCK · 額度 30,000,000</div>
            </div>
            <span class="badge badge-down" style="font-size:10px">正常</span>
            <button class="btn btn-ghost btn-xs" onclick="lockAccount('永豐證敦南')">鎖定</button>
          </div>
          <div class="account-row">
            <span class="badge badge-down" style="font-size:10px">期貨</span>
            <div style="flex:1">
              <div class="account-name">富邦 期-0239847</div>
              <div class="account-meta">Fubon · FUTURES · 額度 40,000,000</div>
            </div>
            <span class="badge badge-warn" style="font-size:10px">監控中</span>
            <button class="btn btn-ghost btn-xs" onclick="lockAccount('富邦期')">鎖定</button>
          </div>
          <div class="account-row">
            <span class="badge badge-accent" style="font-size:10px">股票</span>
            <div style="flex:1">
              <div class="account-name">凱基 證-7832910</div>
              <div class="account-meta">KGI · STOCK · 額度 20,000,000</div>
            </div>
            <span class="badge badge-down" style="font-size:10px">正常</span>
            <button class="btn btn-ghost btn-xs" onclick="lockAccount('凱基證')">鎖定</button>
          </div>
        </div>
      </div>

      <!-- 風控規則 -->
      <div class="section-card">
        <div class="section-header">
          <span class="section-title">風控規則設定</span>
          <button class="btn btn-ghost btn-xs" onclick="openModal('editRuleModal')">編輯</button>
        </div>
        <div class="section-body">
          <div class="rule-row">
            <div style="flex:1">
              <div class="rule-name">單筆最大口數</div>
              <div class="rule-scope">GLOBAL</div>
            </div>
            <span class="rule-val">200 口</span>
            <button class="btn btn-ghost btn-xs" onclick="editRule('單筆最大口數')">改</button>
          </div>
          <div class="rule-row">
            <div style="flex:1">
              <div class="rule-name">單筆最大金額</div>
              <div class="rule-scope">GLOBAL</div>
            </div>
            <span class="rule-val">50,000,000</span>
            <button class="btn btn-ghost btn-xs" onclick="editRule('單筆最大金額')">改</button>
          </div>
          <div class="rule-row">
            <div style="flex:1">
              <div class="rule-name">丙種交易員日損上限</div>
              <div class="rule-scope">TRADER TYPE: CAPITAL</div>
            </div>
            <span class="rule-val">10,000,000</span>
            <button class="btn btn-ghost btn-xs" onclick="editRule('丙種日損上限')">改</button>
          </div>
          <div class="rule-row">
            <div style="flex:1">
              <div class="rule-name">自有交易員日損上限</div>
              <div class="rule-scope">TRADER TYPE: PROP</div>
            </div>
            <span class="rule-val">3,000,000</span>
            <button class="btn btn-ghost btn-xs" onclick="editRule('自有日損上限')">改</button>
          </div>
          <div class="rule-row">
            <div style="flex:1">
              <div class="rule-name">日損警戒觸發比例</div>
              <div class="rule-scope">GLOBAL</div>
            </div>
            <span class="rule-val">40%</span>
            <button class="btn btn-ghost btn-xs" onclick="editRule('日損警戒比例')">改</button>
          </div>
          <div class="rule-row">
            <div style="flex:1">
              <div class="rule-name">熔斷觸發比例</div>
              <div class="rule-scope">GLOBAL</div>
            </div>
            <span class="rule-val">80%</span>
            <button class="btn btn-ghost btn-xs" onclick="editRule('熔斷觸發比例')">改</button>
          </div>
          <div class="rule-row">
            <div style="flex:1">
              <div class="rule-name">大額二次確認閾值</div>
              <div class="rule-scope">GLOBAL</div>
            </div>
            <span class="rule-val">5,000,000</span>
            <button class="btn btn-ghost btn-xs" onclick="editRule('大額確認閾值')">改</button>
          </div>
        </div>
      </div>
    </div>

    <!-- AUDIT LOG -->
    <div class="section-card">
      <div class="section-header">
        <span class="section-title">稽核日誌 (Append-Only · Immutable)</span>
        <span style="font-size:10px;color:var(--warn);">⚠ 不可刪除、不可修改</span>
      </div>
      <div style="padding:12px 14px;overflow-x:auto;">
        <table class="data-table">
          <thead>
            <tr><th>#</th><th>時間戳</th><th>操作人</th><th>角色</th><th>動作類型</th><th>詳細說明</th><th>設備指紋</th></tr>
          </thead>
          <tbody id="auditLogBody"></tbody>
        </table>
      </div>
    </div>

  </div>
</div>

<!-- EDIT RULE MODAL -->
<div class="modal-overlay" id="editRuleModal">
  <div class="modal-box">
    <div class="modal-header">
      <span style="font-size:18px;">⚙️</span>
      <div class="modal-header-title">修改風控規則</div>
    </div>
    <div class="modal-body">
      <div style="background:var(--warn-bg);border:1px solid rgba(255,170,0,0.2);border-radius:var(--radius);padding:8px 12px;font-size:11px;color:var(--warn);margin-bottom:14px;">
        ⚠ 規則修改將立即生效，並記錄於不可修改稽核日誌
      </div>
      <div class="form-row"><label>規則名稱</label><input class="form-input" id="ruleNameInput" readonly></div>
      <div class="form-row"><label>原始值</label><input class="form-input" id="ruleOldVal" readonly></div>
      <div class="form-row"><label>新值</label><input class="form-input" id="ruleNewVal" type="number" placeholder="輸入新值"></div>
      <div class="form-row"><label>修改原因</label><input class="form-input" id="ruleReason" placeholder="請填寫修改原因"></div>
    </div>
    <div class="modal-footer">
      <button class="btn btn-ghost" onclick="closeModal('editRuleModal')">取消</button>
      <button class="btn btn-primary" onclick="saveRule()">確認修改</button>
    </div>
  </div>
</div>

<!-- LOCK CONFIRM MODAL -->
<div class="modal-overlay" id="lockModal">
  <div class="modal-box">
    <div class="modal-header">
      <span style="font-size:18px;">🔒</span>
      <div class="modal-header-title">鎖定帳號確認</div>
    </div>
    <div class="modal-body">
      <div style="font-size:13px;color:var(--text-secondary);margin-bottom:8px;">確定要鎖定帳號 <strong style="color:var(--text-primary)" id="lockTargetName">—</strong> 嗎？</div>
      <div style="font-size:12px;color:var(--text-muted);">鎖定後該帳號將無法下單，需管理員手動解鎖。此操作將記錄於稽核日誌。</div>
    </div>
    <div class="modal-footer">
      <button class="btn btn-ghost" onclick="closeModal('lockModal')">取消</button>
      <button class="btn btn-danger" onclick="confirmLock()">確認鎖定</button>
    </div>
  </div>
</div>

<div class="toast-container" id="toastContainer"></div>
<script src="../js/common.js"></script>
<script>
let lockTarget = '';

const auditLogs = [
  {seq:1005, ts:'14:47:08', user:'交易員A', role:'操盤手',    action:'ORDER_SUBMIT',   detail:'台半期066 買進 10口 82.10', fp:'f3a9b2e1'},
  {seq:1004, ts:'14:22:15', user:'系統',    role:'RISK_ENGINE', action:'RISK_ALERT',  detail:'交易員B 風險指標 121%',   fp:'SYSTEM'},
  {seq:1003, ts:'14:10:00', user:'管理員',  role:'超級管理員', action:'RULE_UPDATE',   detail:'交易員A 日損限額→10,000,000', fp:'ad8c3f12'},
  {seq:1002, ts:'13:58:40', user:'交易員C', role:'操盤手',    action:'ORDER_CONFIRM',  detail:'大額委託確認 2330 50張 @980', fp:'c7d1a4e9'},
  {seq:1001, ts:'13:22:05', user:'管理員',  role:'超級管理員', action:'ACCOUNT_UNLOCK', detail:'解鎖帳號 富邦期-0239847', fp:'ad8c3f12'},
  {seq:1000, ts:'09:00:00', user:'系統',    role:'SYSTEM',    action:'SESSION_START',  detail:'系統開盤啟動 Phase-1', fp:'SYSTEM'},
];

window.onload = () => {
  renderIndexBar();
  startClock(document.getElementById('clockDisplay'));
  setInterval(tickMarket, 3000);
  renderTraderMatrix();
  renderAuditLog();
};

function renderTraderMatrix() {
  const traders = AppState.traders;
  document.getElementById('traderMatrix').innerHTML = Object.entries(traders).map(([id, t]) => {
    const lossColor = t.lossRate >= 80 ? 'var(--danger)' : t.lossRate >= 40 ? 'var(--warn)' : 'var(--down)';
    const quotaColor = t.quotaRate >= 80 ? 'var(--danger)' : t.quotaRate >= 60 ? 'var(--warn)' : 'var(--accent)';
    return `
    <div class="trader-card ${t.locked ? 'locked' : ''}">
      <div class="trader-card-header">
        <div>
          <div class="trader-name">${t.name}</div>
          <div style="display:flex;gap:4px;margin-top:3px;">
            <span class="badge ${t.type==='CAPITAL'?'badge-warn':'badge-accent'}" style="font-size:10px">${t.type==='CAPITAL'?'丙種':'自有'}</span>
            <span class="badge ${t.locked?'badge-danger':'badge-down'}" style="font-size:10px">${t.locked?'🔒 鎖定':'正常'}</span>
          </div>
        </div>
        <div class="trader-actions">
          <button class="btn btn-warn btn-xs" onclick="freezeTrader('${id}')">熔斷</button>
          <button class="btn btn-danger btn-xs" onclick="lockTrader('${id}')">${t.locked?'解鎖':'鎖定'}</button>
        </div>
      </div>
      <div class="progress-row">
        <div class="progress-header">
          <span class="progress-label">日損使用率</span>
          <span class="progress-val" style="color:${lossColor}">${t.lossRate}%</span>
        </div>
        <div class="progress-track">
          <div class="progress-fill" style="width:${t.lossRate}%;background:${lossColor}"></div>
        </div>
      </div>
      <div class="progress-row">
        <div class="progress-header">
          <span class="progress-label">額度使用率</span>
          <span class="progress-val" style="color:${quotaColor}">${t.quotaRate}%</span>
        </div>
        <div class="progress-track">
          <div class="progress-fill" style="width:${t.quotaRate}%;background:${quotaColor}"></div>
        </div>
      </div>
      <div class="trader-stats">
        <div class="ts-item">
          <div class="ts-label">今日損益</div>
          <div class="ts-val ${t.todayPnl>=0?'down':'up'}">${fmtPnl(t.todayPnl)}</div>
        </div>
        <div class="ts-item">
          <div class="ts-label">可動用</div>
          <div class="ts-val">${fmt(t.avail)}</div>
        </div>
        <div class="ts-item">
          <div class="ts-label">浮損</div>
          <div class="ts-val up">${fmtPnl(t.floatPnl)}</div>
        </div>
        <div class="ts-item">
          <div class="ts-label">風險指標</div>
          <div class="ts-val ${t.risk>=100?'up':'down'}">${t.risk}%</div>
        </div>
      </div>
    </div>`;
  }).join('');
}

function renderAuditLog() {
  const actionColor = {
    ORDER_SUBMIT:'badge-accent', RISK_ALERT:'badge-warn', RULE_UPDATE:'badge-danger',
    ORDER_CONFIRM:'badge-down', ACCOUNT_UNLOCK:'badge-warn', SESSION_START:'badge-muted',
  };
  document.getElementById('auditLogBody').innerHTML = auditLogs.map(a => `
    <tr>
      <td class="mono muted">${a.seq}</td>
      <td class="mono">${a.ts}</td>
      <td style="color:var(--text-primary)">${a.user}</td>
      <td><span class="badge badge-muted" style="font-size:10px">${a.role}</span></td>
      <td><span class="badge ${actionColor[a.action]||'badge-accent'}" style="font-size:10px">${a.action}</span></td>
      <td style="font-size:11px;color:var(--text-secondary)">${a.detail}</td>
      <td class="mono muted" style="font-size:10px">${a.fp}</td>
    </tr>
  `).join('');
}

function lockAccount(name) {
  lockTarget = name;
  document.getElementById('lockTargetName').textContent = name;
  openModal('lockModal');
}
function confirmLock() {
  closeModal('lockModal');
  showToast('🔒 帳號已鎖定', lockTarget + ' 鎖定成功，已記錄稽核日誌', 'warn');
  auditLogs.unshift({seq:auditLogs[0].seq+1, ts:fmtTime(), user:'管理員', role:'超級管理員', action:'ACCOUNT_LOCK', detail:'鎖定帳號 '+lockTarget, fp:'ad8c3f12'});
  renderAuditLog();
}
function lockTrader(id) {
  AppState.traders[id].locked = !AppState.traders[id].locked;
  renderTraderMatrix();
  showToast(AppState.traders[id].locked ? '🔒 交易員已鎖定' : '🔓 交易員已解鎖', AppState.traders[id].name, 'warn');
}
function freezeTrader(id) {
  const t = AppState.traders[id];
  showToast('🔴 熔斷執行', t.name + ' 帳號已凍結，禁止下單', 'danger');
  t.locked = true;
  renderTraderMatrix();
  auditLogs.unshift({seq:auditLogs[0].seq+1, ts:fmtTime(), user:'管理員', role:'超級管理員', action:'FREEZE', detail:'熔斷 '+t.name, fp:'ad8c3f12'});
  renderAuditLog();
}
function triggerGlobalFreeze() {
  showToast('🔴 全局熔斷', '所有帳號已凍結，所有下單已停止', 'danger', 0);
  Object.values(AppState.traders).forEach(t => t.locked = true);
  renderTraderMatrix();
}
function editRule(name) {
  document.getElementById('ruleNameInput').value = name;
  document.getElementById('ruleOldVal').value = '—';
  document.getElementById('ruleNewVal').value = '';
  document.getElementById('ruleReason').value = '';
  openModal('editRuleModal');
}
function saveRule() {
  const reason = document.getElementById('ruleReason').value;
  if (!reason) { showToast('請填寫修改原因', '', 'warn'); return; }
  closeModal('editRuleModal');
  showToast('✅ 風控規則已更新', '已記錄稽核日誌', 'success');
  const name = document.getElementById('ruleNameInput').value;
  const newVal = document.getElementById('ruleNewVal').value;
  auditLogs.unshift({seq:auditLogs[0].seq+1, ts:fmtTime(), user:'管理員', role:'超級管理員', action:'RULE_UPDATE', detail:`修改 ${name} → ${newVal}，原因: ${reason}`, fp:'ad8c3f12'});
  renderAuditLog();
}
</script>
</body>
</html>
