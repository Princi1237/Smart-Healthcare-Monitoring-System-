<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Advanced Hospital Monitor – 5G Edition</title>
<script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-database-compat.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.2/dist/chart.umd.min.js"></script>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;600;700&family=DM+Mono:wght@400;500&display=swap" rel="stylesheet">
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body { font-family: 'DM Sans', sans-serif; background: #f0f4f8; min-height: 100vh; }

  /* ── Mode Selection ── */
  .mode-select { display: flex; flex-direction: column; align-items: center; justify-content: center; min-height: 100vh; padding: 2rem; }
  .mode-select h1 { font-size: 22px; font-weight: 700; color: #1a202c; margin-bottom: 4px; }
  .mode-select .mode-tagline { font-size: 12px; color: #a0aec0; margin-bottom: 6px; display:flex; align-items:center; gap:6px; }
  .mode-select p { font-size: 14px; color: #718096; margin-bottom: 32px; }
  .mode-cards { display: flex; gap: 16px; flex-wrap: wrap; justify-content: center; }
  .mode-card { background: #fff; border: 1.5px solid #e2e8f0; border-radius: 16px; padding: 28px 32px; width: 200px; cursor: pointer; text-align: center; transition: border-color 0.2s, box-shadow 0.2s, transform 0.15s; }
  .mode-card:hover { border-color: #4299e1; box-shadow: 0 4px 16px rgba(66,153,225,0.15); transform: translateY(-2px); }
  .mode-icon { font-size: 36px; margin-bottom: 12px; }
  .mode-card h2 { font-size: 15px; font-weight: 600; color: #2d3748; margin-bottom: 4px; }
  .mode-card p { font-size: 12px; color: #a0aec0; }

  /* ── 5G Badge ── */
  .fiveg-badge { background: linear-gradient(135deg,#667eea,#764ba2); color:#fff; font-size:10px; font-weight:700; padding:2px 8px; border-radius:999px; letter-spacing:.06em; }

  /* ── App Shell ── */
  .app { display: none; flex-direction: column; min-height: 100vh; }
  .app.active { display: flex; }

  .header { padding: 14px 20px; background: #fff; border-bottom: 1px solid #e2e8f0; display: flex; align-items: center; justify-content: space-between; }
  .header-left { display: flex; align-items: center; gap: 10px; }
  .header-dot { width: 10px; height: 10px; border-radius: 50%; }
  .header-title { font-size: 15px; font-weight: 600; color: #2d3748; }
  .header-sub { font-size: 12px; color: #a0aec0; }
  .conn-badge { font-size: 11px; padding: 3px 10px; border-radius: 999px; font-weight: 600; }
  .conn-ok { background: #c6f6d5; color: #276749; }
  .conn-off { background: #fed7d7; color: #9b2c2c; }
  .back-btn { font-size: 12px; color: #718096; background: none; border: none; cursor: pointer; padding: 4px 8px; }

  .content { flex: 1; padding: 16px; max-width: 560px; margin: 0 auto; width: 100%; }

  /* ── 5G Panel ── */
  .fiveg-panel { background: linear-gradient(135deg,#1a1a2e,#16213e); border-radius: 14px; padding: 14px 16px; margin-bottom: 14px; color:#fff; }
  .fiveg-panel-head { display:flex; align-items:center; justify-content:space-between; margin-bottom:12px; }
  .fiveg-panel-title { font-size:12px; font-weight:700; color:#a78bfa; text-transform:uppercase; letter-spacing:.07em; display:flex; align-items:center; gap:6px; }
  .fiveg-grid { display:grid; grid-template-columns:repeat(4,1fr); gap:8px; }
  .fiveg-metric { background:rgba(255,255,255,0.07); border-radius:10px; padding:10px 8px; text-align:center; }
  .fiveg-metric-val { font-size:16px; font-weight:700; color:#c4b5fd; font-family:'DM Mono',monospace; }
  .fiveg-metric-label { font-size:9px; color:#94a3b8; margin-top:3px; text-transform:uppercase; letter-spacing:.05em; }
  .fiveg-signal { display:flex; align-items:flex-end; gap:2px; height:14px; margin-top:6px; justify-content:center; }
  .fiveg-bar { width:4px; background:#4c1d95; border-radius:1px; }
  .fiveg-bar.on { background:#a78bfa; }
  .fiveg-latency-dot { width:8px; height:8px; border-radius:50%; background:#34d399; display:inline-block; margin-right:4px; animation:blink 1.4s infinite; }
  .fiveg-network-type { font-size:10px; background:rgba(167,139,250,0.2); color:#a78bfa; padding:2px 8px; border-radius:999px; font-weight:600; }

  /* ── Patient Name Screen ── */
  .name-screen-content { display: flex; flex-direction: column; align-items: center; justify-content: center; flex: 1; padding: 32px 16px; }
  .name-card { background: #fff; border-radius: 20px; padding: 32px 28px; width: 100%; max-width: 380px; border: 1.5px solid #e2e8f0; text-align: center; box-shadow: 0 4px 24px rgba(0,0,0,0.07); }
  .name-card-icon { font-size: 48px; margin-bottom: 14px; }
  .name-card h2 { font-size: 18px; font-weight: 700; color: #2d3748; margin-bottom: 6px; }
  .name-card p { font-size: 13px; color: #718096; margin-bottom: 24px; line-height: 1.6; }
  .name-input { width: 100%; padding: 12px 14px; border: 1.5px solid #e2e8f0; border-radius: 10px; font-size: 15px; font-family: 'DM Sans', sans-serif; color: #2d3748; outline: none; transition: border-color 0.2s; margin-bottom: 6px; }
  .name-input:focus { border-color: #4299e1; box-shadow: 0 0 0 3px rgba(66,153,225,0.15); }
  .name-input.error { border-color: #fc8181; }
  .name-error { font-size: 12px; color: #e53e3e; margin-bottom: 16px; min-height: 18px; text-align: left; padding-left: 2px; }
  .name-btn { width: 100%; padding: 12px; background: #4299e1; color: #fff; border: none; border-radius: 10px; font-size: 14px; font-weight: 600; cursor: pointer; transition: background 0.2s, transform 0.1s; font-family: 'DM Sans', sans-serif; }
  .name-btn:hover { background: #3182ce; }
  .name-btn:active { transform: scale(0.98); }

  /* ── Patient Side ── */
  .patient-name-badge { background: #ebf8ff; border: 1px solid #bee3f8; border-radius: 8px; padding: 4px 10px; font-size: 12px; color: #2b6cb0; font-weight: 600; }
  .param-block { background: #fff; border-radius: 12px; padding: 16px; margin-bottom: 12px; border: 1px solid #e2e8f0; }
  .param-top { display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px; }
  .param-name { font-size: 13px; color: #718096; font-weight: 500; }
  .param-val { font-size: 20px; font-weight: 700; color: #2d3748; font-variant-numeric: tabular-nums; }
  .param-unit { font-size: 12px; color: #a0aec0; font-weight: 400; }
  .param-range { font-size: 11px; color: #cbd5e0; margin-top: 6px; }
  input[type=range] { width: 100%; accent-color: #4299e1; cursor: pointer; }
  .live-sync-bar { display: flex; align-items: center; gap: 8px; background: #ebf8ff; border: 1px solid #bee3f8; border-radius: 10px; padding: 10px 14px; margin-top: 4px; }
  .live-dot { width: 8px; height: 8px; border-radius: 50%; background: #48bb78; flex-shrink: 0; animation: blink 1.4s infinite; }
  .live-sync-bar span { font-size: 12px; color: #2b6cb0; font-weight: 500; }
  .live-sync-bar .live-time { margin-left: auto; font-family: 'DM Mono', monospace; font-size: 11px; color: #63b3ed; }

  /* ── Simulation Controls ── */
  .sim-card { background: linear-gradient(135deg,#f0fff4,#e6fffa); border: 1.5px solid #9ae6b4; border-radius: 14px; padding: 14px 16px; margin-bottom: 14px; }
  .sim-card-head { display:flex; align-items:center; justify-content:space-between; margin-bottom:10px; }
  .sim-card-title { font-size:13px; font-weight:700; color:#276749; display:flex; align-items:center; gap:6px; }
  .sim-toggle-wrap { display:flex; align-items:center; gap:8px; }
  .sim-toggle-label { font-size:12px; color:#718096; }
  .toggle-switch { position:relative; width:38px; height:20px; cursor:pointer; }
  .toggle-switch input { opacity:0; width:0; height:0; }
  .toggle-slider { position:absolute; inset:0; background:#cbd5e0; border-radius:999px; transition:.3s; }
  .toggle-slider:before { content:''; position:absolute; width:16px; height:16px; left:2px; top:2px; background:#fff; border-radius:50%; transition:.3s; }
  .toggle-switch input:checked + .toggle-slider { background:#48bb78; }
  .toggle-switch input:checked + .toggle-slider:before { transform:translateX(18px); }
  .sim-scenarios { display:flex; gap:8px; flex-wrap:wrap; margin-top:8px; }
  .sim-btn { font-size:11px; padding:6px 12px; border-radius:999px; border:1.5px solid #9ae6b4; background:#fff; color:#276749; font-weight:600; cursor:pointer; transition:background .15s,border-color .15s; font-family:'DM Sans',sans-serif; }
  .sim-btn:hover { background:#c6f6d5; border-color:#48bb78; }
  .sim-btn.crisis { border-color:#fc8181; color:#c53030; }
  .sim-btn.crisis:hover { background:#fff5f5; border-color:#e53e3e; }
  .sim-btn.active-sim { background:#c6f6d5; border-color:#48bb78; }
  .sim-btn.crisis.active-sim { background:#fff5f5; border-color:#e53e3e; }
  .sim-status { font-size:11px; color:#718096; margin-top:8px; display:flex; align-items:center; gap:5px; }

  /* ── Doctor Patient List ── */
  .section-title { font-size: 11px; font-weight: 700; color: #a0aec0; text-transform: uppercase; letter-spacing: 0.07em; margin-bottom: 10px; margin-top: 4px; }
  .patients-list { display: flex; flex-direction: column; gap: 8px; margin-bottom: 18px; }
  .patient-item { background: #fff; border: 1.5px solid #e2e8f0; border-radius: 12px; padding: 12px 14px; display: flex; align-items: center; gap: 12px; cursor: pointer; transition: border-color 0.2s, box-shadow 0.15s, transform 0.1s; }
  .patient-item:hover { border-color: #4299e1; box-shadow: 0 2px 10px rgba(66,153,225,0.12); transform: translateY(-1px); }
  .patient-item.selected { border-color: #4299e1; background: #ebf8ff; }
  .patient-item.alerted { border-color: #fc8181; background: #fff5f5; }
  .patient-item.alerted.selected { border-color: #e53e3e; background: #fff5f5; }
  .patient-avatar { width: 38px; height: 38px; border-radius: 50%; background: #e2e8f0; display: flex; align-items: center; justify-content: center; font-size: 15px; font-weight: 700; color: #4a5568; flex-shrink: 0; }
  .patient-item.selected .patient-avatar { background: #bee3f8; color: #2b6cb0; }
  .patient-item.alerted .patient-avatar { background: #fed7d7; color: #c53030; }
  .patient-info { flex: 1; min-width: 0; }
  .patient-item-name { font-size: 14px; font-weight: 600; color: #2d3748; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
  .patient-item-time { font-size: 11px; color: #a0aec0; font-family: 'DM Mono', monospace; margin-top: 2px; }
  .patient-item-sub { font-size:10px; color:#a78bfa; font-weight:600; margin-top:2px; }
  .patient-item.alerted .patient-item-name { color: #c53030; }

  /* ── Risk Score ── */
  .risk-badge { font-size:11px; padding:3px 9px; border-radius:999px; font-weight:700; white-space:nowrap; }
  .risk-low { background:#c6f6d5; color:#276749; }
  .risk-mod { background:#fefcbf; color:#744210; }
  .risk-high { background:#fed7d7; color:#9b2c2c; }

  /* ── Selected Patient Bar ── */
  .selected-patient-bar { background: #fff; border: 1.5px solid #4299e1; border-radius: 12px; padding: 12px 14px; margin-bottom: 12px; display: flex; align-items: center; justify-content: space-between; }
  .selected-pt-avatar { width: 34px; height: 34px; border-radius: 50%; background: #bee3f8; color: #2b6cb0; font-size: 14px; font-weight: 700; display: flex; align-items: center; justify-content: center; flex-shrink: 0; }
  .selected-pt-name { font-size: 14px; font-weight: 700; color: #2d3748; }
  .selected-pt-sub { font-size: 11px; color: #a0aec0; margin-top: 1px; }
  .deselect-btn { font-size: 11px; color: #4299e1; background: none; border: 1px solid #bee3f8; border-radius: 999px; padding: 4px 10px; cursor: pointer; font-weight: 600; white-space: nowrap; }
  .deselect-btn:hover { background: #ebf8ff; }
  .no-patient-msg { text-align: center; color: #cbd5e0; font-size: 14px; padding: 40px 0 20px; }

  /* ── Tabs ── */
  .tab-bar { display: flex; gap: 6px; margin-bottom: 14px; background: #edf2f7; padding: 4px; border-radius: 10px; }
  .tab-btn { flex: 1; padding: 8px 12px; border: none; border-radius: 7px; font-size: 12px; font-weight: 600; cursor: pointer; background: transparent; color: #718096; transition: background 0.15s, color 0.15s; font-family: 'DM Sans', sans-serif; }
  .tab-btn.active { background: #fff; color: #2d3748; box-shadow: 0 1px 4px rgba(0,0,0,0.08); }
  .tab-btn:hover:not(.active) { color: #4a5568; }

  /* ── Monitor Live ── */
  .notif-banner { display: flex; align-items: center; gap: 10px; background: #fffbeb; border: 1px solid #f6e05e; border-radius: 10px; padding: 10px 14px; margin-bottom: 12px; }
  .notif-banner span { font-size: 12px; color: #744210; }
  .notif-btn { margin-left: auto; font-size: 11px; padding: 4px 12px; border-radius: 999px; background: #f6ad55; color: #7b341e; font-weight: 600; border: none; cursor: pointer; white-space: nowrap; }
  .notif-btn:hover { background: #ed8936; }
  .notif-banner.granted { background: #f0fff4; border-color: #9ae6b4; }
  .notif-banner.granted span { color: #276749; }
  .notif-banner.denied { background: #fff5f5; border-color: #fc8181; }
  .notif-banner.denied span { color: #9b2c2c; }

  .alert-banner { background: #fff5f5; border: 1px solid #fc8181; border-radius: 10px; padding: 12px 16px; margin-bottom: 14px; font-size: 13px; color: #c53030; font-weight: 600; display: none; animation: shake 0.4s ease; }
  .alert-banner.show { display: block; }
  @keyframes shake { 0%,100%{transform:translateX(0)} 20%,60%{transform:translateX(-4px)} 40%,80%{transform:translateX(4px)} }

  .last-update { font-size: 11px; color: #a0aec0; text-align: right; margin-bottom: 12px; font-family: 'DM Mono', monospace; }

  .vital-card { background: #fff; border-radius: 12px; padding: 14px 16px; margin-bottom: 10px; border: 1.5px solid #e2e8f0; display: flex; justify-content: space-between; align-items: center; transition: border-color 0.3s, background 0.3s; }
  .vital-card.alert { border-color: #fc8181; background: #fff5f5; }
  .vital-label { font-size: 12px; color: #a0aec0; margin-bottom: 4px; }
  .vital-num { font-size: 26px; font-weight: 700; color: #2d3748; font-variant-numeric: tabular-nums; }
  .vital-card.alert .vital-num { color: #c53030; }
  .vital-unit-lbl { font-size: 12px; color: #a0aec0; }
  .pill { font-size: 11px; padding: 4px 10px; border-radius: 999px; font-weight: 600; }
  .pill-ok { background: #c6f6d5; color: #276749; }
  .pill-alert { background: #fed7d7; color: #9b2c2c; }

  .waiting { text-align: center; color: #cbd5e0; font-size: 14px; padding: 40px 0; }
  .sync-row { display: flex; align-items: center; justify-content: center; gap: 6px; font-size: 11px; color: #a0aec0; margin-top: 14px; }
  .sync-dot { width: 7px; height: 7px; border-radius: 50%; background: #48bb78; animation: blink 1.4s infinite; }
  @keyframes blink { 0%,100%{opacity:1} 50%{opacity:0.2} }

  /* ── History Charts ── */
  .history-meta { display: flex; align-items: center; justify-content: space-between; margin-bottom: 14px; }
  .history-count-badge { font-size: 11px; background: #edf2f7; color: #718096; border-radius: 999px; padding: 3px 10px; font-weight: 600; }
  .history-loading { text-align: center; color: #a0aec0; font-size: 13px; padding: 32px 0; }

  .chart-card { background: #fff; border-radius: 14px; border: 1.5px solid #e2e8f0; padding: 14px 16px; margin-bottom: 12px; transition: border-color 0.2s; }
  .chart-card.alert { border-color: #fc8181; }
  .chart-card-header { display: flex; align-items: center; justify-content: space-between; margin-bottom: 12px; }
  .chart-card-left { display: flex; flex-direction: column; gap: 2px; }
  .chart-card-label { font-size: 11px; font-weight: 600; color: #a0aec0; text-transform: uppercase; letter-spacing: 0.06em; }
  .chart-card-value { font-size: 22px; font-weight: 700; color: #2d3748; font-variant-numeric: tabular-nums; line-height: 1; }
  .chart-card-value span { font-size: 12px; color: #a0aec0; font-weight: 400; }
  .chart-card.alert .chart-card-value { color: #c53030; }
  .chart-canvas-wrap { position: relative; height: 100px; }
  .chart-range-hint { font-size: 10px; color: #cbd5e0; margin-top: 6px; text-align: right; font-family: 'DM Mono', monospace; }

  /* ── Averages Tab ── */
  .avg-period-bar { display:flex; gap:6px; margin-bottom:16px; }
  .avg-period-btn { flex:1; padding:7px 8px; border:1.5px solid #e2e8f0; border-radius:8px; font-size:12px; font-weight:600; color:#718096; background:#fff; cursor:pointer; transition:.15s; font-family:'DM Sans',sans-serif; text-align:center; }
  .avg-period-btn.active { border-color:#4299e1; color:#2b6cb0; background:#ebf8ff; }
  .avg-period-btn:hover:not(.active) { border-color:#bee3f8; color:#4a5568; }

  .avg-vital-card { background:#fff; border-radius:14px; border:1.5px solid #e2e8f0; padding:14px 16px; margin-bottom:10px; }
  .avg-vital-header { display:flex; align-items:center; justify-content:space-between; margin-bottom:12px; }
  .avg-vital-label { font-size:11px; font-weight:700; color:#a0aec0; text-transform:uppercase; letter-spacing:.07em; }
  .avg-vital-current { font-size:13px; font-weight:700; color:#2d3748; }
  .avg-stats-row { display:grid; grid-template-columns:repeat(3,1fr); gap:8px; }
  .avg-stat { background:#f7fafc; border-radius:10px; padding:10px; text-align:center; }
  .avg-stat-val { font-size:18px; font-weight:700; color:#2d3748; font-family:'DM Mono',monospace; font-variant-numeric:tabular-nums; }
  .avg-stat-val.alert-val { color:#e53e3e; }
  .avg-stat-label { font-size:10px; color:#a0aec0; margin-top:2px; text-transform:uppercase; letter-spacing:.05em; }
  .avg-trend { display:flex; align-items:center; gap:4px; margin-top:10px; font-size:11px; color:#718096; padding-top:10px; border-top:1px solid #f0f4f8; }
  .trend-up { color:#e53e3e; } .trend-down { color:#38a169; } .trend-flat { color:#a0aec0; }
  .avg-no-data { text-align:center; color:#cbd5e0; font-size:13px; padding:24px 0; }

  /* ── Notification Bell ── */
  .notif-bell-wrap { position: relative; display: inline-flex; }
  .notif-bell-btn { background: none; border: 1.5px solid #e2e8f0; border-radius: 8px; padding: 5px 8px; cursor: pointer; font-size: 16px; line-height: 1; color: #4a5568; transition: background 0.15s, border-color 0.15s; }
  .notif-bell-btn:hover { background: #ebf8ff; border-color: #bee3f8; }
  .notif-bell-btn.has-alerts { border-color: #fc8181; animation: bellShake 0.5s ease; }
  @keyframes bellShake { 0%,100%{transform:rotate(0)} 25%{transform:rotate(-12deg)} 75%{transform:rotate(12deg)} }
  .notif-badge { position: absolute; top: -6px; right: -6px; background: #e53e3e; color: #fff; font-size: 10px; font-weight: 700; min-width: 18px; height: 18px; border-radius: 999px; display: none; align-items: center; justify-content: center; padding: 0 4px; border: 2px solid #fff; }
  .notif-badge.show { display: flex; }

  /* ── Notification Panel ── */
  .notif-panel-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.25); z-index: 100; display: none; }
  .notif-panel-overlay.open { display: block; }
  .notif-panel { position: fixed; top: 0; right: -340px; width: 320px; max-width: 94vw; height: 100vh; background: #fff; box-shadow: -4px 0 24px rgba(0,0,0,0.13); z-index: 101; display: flex; flex-direction: column; transition: right 0.28s cubic-bezier(.4,0,.2,1); }
  .notif-panel.open { right: 0; }
  .notif-panel-head { padding: 16px; border-bottom: 1px solid #e2e8f0; display: flex; align-items: center; justify-content: space-between; }
  .notif-panel-head h3 { font-size: 14px; font-weight: 700; color: #2d3748; }
  .notif-panel-actions { display: flex; gap: 6px; }
  .notif-clear-btn { font-size: 11px; padding: 4px 10px; border-radius: 999px; background: #edf2f7; color: #4a5568; border: none; cursor: pointer; font-weight: 600; }
  .notif-clear-btn:hover { background: #e2e8f0; }
  .notif-close-btn { font-size: 18px; background: none; border: none; cursor: pointer; color: #a0aec0; line-height: 1; padding: 2px 4px; }
  .notif-close-btn:hover { color: #4a5568; }
  .notif-list { flex: 1; overflow-y: auto; padding: 10px 12px; }
  .notif-empty { text-align: center; color: #cbd5e0; font-size: 13px; padding: 40px 0; }
  .notif-item { border-radius: 10px; padding: 10px 12px; margin-bottom: 8px; border-left: 3px solid #fc8181; background: #fff5f5; }
  .notif-item.resolved { border-left-color: #9ae6b4; background: #f0fff4; }
  .notif-item-title { font-size: 12px; font-weight: 700; color: #c53030; margin-bottom: 2px; }
  .notif-item.resolved .notif-item-title { color: #276749; }
  .notif-item-body { font-size: 11px; color: #718096; line-height: 1.5; }
  .notif-item-time { font-size: 10px; color: #a0aec0; margin-top: 4px; font-family: 'DM Mono', monospace; }
  .notif-sound-row { padding: 12px 16px; border-top: 1px solid #e2e8f0; display: flex; align-items: center; gap: 8px; }
  .notif-sound-row label { font-size: 12px; color: #4a5568; display: flex; align-items: center; gap: 6px; cursor: pointer; }
  .notif-sound-row input[type=checkbox] { accent-color: #4299e1; width: 14px; height: 14px; cursor: pointer; }

  /* ── Toast ── */
  .toast-container { position: fixed; bottom: 20px; right: 16px; z-index: 200; display: flex; flex-direction: column-reverse; gap: 8px; pointer-events: none; }
  .toast { background: #fff; border-radius: 10px; padding: 12px 16px; box-shadow: 0 4px 20px rgba(0,0,0,0.15); border-left: 4px solid #fc8181; max-width: 280px; pointer-events: auto; display: flex; align-items: flex-start; gap: 10px; font-size: 12px; animation: toastIn 0.3s ease; }
  .toast.resolve { border-left-color: #48bb78; }
  @keyframes toastIn { from{opacity:0;transform:translateY(16px)} to{opacity:1;transform:translateY(0)} }
  @keyframes toastOut { from{opacity:1;transform:translateY(0)} to{opacity:0;transform:translateY(16px)} }
  .toast-icon { font-size: 16px; flex-shrink: 0; }
  .toast-content { flex: 1; }
  .toast-title { font-weight: 700; color: #c53030; margin-bottom: 2px; }
  .toast.resolve .toast-title { color: #276749; }
  .toast-body { color: #4a5568; line-height: 1.4; }
  .toast-close { background: none; border: none; cursor: pointer; color: #a0aec0; font-size: 14px; padding: 0; flex-shrink: 0; }

  /* ── Demo Patients Banner ── */
  .demo-banner { background: linear-gradient(135deg,#ebf8ff,#e9d8fd); border:1.5px solid #bee3f8; border-radius:12px; padding:12px 14px; margin-bottom:14px; }
  .demo-banner-head { font-size:12px; font-weight:700; color:#553c9a; margin-bottom:8px; display:flex; align-items:center; gap:6px; }
  .demo-patients-row { display:flex; gap:8px; flex-wrap:wrap; }
  .demo-pt-btn { font-size:11px; padding:6px 12px; border-radius:999px; border:1.5px solid #b794f4; background:#fff; color:#553c9a; font-weight:600; cursor:pointer; transition:.15s; font-family:'DM Sans',sans-serif; }
  .demo-pt-btn:hover { background:#e9d8fd; }
  .demo-pt-btn.active-demo { background:#d6bcfa; border-color:#9f7aea; }
</style>
</head>
<body>

<!-- Mode Selection -->
<div class="mode-select" id="modeSelect">
  <h1>Advanced Hospital Monitor</h1>
  <div class="mode-tagline">
    <span class="fiveg-badge">5G NR</span>
    <span style="color:#a0aec0;">Ultra-low latency patient monitoring</span>
  </div>
  <p style="margin-bottom:32px;">Choose your role on this device</p>
  <div class="mode-cards">
    <div class="mode-card" onclick="startApp('patient')">
      <div class="mode-icon">🏥</div>
      <h2>Patient</h2>
      <p>Enter vitals manually or simulate</p>
    </div>
    <div class="mode-card" onclick="startApp('monitor')">
      <div class="mode-icon">🩺</div>
      <h2>Doctor / Nurse</h2>
      <p>Monitor dashboard</p>
    </div>
  </div>
</div>

<!-- Patient Name Screen -->
<div class="app" id="patientNameScreen">
  <div class="header">
    <div class="header-left">
      <div class="header-dot" style="background:#48bb78"></div>
      <div>
        <div class="header-title">Patient Login</div>
        <div class="header-sub">Enter your name to continue</div>
      </div>
    </div>
    <button class="back-btn" onclick="goBack()">&#8592; Back</button>
  </div>
  <div class="name-screen-content">
    <div class="name-card">
      <div class="name-card-icon">🏥</div>
      <h2>Welcome, Patient</h2>
      <p>Enter your name so your doctor can identify your readings on their dashboard.</p>
      <input type="text" id="patientNameInput" class="name-input" placeholder="e.g. John Smith"
             maxlength="50" autocomplete="off" spellcheck="false"
             oninput="clearNameError()"
             onkeydown="if(event.key==='Enter') confirmPatientName()">
      <div class="name-error" id="nameError"></div>
      <button class="name-btn" onclick="confirmPatientName()">Start Monitoring &rarr;</button>
    </div>
  </div>
</div>

<!-- Patient App -->
<div class="app" id="patientApp">
  <div class="header">
    <div class="header-left">
      <div class="header-dot" style="background:#48bb78"></div>
      <div>
        <div class="header-title">Patient Device</div>
        <div class="header-sub">Vitals input &mdash; 5G live sync</div>
      </div>
    </div>
    <div style="display:flex;gap:8px;align-items:center;">
      <span class="patient-name-badge" id="patientNameBadge"></span>
      <span class="fiveg-badge">5G</span>
      <span class="conn-badge conn-off" id="pConn">Connecting&hellip;</span>
      <button class="back-btn" onclick="goBack()">&#8592; Back</button>
    </div>
  </div>
  <div class="content">

    <!-- Simulation Card -->
    <div class="sim-card">
      <div class="sim-card-head">
        <div class="sim-card-title">🤖 Virtual Patient Simulator</div>
        <div class="sim-toggle-wrap">
          <span class="sim-toggle-label">Auto-simulate</span>
          <label class="toggle-switch">
            <input type="checkbox" id="simToggle" onchange="toggleSimulation()">
            <span class="toggle-slider"></span>
          </label>
        </div>
      </div>
      <div class="sim-scenarios">
        <button class="sim-btn" id="scenNormal" onclick="setScenario('normal')">💚 Normal</button>
        <button class="sim-btn" id="scenTachy" onclick="setScenario('tachycardia')">💛 Tachycardia</button>
        <button class="sim-btn" id="scenHyper" onclick="setScenario('hypertension')">🟠 Hypertension</button>
        <button class="sim-btn crisis" id="scenCrisis" onclick="setScenario('crisis')">🔴 Crisis</button>
        <button class="sim-btn" id="scenRecovery" onclick="setScenario('recovery')">🔵 Recovery</button>
      </div>
      <div class="sim-status" id="simStatus">
        <span>Enable auto-simulate to generate realistic sensor-like readings</span>
      </div>
    </div>

    <!-- Sliders -->
    <div class="param-block">
      <div class="param-top">
        <span class="param-name">Heart rate</span>
        <span class="param-val" id="hrVal">78 <span class="param-unit">bpm</span></span>
      </div>
      <input type="range" id="hr" min="0" max="250" value="78" step="1"
             oninput="updateSlider('hr','hrVal','bpm'); scheduleSync()">
      <div class="param-range">Safe range: 60 &ndash; 100 bpm</div>
    </div>
    <div class="param-block">
      <div class="param-top">
        <span class="param-name">SpO&#8322;</span>
        <span class="param-val" id="spo2Val">97 <span class="param-unit">%</span></span>
      </div>
      <input type="range" id="spo2" min="0" max="100" value="97" step="1"
             oninput="updateSlider('spo2','spo2Val','%'); scheduleSync()">
      <div class="param-range">Safe range: &ge; 94%</div>
    </div>
    <div class="param-block">
      <div class="param-top">
        <span class="param-name">Temperature</span>
        <span class="param-val" id="tempVal">98.6 <span class="param-unit">&deg;F</span></span>
      </div>
      <input type="range" id="temp" min="0" max="115" value="98.6" step="0.1"
             oninput="updateTempSlider(); scheduleSync()">
      <div class="param-range">Safe range: 97 &ndash; 99.5 &deg;F</div>
    </div>
    <div class="param-block">
      <div class="param-top">
        <span class="param-name">Blood pressure (systolic)</span>
        <span class="param-val" id="bpVal">120 <span class="param-unit">mmHg</span></span>
      </div>
      <input type="range" id="bp" min="0" max="250" value="120" step="1"
             oninput="updateSlider('bp','bpVal','mmHg'); scheduleSync()">
      <div class="param-range">Safe range: 90 &ndash; 140 mmHg</div>
    </div>
    <div class="live-sync-bar">
      <div class="live-dot"></div>
      <span>Data syncs via 5G — ultra-low latency (&lt;5ms)</span>
      <span class="live-time" id="patientSyncTime">&mdash;</span>
    </div>
  </div>
</div>

<!-- Monitor App -->
<div class="app" id="monitorApp">
  <div class="header">
    <div class="header-left">
      <div class="header-dot" style="background:#4299e1"></div>
      <div>
        <div class="header-title">Doctor Dashboard</div>
        <div class="header-sub">5G Real-time monitoring</div>
      </div>
    </div>
    <div style="display:flex;gap:8px;align-items:center;">
      <span class="fiveg-badge">5G</span>
      <span class="conn-badge conn-off" id="mConn">Connecting&hellip;</span>
      <div class="notif-bell-wrap">
        <button class="notif-bell-btn" id="notifBellBtn" onclick="toggleNotifPanel()" title="Notifications">🔔</button>
        <span class="notif-badge" id="notifBadge">0</span>
      </div>
      <button class="back-btn" onclick="goBack()">&#8592; Back</button>
    </div>
  </div>
  <div class="content">

    <!-- Real Latency Panel -->
    <div class="fiveg-panel" id="fivegPanel">
      <div class="fiveg-panel-head">
        <div class="fiveg-panel-title">📡 Live Network Latency <span style="font-size:9px;color:#6ee7b7;font-weight:400;text-transform:none;letter-spacing:0">&nbsp;— real Firebase RTT</span></div>
        <span class="fiveg-network-type" id="fivegNetType" style="cursor:pointer" title="Click to run a ping now" onclick="runPing()">Measuring…</span>
      </div>

      <!-- Main RTT row -->
      <div style="display:flex;align-items:flex-end;gap:10px;margin-bottom:12px;">
        <div style="flex:1;background:rgba(255,255,255,0.07);border-radius:10px;padding:12px 14px;">
          <div style="font-size:10px;color:#94a3b8;text-transform:uppercase;letter-spacing:.06em;margin-bottom:4px;">Firebase RTT (ping)</div>
          <div style="display:flex;align-items:baseline;gap:4px;">
            <span class="fiveg-latency-dot"></span>
            <span id="rttValue" style="font-size:28px;font-weight:700;color:#c4b5fd;font-family:'DM Mono',monospace;">—</span>
            <span style="font-size:13px;color:#94a3b8;">ms</span>
          </div>
          <div style="font-size:10px;color:#6ee7b7;margin-top:4px;" id="rttQuality">Waiting for first ping…</div>
        </div>
        <div style="flex:1;background:rgba(255,255,255,0.07);border-radius:10px;padding:12px 14px;">
          <div style="font-size:10px;color:#94a3b8;text-transform:uppercase;letter-spacing:.06em;margin-bottom:4px;">Data Propagation</div>
          <div style="display:flex;align-items:baseline;gap:4px;">
            <span id="propValue" style="font-size:28px;font-weight:700;color:#67e8f9;font-family:'DM Mono',monospace;">—</span>
            <span style="font-size:13px;color:#94a3b8;">ms</span>
          </div>
          <div style="font-size:10px;color:#94a3b8;margin-top:4px;">Patient → Doctor</div>
        </div>
      </div>

      <!-- Stats row -->
      <div class="fiveg-grid" style="grid-template-columns:repeat(4,1fr);">
        <div class="fiveg-metric">
          <div class="fiveg-metric-val" id="rttMin" style="color:#6ee7b7;">—</div>
          <div class="fiveg-metric-label">Min ms</div>
        </div>
        <div class="fiveg-metric">
          <div class="fiveg-metric-val" id="rttAvg" style="color:#c4b5fd;">—</div>
          <div class="fiveg-metric-label">Avg ms</div>
        </div>
        <div class="fiveg-metric">
          <div class="fiveg-metric-val" id="rttMax" style="color:#f9a8d4;">—</div>
          <div class="fiveg-metric-label">Max ms</div>
        </div>
        <div class="fiveg-metric">
          <div class="fiveg-metric-val" id="pingCount" style="color:#fcd34d;">0</div>
          <div class="fiveg-metric-label">Pings</div>
        </div>
      </div>

      <!-- Sparkline -->
      <div style="margin-top:12px;">
        <div style="font-size:9px;color:#64748b;text-transform:uppercase;letter-spacing:.06em;margin-bottom:5px;">RTT history (last 20 pings)</div>
        <canvas id="latencySparkline" height="36" style="width:100%;display:block;"></canvas>
      </div>

      <!-- Footer row -->
      <div style="display:flex;align-items:center;justify-content:space-between;margin-top:10px;">
        <div style="font-size:10px;color:#64748b;" id="lastPingTime">No pings yet</div>
        <div style="display:flex;gap:6px;align-items:center;">
          <div class="fiveg-signal" id="fivegSignal">
            <div class="fiveg-bar" style="height:4px"></div>
            <div class="fiveg-bar" style="height:6px"></div>
            <div class="fiveg-bar" style="height:9px"></div>
            <div class="fiveg-bar" style="height:12px"></div>
            <div class="fiveg-bar" style="height:14px"></div>
          </div>
          <span style="font-size:10px;color:#94a3b8;" id="connQualityLabel">—</span>
        </div>
      </div>
    </div>

    <!-- Notification permission banner -->
    <div class="notif-banner" id="notifBanner">
      <span id="notifText">🔔 Enable notifications to get phone alerts when vitals exceed safe limits.</span>
      <button class="notif-btn" id="notifBtn" onclick="requestNotifPermission()">Allow</button>
    </div>

    <!-- Demo Patients Banner -->
    <div class="demo-banner">
      <div class="demo-banner-head">🧪 Demo Scenario Patients</div>
      <div class="demo-patients-row" id="demoPatientsRow">
        <button class="demo-pt-btn" id="demoA" onclick="activateDemoPatient('A')">🟢 Alice – Normal</button>
        <button class="demo-pt-btn" id="demoB" onclick="activateDemoPatient('B')">🟡 Bob – ICU</button>
        <button class="demo-pt-btn" id="demoC" onclick="activateDemoPatient('C')">🔴 Carol – Critical</button>
      </div>
    </div>

    <!-- Patients List -->
    <div class="section-title">Active Patients</div>
    <div class="patients-list" id="patientsList">
      <div class="waiting">No patients connected yet.</div>
    </div>

    <!-- Selected Patient Area -->
    <div id="selectedVitalsArea" style="display:none">

      <!-- Patient bar -->
      <div class="selected-patient-bar">
        <div style="display:flex;align-items:center;gap:10px;">
          <div class="selected-pt-avatar" id="selectedPtAvatar">?</div>
          <div>
            <div class="selected-pt-name" id="selectedPtName">&mdash;</div>
            <div class="selected-pt-sub">5G Live Monitoring</div>
          </div>
        </div>
        <button class="deselect-btn" onclick="deselectPatient()">&#8592; All patients</button>
      </div>

      <!-- Tabs -->
      <div class="tab-bar">
        <button class="tab-btn active" id="tabLiveBtn" onclick="switchTab('live')">📈 Live</button>
        <button class="tab-btn" id="tabHistoryBtn" onclick="switchTab('history')">📊 History</button>
        <button class="tab-btn" id="tabAvgBtn" onclick="switchTab('averages')">📋 Averages</button>
      </div>

      <!-- Live Tab -->
      <div id="liveTab">
        <div class="alert-banner" id="alertBanner">⚠️ Abnormal reading detected!</div>
        <div class="last-update" id="lastUpdate">Waiting for patient data&hellip;</div>
        <div id="monitorCards">
          <div class="waiting">No data yet. Waiting for patient device to send vitals.</div>
        </div>
      </div>

      <!-- History Tab -->
      <div id="historyTab" style="display:none">
        <div class="history-meta">
          <div class="section-title" style="margin:0">Vitals over time</div>
          <span class="history-count-badge" id="historyCountBadge">Loading&hellip;</span>
        </div>
        <div id="historyContent">
          <div class="history-loading">Loading history&hellip;</div>
        </div>
      </div>

      <!-- Averages Tab -->
      <div id="averagesTab" style="display:none">
        <div class="avg-period-bar">
          <button class="avg-period-btn active" id="avgPeriodDay" onclick="setAvgPeriod('day')">Day</button>
          <button class="avg-period-btn" id="avgPeriodWeek" onclick="setAvgPeriod('week')">Week</button>
          <button class="avg-period-btn" id="avgPeriodMonth" onclick="setAvgPeriod('month')">Month</button>
          <button class="avg-period-btn" id="avgPeriodYear" onclick="setAvgPeriod('year')">Year</button>
        </div>
        <div id="avgContent">
          <div class="avg-no-data">Select a patient and open the Averages tab to view statistics.</div>
        </div>
      </div>

    </div>

    <!-- No patient selected -->
    <div class="no-patient-msg" id="noPatientMsg">
      <div style="font-size:40px;margin-bottom:10px;">📈</div>
      Select a patient above to view their vitals.
    </div>

    <div class="sync-row"><div class="sync-dot"></div>5G real-time sync active</div>
  </div>
</div>

<!-- Toast Container -->
<div class="toast-container" id="toastContainer"></div>

<!-- Notification Panel Overlay -->
<div class="notif-panel-overlay" id="notifOverlay" onclick="closeNotifPanel()"></div>

<!-- Notification Panel -->
<div class="notif-panel" id="notifPanel">
  <div class="notif-panel-head">
    <h3>🔔 Notifications</h3>
    <div class="notif-panel-actions">
      <button class="notif-clear-btn" onclick="clearNotifications()">Clear all</button>
      <button class="notif-close-btn" onclick="closeNotifPanel()">&times;</button>
    </div>
  </div>
  <div class="notif-list" id="notifList">
    <div class="notif-empty">No notifications yet.</div>
  </div>
  <div class="notif-sound-row">
    <label><input type="checkbox" id="soundToggle" checked> Play alert sound</label>
  </div>
</div>

<script>
/* ─── Firebase ─── */
const firebaseConfig = {
  apiKey: "AIzaSyCqkYY5YJP6cLZHamuebVuT29PMMojpCNY",
  authDomain: "monitoring-4ac36.firebaseapp.com",
  databaseURL: "https://monitoring-4ac36-default-rtdb.firebaseio.com",
  projectId: "monitoring-4ac36",
  storageBucket: "monitoring-4ac36.firebasestorage.app",
  messagingSenderId: "703860132995",
  appId: "1:703860132995:web:2211861c0f47e17d9fdfb6"
};
firebase.initializeApp(firebaseConfig);
const db = firebase.database();

/* ─── Thresholds ─── */
const THRESHOLDS = {
  hr:   { min: 60,  max: 100,  unit: 'bpm',    label: 'Heart Rate',     color: '#4299e1', colorAlert: '#fc8181' },
  spo2: { min: 94,  max: 100,  unit: '%',       label: 'SpO₂',           color: '#48bb78', colorAlert: '#fc8181' },
  temp: { min: 97,  max: 99.5, unit: '°F',      label: 'Temperature',    color: '#ed8936', colorAlert: '#fc8181' },
  bp:   { min: 90,  max: 140,  unit: 'mmHg',    label: 'Blood Pressure', color: '#9f7aea', colorAlert: '#fc8181' }
};

/* ─── Patient state ─── */
let currentPatientName = '', currentPatientKey = '', currentPatientRef = null;
let _lastHistoryPush = 0;
const HISTORY_INTERVAL = 10000;

/* ─── Doctor state ─── */
let selectedPatientKey = '', allPatientsData = {}, _patientsListenerActive = false;
let activeTab = 'live', _historyListener = null, _chartInstances = {};
let avgPeriod = 'day', _avgHistoryCache = [];

/* ─── Utilities ─── */
function sanitizeName(n) {
  return n.trim().toLowerCase().replace(/[^a-z0-9\s]/g,'').replace(/\s+/g,'_').replace(/^_|_$/g,'') || 'patient';
}
function getInitial(n) { return (n||'?').trim().charAt(0).toUpperCase(); }
function escapeHtml(s) { return String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;'); }
function clamp(v,mn,mx) { return Math.min(mx, Math.max(mn, v)); }

/* ════════════════════════════════════════════
   ── REAL FIREBASE LATENCY MEASUREMENT ──
   ════════════════════════════════════════════ */
let _fivegTimer = null;
let _rttHistory = [];          // last 20 real RTT samples (ms)
let _pingCount  = 0;
let _sparkChart = null;
const PING_PATH = 'latency_ping/doctor';   // ephemeral Firebase node used for pings

function rnd(mn,mx) { return Math.random()*(mx-mn)+mn; }  // kept for simulation engine below

/* ── Run one real Firebase RTT ping ── */
function runPing() {
  const t0 = performance.now();
  const pingRef = db.ref(PING_PATH);
  // Write a tiny payload; measure time until the write is ACK'd by Firebase
  pingRef.set({ t: t0, seq: _pingCount }).then(() => {
    const rtt = Math.round(performance.now() - t0);
    _pingCount++;
    _rttHistory.push(rtt);
    if (_rttHistory.length > 20) _rttHistory.shift();
    updateLatencyPanel(rtt);
  }).catch(() => {
    // Firebase offline — show degraded state
    document.getElementById('rttQuality').textContent = '⚠ Firebase unreachable';
    document.getElementById('fivegNetType').textContent = 'Offline';
  });
}

/* ── Measure data propagation delay from latest patient timestamp ── */
function measurePropagation(patientTimestamp) {
  if (!patientTimestamp) return;
  const delay = Math.max(0, Date.now() - patientTimestamp);
  const el = document.getElementById('propValue');
  if (el) el.textContent = delay < 60000 ? delay : '—';
}

/* ── Update the panel with a new RTT sample ── */
function updateLatencyPanel(rtt) {
  const min = Math.min(..._rttHistory);
  const max = Math.max(..._rttHistory);
  const avg = Math.round(_rttHistory.reduce((a,b)=>a+b,0) / _rttHistory.length);

  // Main RTT value
  const rttEl = document.getElementById('rttValue');
  if (rttEl) {
    rttEl.textContent = rtt;
    rttEl.style.color = rtt < 80 ? '#6ee7b7' : rtt < 200 ? '#fcd34d' : '#f87171';
  }

  // Stats
  const safe = id => document.getElementById(id);
  if (safe('rttMin')) safe('rttMin').textContent = min;
  if (safe('rttAvg')) safe('rttAvg').textContent = avg;
  if (safe('rttMax')) safe('rttMax').textContent = max;
  if (safe('pingCount')) safe('pingCount').textContent = _pingCount;

  // Quality label + signal bars
  let bars, qLabel, qText, netType;
  if (avg < 60)       { bars=5; qLabel='Excellent'; qText='⚡ Excellent — sub-60ms'; netType='Connection: Fast'; }
  else if (avg < 150) { bars=4; qLabel='Good';      qText='✅ Good — under 150ms';  netType='Connection: Good'; }
  else if (avg < 350) { bars=3; qLabel='Fair';      qText='🟡 Fair — ~'+avg+'ms';   netType='Connection: Fair'; }
  else if (avg < 700) { bars=2; qLabel='Poor';      qText='🔴 Poor — '+avg+'ms';    netType='Connection: Poor'; }
  else                { bars=1; qLabel='Bad';        qText='🚨 Very high latency';   netType='Connection: Bad'; }

  if (safe('rttQuality'))      safe('rttQuality').textContent = qText;
  if (safe('connQualityLabel'))safe('connQualityLabel').textContent = qLabel;
  if (safe('fivegNetType'))    safe('fivegNetType').textContent = netType;
  if (safe('lastPingTime'))    safe('lastPingTime').textContent =
    'Last ping: ' + new Date().toLocaleTimeString();

  // Signal bars
  const sigDiv = safe('fivegSignal');
  if (sigDiv) {
    sigDiv.innerHTML = [4,6,9,12,14].map((h,i) =>
      `<div class="fiveg-bar ${i < bars ? 'on':''}" style="height:${h}px"></div>`
    ).join('');
  }

  // Sparkline
  drawSparkline();
}

/* ── Tiny inline sparkline using Canvas ── */
function drawSparkline() {
  const canvas = document.getElementById('latencySparkline');
  if (!canvas || _rttHistory.length < 2) return;
  const dpr = window.devicePixelRatio || 1;
  const W = canvas.offsetWidth, H = 36;
  canvas.width  = W * dpr;
  canvas.height = H * dpr;
  const ctx = canvas.getContext('2d');
  ctx.scale(dpr, dpr);

  const data = _rttHistory;
  const mn = Math.min(...data), mx = Math.max(...data);
  const range = Math.max(mx - mn, 10);
  const pad = 2;
  const xStep = (W - pad*2) / (data.length - 1);

  // Gradient fill
  const grad = ctx.createLinearGradient(0,0,0,H);
  grad.addColorStop(0, 'rgba(167,139,250,0.35)');
  grad.addColorStop(1, 'rgba(167,139,250,0)');

  ctx.beginPath();
  data.forEach((v,i) => {
    const x = pad + i * xStep;
    const y = pad + (1 - (v-mn)/range) * (H - pad*2);
    i === 0 ? ctx.moveTo(x,y) : ctx.lineTo(x,y);
  });
  // Close fill path
  ctx.lineTo(pad + (data.length-1)*xStep, H);
  ctx.lineTo(pad, H);
  ctx.closePath();
  ctx.fillStyle = grad;
  ctx.fill();

  // Line
  ctx.beginPath();
  data.forEach((v,i) => {
    const x = pad + i * xStep;
    const y = pad + (1 - (v-mn)/range) * (H - pad*2);
    i === 0 ? ctx.moveTo(x,y) : ctx.lineTo(x,y);
  });
  ctx.strokeStyle = '#a78bfa';
  ctx.lineWidth = 1.5;
  ctx.lineJoin = 'round';
  ctx.stroke();

  // Latest dot
  const lastX = pad + (data.length-1)*xStep;
  const lastY = pad + (1 - (data[data.length-1]-mn)/range) * (H - pad*2);
  ctx.beginPath();
  ctx.arc(lastX, lastY, 3, 0, Math.PI*2);
  ctx.fillStyle = '#c4b5fd';
  ctx.fill();
}

/* ── Start: run pings every 3 seconds ── */
function startFiveg() {
  runPing(); // immediate first ping
  _fivegTimer = setInterval(runPing, 3000);
}

/* ════════════════════════════════════════════
   ── SIMULATION ENGINE ──
   ════════════════════════════════════════════ */
const SCENARIOS = {
  normal:       { hr:[68,82],   spo2:[96,99],  temp:[98.2,98.8], bp:[110,125], name:'Normal' },
  tachycardia:  { hr:[105,135], spo2:[93,97],  temp:[98.4,99.2], bp:[118,135], name:'Tachycardia' },
  hypertension: { hr:[72,90],   spo2:[95,98],  temp:[98.3,98.9], bp:[148,175], name:'Hypertension' },
  crisis:       { hr:[130,155], spo2:[80,89],  temp:[102,104.5], bp:[180,198], name:'Crisis' },
  recovery:     { hr:[88,100],  spo2:[92,96],  temp:[99.5,101],  bp:[130,148], name:'Recovery' }
};
let _simInterval = null, _currentScenario = 'normal', _simPhase = 0;

function toggleSimulation() {
  const on = document.getElementById('simToggle').checked;
  if (on) {
    startSimulation();
  } else {
    stopSimulation();
    document.getElementById('simStatus').innerHTML = '<span>Simulation stopped. Sliders are manual again.</span>';
  }
}

function setScenario(s) {
  _currentScenario = s;
  document.querySelectorAll('.sim-btn').forEach(b => b.classList.remove('active-sim'));
  document.getElementById('scen'+s.charAt(0).toUpperCase()+s.slice(1))?.classList.add('active-sim');
  if (document.getElementById('simToggle').checked) {
    document.getElementById('simStatus').innerHTML =
      `<div class="live-dot" style="background:#48bb78;width:7px;height:7px;border-radius:50%;display:inline-block;margin-right:4px;animation:blink 1.4s infinite;"></div>
       Simulating: <strong>${SCENARIOS[s].name}</strong>`;
  }
}

function startSimulation() {
  stopSimulation();
  _simPhase = 0;
  setScenario(_currentScenario);
  _simInterval = setInterval(simulateStep, 1200);
  document.getElementById('simStatus').innerHTML =
    `<div class="live-dot" style="background:#48bb78;width:7px;height:7px;border-radius:50%;display:inline-block;margin-right:4px;animation:blink 1.4s infinite;"></div>
     Simulating: <strong>${SCENARIOS[_currentScenario].name}</strong>`;
}

function stopSimulation() {
  if (_simInterval) { clearInterval(_simInterval); _simInterval = null; }
}

function simulateStep() {
  const sc = SCENARIOS[_currentScenario];
  _simPhase += 0.15;

  // Realistic sine-wave variation mimicking physiological rhythms
  const hrBase = (sc.hr[0]+sc.hr[1])/2;
  const hrAmp  = (sc.hr[1]-sc.hr[0])/2;
  const hr = Math.round(clamp(hrBase + hrAmp*Math.sin(_simPhase) + rnd(-1,1), sc.hr[0]-2, sc.hr[1]+2));

  const spo2 = Math.round(clamp(rnd(sc.spo2[0], sc.spo2[1]) + Math.sin(_simPhase*0.7)*0.5, 70, 100));
  const temp = parseFloat(clamp(rnd(sc.temp[0], sc.temp[1]) + Math.sin(_simPhase*0.4)*0.1, 95, 106).toFixed(1));
  const bp   = Math.round(clamp(rnd(sc.bp[0], sc.bp[1]) + Math.sin(_simPhase*0.5)*2, 70, 200));

  // Update sliders + labels
  setSlider('hr', hr, 'hrVal', 'bpm');
  setSlider('spo2', spo2, 'spo2Val', '%');
  document.getElementById('temp').value = temp;
  document.getElementById('tempVal').innerHTML = `${temp} <span class="param-unit">°F</span>`;
  setSlider('bp', bp, 'bpVal', 'mmHg');

  scheduleSync();
}

function setSlider(id, val, valId, unit) {
  document.getElementById(id).value = val;
  document.getElementById(valId).innerHTML = `${val} <span class="param-unit">${unit}</span>`;
}

/* ════════════════════════════════════════════
   ── DEMO PATIENTS (Doctor side) ──
   ════════════════════════════════════════════ */
const DEMO_PATIENTS = {
  A: { key:'demo_alice_normal', name:'Alice – Normal', hr:74, spo2:98, temp:98.5, bp:118 },
  B: { key:'demo_bob_icu',      name:'Bob – ICU',      hr:108, spo2:91, temp:99.8, bp:145 },
  C: { key:'demo_carol_critical', name:'Carol – Critical', hr:142, spo2:82, temp:103.2, bp:185 }
};
let _demoIntervals = {};
let _activeDemos = new Set();

function activateDemoPatient(id) {
  const btn = document.getElementById('demo'+id);
  if (_activeDemos.has(id)) {
    // Deactivate
    _activeDemos.delete(id);
    btn.classList.remove('active-demo');
    if (_demoIntervals[id]) { clearInterval(_demoIntervals[id]); delete _demoIntervals[id]; }
    return;
  }
  _activeDemos.add(id);
  btn.classList.add('active-demo');
  const p = DEMO_PATIENTS[id];
  pushDemoPatient(p);
  _demoIntervals[id] = setInterval(() => pushDemoPatient(p), 800);
}

function pushDemoPatient(p) {
  const data = {
    patientName: p.name,
    hr:    Math.round(clamp(p.hr   + rnd(-4, 4),   0, 250)),
    spo2:  Math.round(clamp(p.spo2 + rnd(-3, 3),   0, 100)),
    temp:  parseFloat(clamp(p.temp + rnd(-0.3, 0.3), 0, 115).toFixed(1)),
    bp:    Math.round(clamp(p.bp   + rnd(-4, 4),   0, 250)),
    timestamp: Date.now()
  };
  db.ref('patients/' + p.key).set(data);
  // Also push to history
  db.ref('patientHistory/' + p.key).push({
    hr: data.hr, spo2: data.spo2, temp: data.temp, bp: data.bp, timestamp: data.timestamp
  });
}

/* ════════════════════════════════════════════
   ── MODE SWITCHING ──
   ════════════════════════════════════════════ */
function startApp(mode) {
  document.getElementById('modeSelect').style.display = 'none';
  if (mode === 'patient') {
    document.getElementById('patientNameScreen').classList.add('active');
    setTimeout(() => document.getElementById('patientNameInput').focus(), 100);
  } else {
    document.getElementById('monitorApp').classList.add('active');
    checkConn('mConn');
    initNotifBanner();
    listenAllPatients();
    startFiveg();
  }
}

function goBack() {
  stopSimulation();
  if (_fivegTimer) { clearInterval(_fivegTimer); _fivegTimer = null; }
  Object.values(_demoIntervals).forEach(clearInterval);
  _demoIntervals = {}; _activeDemos.clear();
  if (currentPatientRef)       { currentPatientRef.off(); currentPatientRef = null; }
  if (_patientsListenerActive) { db.ref('patients').off(); _patientsListenerActive = false; }
  if (_historyListener)        { _historyListener.off(); _historyListener = null; }
  destroyCharts();
  currentPatientName=''; currentPatientKey='';
  selectedPatientKey=''; allPatientsData={};
  _prevAlertKeys=[]; _alertedVitals.clear(); activeTab='live';
  avgPeriod='day'; _avgHistoryCache=[];

  document.getElementById('patientNameInput').value='';
  document.getElementById('nameError').textContent='';
  document.getElementById('patientNameInput').classList.remove('error');
  document.getElementById('patientNameScreen').classList.remove('active');
  document.getElementById('patientApp').classList.remove('active');
  document.getElementById('monitorApp').classList.remove('active');
  document.getElementById('modeSelect').style.display='flex';
  document.getElementById('simToggle').checked = false;
  document.getElementById('simStatus').innerHTML = '<span>Enable auto-simulate to generate realistic sensor-like readings</span>';
  document.querySelectorAll('.sim-btn').forEach(b => b.classList.remove('active-sim'));
  document.querySelectorAll('.demo-pt-btn').forEach(b => b.classList.remove('active-demo'));
}

function checkConn(id) {
  db.ref('.info/connected').on('value', snap => {
    const b = document.getElementById(id);
    if (!b) return;
    b.textContent = snap.val() ? 'Connected' : 'Offline';
    b.className = 'conn-badge ' + (snap.val() ? 'conn-ok' : 'conn-off');
  });
}

/* ─── Patient: name entry ─── */
function clearNameError() {
  document.getElementById('nameError').textContent='';
  document.getElementById('patientNameInput').classList.remove('error');
}
function confirmPatientName() {
  const raw = document.getElementById('patientNameInput').value.trim();
  if (!raw) {
    document.getElementById('nameError').textContent='Please enter your name.';
    document.getElementById('patientNameInput').classList.add('error');
    document.getElementById('patientNameInput').focus();
    return;
  }
  currentPatientName = raw;
  currentPatientKey  = sanitizeName(raw);
  currentPatientRef  = db.ref('patients/' + currentPatientKey);
  document.getElementById('patientNameBadge').textContent = raw;
  document.getElementById('patientNameScreen').classList.remove('active');
  document.getElementById('patientApp').classList.add('active');
  checkConn('pConn');
  pushVitals();
}

/* ─── Patient: sliders ─── */
function updateSlider(id, valId, unit) {
  const v = document.getElementById(id).value;
  document.getElementById(valId).innerHTML = v + ' <span class="param-unit">' + unit + '</span>';
}
function updateTempSlider() {
  const v = parseFloat(document.getElementById('temp').value).toFixed(1);
  document.getElementById('tempVal').innerHTML = v + ' <span class="param-unit">°F</span>';
}

let _syncTimer = null;
function scheduleSync() { clearTimeout(_syncTimer); _syncTimer = setTimeout(pushVitals, 300); }

function pushVitals() {
  if (!currentPatientRef) return;
  const now = Date.now();
  const data = {
    patientName: currentPatientName,
    hr:    parseInt(document.getElementById('hr').value),
    spo2:  parseInt(document.getElementById('spo2').value),
    temp:  parseFloat(parseFloat(document.getElementById('temp').value).toFixed(1)),
    bp:    parseInt(document.getElementById('bp').value),
    timestamp: now
  };
  currentPatientRef.set(data).then(() => {
    const el = document.getElementById('patientSyncTime');
    if (el) el.textContent = new Date().toLocaleTimeString();
  }).catch(e => console.error('Sync error:', e.message));

  if (now - _lastHistoryPush >= HISTORY_INTERVAL) {
    _lastHistoryPush = now;
    db.ref('patientHistory/' + currentPatientKey).push({
      hr: data.hr, spo2: data.spo2, temp: data.temp, bp: data.bp, timestamp: now
    });
  }
}

/* ─── Doctor: patient list ─── */
function listenAllPatients() {
  if (_patientsListenerActive) return;
  _patientsListenerActive = true;
  db.ref('patients').on('value', snap => {
    allPatientsData = snap.val() || {};
    renderPatientList();
    if (selectedPatientKey && allPatientsData[selectedPatientKey]) {
      renderMonitor(allPatientsData[selectedPatientKey]);
    }
  });
}

function getRiskScore(d) {
  let score = 0;
  for (const [k, cfg] of Object.entries(THRESHOLDS)) {
    const v = d[k]; if (v == null) continue;
    const range = cfg.max - cfg.min;
    if (v < cfg.min) score += Math.min(40, ((cfg.min - v) / range) * 50);
    else if (v > cfg.max) score += Math.min(40, ((v - cfg.max) / range) * 50);
  }
  return Math.min(100, Math.round(score));
}

function getRiskLabel(score) {
  if (score < 20) return { label:'Low Risk', cls:'risk-low' };
  if (score < 50) return { label:'Moderate', cls:'risk-mod' };
  return { label:'High Risk', cls:'risk-high' };
}

function getAlertKeys(d) {
  return Object.keys(THRESHOLDS).filter(k => {
    const v = d[k], c = THRESHOLDS[k];
    return v != null && (v < c.min || v > c.max);
  });
}

function renderPatientList() {
  const container = document.getElementById('patientsList');
  if (!container) return;
  const keys = Object.keys(allPatientsData);
  if (keys.length === 0) { container.innerHTML = '<div class="waiting">No patients connected yet.</div>'; return; }
  container.innerHTML = keys.map(key => {
    const d = allPatientsData[key];
    const name = d.patientName || key;
    const isStale = Date.now() - (d.timestamp||0) > 60000;
    const alertKeys = isStale ? [] : getAlertKeys(d);
    const hasAlert = alertKeys.length > 0;
    const isSel = key === selectedPatientKey;
    const score = isStale ? null : getRiskScore(d);
    const risk = score != null ? getRiskLabel(score) : null;
    let sc = isSel ? 'selected' : '';
    if (hasAlert) sc += ' alerted';
    const timeStr = d.timestamp ? new Date(d.timestamp).toLocaleTimeString() : 'Never';
    // Sub-label for demo patients
    const isDemo = key.startsWith('demo_');
    const subLabel = isDemo ? '<div class="patient-item-sub">🧪 Demo scenario</div>' : '';
    return `<div class="patient-item ${sc}" onclick="selectPatient('${escapeHtml(key)}')">
      <div class="patient-avatar">${getInitial(name)}</div>
      <div class="patient-info">
        <div class="patient-item-name">${escapeHtml(name)}</div>
        <div class="patient-item-time">Last seen: ${timeStr}</div>
        ${subLabel}
      </div>
      ${risk ? `<span class="risk-badge ${risk.cls}">${risk.label}</span>` : `<span class="pill pill-alert">Offline</span>`}
    </div>`;
  }).join('');
}

function selectPatient(key) {
  selectedPatientKey = key;
  _prevAlertKeys=[]; _alertedVitals.clear();
  activeTab='live'; avgPeriod='day'; _avgHistoryCache=[];
  _historyInitialized = false; _avgInitialized = false;
  const d = allPatientsData[key]||{};
  const name = d.patientName||key;
  document.getElementById('selectedPtAvatar').textContent = getInitial(name);
  document.getElementById('selectedPtName').textContent = name;
  document.getElementById('selectedVitalsArea').style.display='block';
  document.getElementById('noPatientMsg').style.display='none';
  document.getElementById('liveTab').style.display='block';
  document.getElementById('historyTab').style.display='none';
  document.getElementById('averagesTab').style.display='none';
  document.getElementById('tabLiveBtn').classList.add('active');
  document.getElementById('tabHistoryBtn').classList.remove('active');
  document.getElementById('tabAvgBtn').classList.remove('active');
  renderPatientList();
  if (d.hr != null) renderMonitor(d);
}

function deselectPatient() {
  selectedPatientKey='';
  _prevAlertKeys=[]; _alertedVitals.clear();
  _historyInitialized = false; _avgInitialized = false;
  destroyCharts();
  if (_historyListener) { _historyListener.off(); _historyListener=null; }
  document.getElementById('selectedVitalsArea').style.display='none';
  document.getElementById('noPatientMsg').style.display='block';
  document.getElementById('alertBanner').classList.remove('show');
  renderPatientList();
}

/* ─── Tabs ─── */
function switchTab(tab) {
  activeTab = tab;
  document.getElementById('liveTab').style.display    = tab==='live'     ? 'block' : 'none';
  document.getElementById('historyTab').style.display = tab==='history'  ? 'block' : 'none';
  document.getElementById('averagesTab').style.display= tab==='averages' ? 'block' : 'none';
  document.getElementById('tabLiveBtn').classList.toggle('active',    tab==='live');
  document.getElementById('tabHistoryBtn').classList.toggle('active', tab==='history');
  document.getElementById('tabAvgBtn').classList.toggle('active',     tab==='averages');
  if (tab==='history') loadHistory(selectedPatientKey);
  if (tab==='averages') loadAverages(selectedPatientKey);
}

/* ─── History ─── */
let _historyInitialized = false;

function loadHistory(key) {
  if (!key) return;
  // Only show loading state the very first time
  if (!_historyInitialized) {
    document.getElementById('historyContent').innerHTML='<div class="history-loading">Loading history…</div>';
    document.getElementById('historyCountBadge').textContent='Loading…';
  }
  if (_historyListener) { _historyListener.off(); _historyListener=null; }
  _historyInitialized = false;
  const ref = db.ref('patientHistory/'+key).limitToLast(40);
  _historyListener = ref;
  ref.on('value', snap => {
    const raw = snap.val();
    if (!raw) {
      document.getElementById('historyContent').innerHTML='<div class="waiting">No history yet.</div>';
      document.getElementById('historyCountBadge').textContent='0 readings';
      _historyInitialized = false;
      return;
    }
    const entries = Object.values(raw).sort((a,b)=>a.timestamp-b.timestamp);
    document.getElementById('historyCountBadge').textContent=entries.length+' readings';
    if (!_historyInitialized) {
      buildHistoryCharts(entries);
      _historyInitialized = true;
    } else {
      updateHistoryCharts(entries);
    }
    _avgHistoryCache = entries;
  });
}

function buildHistoryCharts(entries) {
  // Build DOM + Chart.js instances once
  destroyCharts();
  const labels = entries.map(e => new Date(e.timestamp).toLocaleTimeString([],{hour:'2-digit',minute:'2-digit',second:'2-digit'}));
  const content = document.getElementById('historyContent');
  content.innerHTML='';
  Object.entries(THRESHOLDS).forEach(([key,cfg]) => {
    const values = entries.map(e => e[key]!=null ? (key==='temp'?parseFloat(e[key].toFixed(1)):e[key]) : null);
    const lastVal = values[values.length-1];
    const isAlert = lastVal!=null && (lastVal<cfg.min||lastVal>cfg.max);
    const lineColor = isAlert ? cfg.colorAlert : cfg.color;
    const fillColor = isAlert ? 'rgba(252,129,129,0.08)' : hexToRgba(cfg.color,0.08);
    const displayVal = lastVal!=null?(key==='temp'?lastVal.toFixed(1):lastVal):'—';
    const n = entries.length;
    const card = document.createElement('div');
    card.className='chart-card'+(isAlert?' alert':'');
    card.id='chartcard-'+key;
    card.innerHTML=`<div class="chart-card-header">
      <div class="chart-card-left">
        <div class="chart-card-label">${cfg.label}</div>
        <div class="chart-card-value" id="chartval-${key}">${displayVal} <span>${cfg.unit}</span></div>
      </div>
      <span class="pill ${isAlert?'pill-alert':'pill-ok'}" id="chartpill-${key}">${isAlert?'Alert':'Normal'}</span>
    </div>
    <div class="chart-canvas-wrap"><canvas id="chart-${key}"></canvas></div>
    <div class="chart-range-hint">Safe: ${cfg.min}–${cfg.max} ${cfg.unit}</div>`;
    content.appendChild(card);
    const ctx = document.getElementById('chart-'+key).getContext('2d');
    _chartInstances[key] = new Chart(ctx, {
      type:'line', data:{ labels, datasets:[
        { label:cfg.label, data:values, borderColor:lineColor, backgroundColor:fillColor, fill:true, tension:0.35,
          pointRadius:n>20?0:3, pointHoverRadius:5, borderWidth:2, spanGaps:true },
        { label:'Max', data:Array(n).fill(cfg.max), borderColor:'rgba(252,129,129,0.45)', borderDash:[5,4], borderWidth:1, pointRadius:0, fill:false, tension:0 },
        { label:'Min', data:Array(n).fill(cfg.min), borderColor:'rgba(252,129,129,0.45)', borderDash:[5,4], borderWidth:1, pointRadius:0, fill:false, tension:0 }
      ]},
      options:{ responsive:true, maintainAspectRatio:false, animation:{duration:300},
        interaction:{mode:'index',intersect:false},
        plugins:{ legend:{display:false},
          tooltip:{ callbacks:{
            title:items=>items[0].label,
            label:item=>{ if(item.datasetIndex===0) return ` ${item.parsed.y} ${cfg.unit}`; if(item.datasetIndex===1) return ` Max: ${cfg.max} ${cfg.unit}`; return ` Min: ${cfg.min} ${cfg.unit}`; }
          }, backgroundColor:'rgba(26,32,44,0.85)', padding:8, cornerRadius:8, titleFont:{size:11}, bodyFont:{size:12,weight:'600'} }
        },
        scales:{ x:{ display:n<=15, ticks:{font:{size:9,family:'DM Mono'},color:'#a0aec0',maxRotation:0,autoSkip:true,maxTicksLimit:6}, grid:{color:'#f0f4f8'} },
          y:{ ticks:{font:{size:10,family:'DM Mono'},color:'#a0aec0',padding:4}, grid:{color:'#f0f4f8'} } }
      }
    });
  });
}

function updateHistoryCharts(entries) {
  // Only update data inside existing charts — no DOM rebuild
  const labels = entries.map(e => new Date(e.timestamp).toLocaleTimeString([],{hour:'2-digit',minute:'2-digit',second:'2-digit'}));
  const n = entries.length;
  Object.entries(THRESHOLDS).forEach(([key,cfg]) => {
    const chart = _chartInstances[key];
    if (!chart) return;
    const values = entries.map(e => e[key]!=null ? (key==='temp'?parseFloat(e[key].toFixed(1)):e[key]) : null);
    const lastVal = values[values.length-1];
    const isAlert = lastVal!=null && (lastVal<cfg.min||lastVal>cfg.max);
    const lineColor = isAlert ? cfg.colorAlert : cfg.color;
    const fillColor = isAlert ? 'rgba(252,129,129,0.08)' : hexToRgba(cfg.color,0.08);

    // Update chart data in place
    chart.data.labels = labels;
    chart.data.datasets[0].data = values;
    chart.data.datasets[0].borderColor = lineColor;
    chart.data.datasets[0].backgroundColor = fillColor;
    chart.data.datasets[0].pointRadius = n>20?0:3;
    chart.data.datasets[1].data = Array(n).fill(cfg.max);
    chart.data.datasets[2].data = Array(n).fill(cfg.min);
    chart.options.scales.x.display = n<=15;
    chart.update('none'); // 'none' = no animation, instant update

    // Update only the value text and pill — no layout change
    const displayVal = lastVal!=null?(key==='temp'?lastVal.toFixed(1):lastVal):'—';
    const valEl = document.getElementById('chartval-'+key);
    const pillEl = document.getElementById('chartpill-'+key);
    const cardEl = document.getElementById('chartcard-'+key);
    if (valEl) valEl.innerHTML = `${displayVal} <span>${cfg.unit}</span>`;
    if (pillEl) { pillEl.textContent = isAlert?'Alert':'Normal'; pillEl.className='pill '+(isAlert?'pill-alert':'pill-ok'); }
    if (cardEl) cardEl.className='chart-card'+(isAlert?' alert':'');
  });
}

function destroyCharts() {
  Object.values(_chartInstances).forEach(c => { try { c.destroy(); } catch(e){} });
  _chartInstances={};
}
function hexToRgba(hex,alpha) {
  const r=parseInt(hex.slice(1,3),16),g=parseInt(hex.slice(3,5),16),b=parseInt(hex.slice(5,7),16);
  return `rgba(${r},${g},${b},${alpha})`;
}

/* ════════════════════════════════════════════
   ── AVERAGES TAB ──
   ════════════════════════════════════════════ */
function setAvgPeriod(p) {
  avgPeriod = p;
  _avgInitialized = false;
  ['day','week','month','year'].forEach(x => {
    document.getElementById('avgPeriod'+x.charAt(0).toUpperCase()+x.slice(1))
      .classList.toggle('active', x===p);
  });
  loadAverages(selectedPatientKey);
}

let _avgInitialized = false;

function loadAverages(key) {
  if (!key) return;
  // Show loading only first time
  if (!_avgInitialized) {
    document.getElementById('avgContent').innerHTML='<div class="avg-no-data">Loading…</div>';
  }
  const limits = { day:200, week:500, month:1000, year:2000 };
  const limit = limits[avgPeriod] || 200;

  db.ref('patientHistory/'+key).limitToLast(limit).once('value', snap => {
    const raw = snap.val();
    if (!raw) {
      if (!_avgInitialized) {
        document.getElementById('avgContent').innerHTML=
          `<div class="avg-no-data">No history data yet.<br><small>Keep the patient device active to accumulate readings.</small></div>`;
      }
      return;
    }
    const entries = Object.values(raw).sort((a,b)=>a.timestamp-b.timestamp);
    _avgHistoryCache = entries;
    const now = Date.now();
    const windows = { day:86400000, week:604800000, month:2592000000, year:31536000000 };
    const filtered = entries.filter(e => (now - e.timestamp) <= windows[avgPeriod]);

    if (filtered.length === 0) {
      if (!_avgInitialized) {
        document.getElementById('avgContent').innerHTML=
          `<div class="avg-no-data">No data in this period.<br><small>Try a shorter period or collect more readings.</small></div>`;
      }
      return;
    }

    if (!_avgInitialized) {
      buildAvgTable(filtered);
      _avgInitialized = true;
    } else {
      updateAvgTable(filtered);
    }
  });
}

function buildAvgTable(filtered) {
  // Build the full DOM structure once — never rebuilt again
  const content = document.getElementById('avgContent');
  content.innerHTML = '';

  const periodLabels = { day:'Past 24 Hours', week:'Past 7 Days', month:'Past 30 Days', year:'Past Year' };
  const infoDiv = document.createElement('div');
  infoDiv.id = 'avgInfoLine';
  infoDiv.style.cssText='font-size:11px;color:#a0aec0;margin-bottom:14px;text-align:center;';
  infoDiv.textContent = `${periodLabels[avgPeriod]} · ${filtered.length} readings`;
  content.appendChild(infoDiv);

  Object.entries(THRESHOLDS).forEach(([key, cfg]) => {
    const card = document.createElement('div');
    card.className = 'avg-vital-card';
    card.id = 'avgcard-'+key;
    card.innerHTML = `
      <div class="avg-vital-header">
        <div class="avg-vital-label">${cfg.label}</div>
        <span class="pill pill-ok" id="avgpill-${key}">Avg Normal</span>
      </div>
      <div class="avg-stats-row">
        <div class="avg-stat">
          <div class="avg-stat-val" id="avgval-avg-${key}">—</div>
          <div class="avg-stat-label">Average<br><span style="color:#cbd5e0">${cfg.unit}</span></div>
        </div>
        <div class="avg-stat">
          <div class="avg-stat-val" id="avgval-min-${key}">—</div>
          <div class="avg-stat-label">Minimum<br><span style="color:#cbd5e0">${cfg.unit}</span></div>
        </div>
        <div class="avg-stat">
          <div class="avg-stat-val" id="avgval-max-${key}">—</div>
          <div class="avg-stat-label">Maximum<br><span style="color:#cbd5e0">${cfg.unit}</span></div>
        </div>
      </div>
      <div class="avg-trend">
        <span id="avgtrendicon-${key}" style="font-size:16px;font-weight:700" class="trend-flat">→</span>
        <span>Trend: <strong id="avgtrendlabel-${key}">Stable</strong></span>
        <span style="margin-left:auto;font-family:'DM Mono',monospace;font-size:10px;color:#cbd5e0">
          Safe: ${cfg.min}–${cfg.max} ${cfg.unit}
        </span>
      </div>`;
    content.appendChild(card);
  });

  // Fill values immediately after building
  updateAvgTable(filtered);
}

function updateAvgTable(filtered) {
  // Only update text content — zero DOM restructuring
  const periodLabels = { day:'Past 24 Hours', week:'Past 7 Days', month:'Past 30 Days', year:'Past Year' };
  const infoEl = document.getElementById('avgInfoLine');
  if (infoEl) infoEl.textContent = `${periodLabels[avgPeriod]} · ${filtered.length} readings`;

  Object.entries(THRESHOLDS).forEach(([key, cfg]) => {
    const vals = filtered.map(e => e[key]).filter(v => v != null);
    if (!vals.length) return;

    const avg  = vals.reduce((a,b)=>a+b,0)/vals.length;
    const minV = Math.min(...vals);
    const maxV = Math.max(...vals);
    const fmt  = v => key==='temp' ? v.toFixed(1) : Math.round(v);

    const isAvgAlert = avg  < cfg.min || avg  > cfg.max;
    const isMinAlert = minV < cfg.min;
    const isMaxAlert = maxV > cfg.max;

    const half = Math.floor(vals.length/2);
    let trendIcon='→', trendLabel='Stable', trendCls='trend-flat';
    if (half > 0) {
      const fa = vals.slice(0,half).reduce((a,b)=>a+b,0)/half;
      const sa = vals.slice(half).reduce((a,b)=>a+b,0)/(vals.length-half);
      const diff = sa - fa, thresh = (cfg.max-cfg.min)*0.03;
      if (diff > thresh)       { trendIcon='↑'; trendLabel='Rising';    trendCls='trend-up'; }
      else if (diff < -thresh) { trendIcon='↓'; trendLabel='Declining'; trendCls='trend-down'; }
    }

    // Update individual elements — layout stays intact
    const avgEl  = document.getElementById('avgval-avg-'+key);
    const minEl  = document.getElementById('avgval-min-'+key);
    const maxEl  = document.getElementById('avgval-max-'+key);
    const pillEl = document.getElementById('avgpill-'+key);
    const iconEl = document.getElementById('avgtrendicon-'+key);
    const lblEl  = document.getElementById('avgtrendlabel-'+key);

    if (avgEl)  { avgEl.textContent  = fmt(avg);  avgEl.className  = 'avg-stat-val'+(isAvgAlert?' alert-val':''); }
    if (minEl)  { minEl.textContent  = fmt(minV); minEl.className  = 'avg-stat-val'+(isMinAlert?' alert-val':''); }
    if (maxEl)  { maxEl.textContent  = fmt(maxV); maxEl.className  = 'avg-stat-val'+(isMaxAlert?' alert-val':''); }
    if (pillEl) { pillEl.textContent = isAvgAlert?'Avg Alert':'Avg Normal'; pillEl.className='pill '+(isAvgAlert?'pill-alert':'pill-ok'); }
    if (iconEl) { iconEl.textContent = trendIcon; iconEl.className = trendCls; }
    if (lblEl)  { lblEl.textContent  = trendLabel; }
  });
}

/* ─── Monitor: render live vitals ─── */
function renderMonitor(d) {
  let anyAlert=false, alertKeys=[], html='';
  for (const [key,cfg] of Object.entries(THRESHOLDS)) {
    const val = d[key];
    const isAlert = val!=null && (val<cfg.min||val>cfg.max);
    if (isAlert) { anyAlert=true; alertKeys.push(key); }
    const display = (key==='temp') ? (val!=null?val.toFixed(1):'—') : (val??'—');
    html+=`<div class="vital-card ${isAlert?'alert':''}">
      <div>
        <div class="vital-label">${cfg.label}</div>
        <span class="vital-num">${display}</span>
        <span class="vital-unit-lbl">${cfg.unit}</span>
      </div>
      <span class="pill ${isAlert?'pill-alert':'pill-ok'}">${isAlert?'Alert':'Normal'}</span>
    </div>`;
  }
  document.getElementById('monitorCards').innerHTML=html;
  document.getElementById('lastUpdate').textContent='Last update: '+new Date(d.timestamp).toLocaleTimeString();
  measurePropagation(d.timestamp);
  const banner = document.getElementById('alertBanner');
  if (anyAlert) {
    banner.textContent='⚠ Alert: abnormal '+alertKeys.map(k=>THRESHOLDS[k].label).join(', ');
    banner.classList.add('show');
  } else banner.classList.remove('show');
  maybeNotify(alertKeys);
  handleInAppNotifications(alertKeys, d);
  if (activeTab==='history') loadHistory(selectedPatientKey);
  if (activeTab==='averages') loadAverages(selectedPatientKey);
}

/* ─── Notifications ─── */
const _alertedVitals = new Set();
function initNotifBanner() {
  const banner=document.getElementById('notifBanner'), text=document.getElementById('notifText'), btn=document.getElementById('notifBtn');
  if (!('Notification' in window)) { banner.classList.add('denied'); text.textContent='⚠ This browser does not support notifications.'; btn.style.display='none'; return; }
  if (Notification.permission==='granted') showGrantedState();
  else if (Notification.permission==='denied') showDeniedState();
}
function requestNotifPermission() {
  if (!('Notification' in window)) return;
  Notification.requestPermission().then(p => {
    if (p==='granted') { showGrantedState(); new Notification('Advanced Hospital Monitor',{body:'✅ Notifications enabled.'}); }
    else showDeniedState();
  });
}
function showGrantedState() { const b=document.getElementById('notifBanner'); b.classList.remove('denied'); b.classList.add('granted'); document.getElementById('notifText').textContent='🔔 Notifications enabled.'; document.getElementById('notifBtn').style.display='none'; }
function showDeniedState() { const b=document.getElementById('notifBanner'); b.classList.remove('granted'); b.classList.add('denied'); document.getElementById('notifText').textContent='🔕 Notifications blocked. Enable in browser settings.'; document.getElementById('notifBtn').style.display='none'; }
function maybeNotify(alertedKeys) {
  if (!('Notification' in window)||Notification.permission!=='granted') return;
  const newAlerts=alertedKeys.filter(k=>!_alertedVitals.has(k));
  const cleared=[..._alertedVitals].filter(k=>!alertedKeys.includes(k));
  cleared.forEach(k=>_alertedVitals.delete(k)); newAlerts.forEach(k=>_alertedVitals.add(k));
  if (newAlerts.length===0) return;
  const pName=(allPatientsData[selectedPatientKey]||{}).patientName||selectedPatientKey;
  new Notification('⚠ Alert — '+pName,{body:'Abnormal: '+newAlerts.map(k=>THRESHOLDS[k].label).join(', '),tag:'vitals-alert',renotify:true,requireInteraction:true});
}

const _notifLog=[];
let _unreadCount=0, _prevAlertKeys=[];

function handleInAppNotifications(alertKeys,d) {
  const newAlerts=alertKeys.filter(k=>!_prevAlertKeys.includes(k));
  const resolved=_prevAlertKeys.filter(k=>!alertKeys.includes(k));
  _prevAlertKeys=[...alertKeys];
  const pName=d.patientName||selectedPatientKey||'Patient';
  newAlerts.forEach(k=>{ const cfg=THRESHOLDS[k]; const val=(k==='temp')?d[k].toFixed(1):d[k]; const body=`${pName} — ${cfg.label}: ${val} ${cfg.unit} (normal: ${cfg.min}–${cfg.max})`; addNotif('alert','⚠ Abnormal vital',body); showToast('alert','⚠ Alert',body); if(document.getElementById('soundToggle')?.checked) playAlertSound(); });
  resolved.forEach(k=>{ const cfg=THRESHOLDS[k]; const val=(k==='temp')?d[k].toFixed(1):d[k]; addNotif('resolve','✅ Vital normalized',`${pName} — ${cfg.label} back to normal: ${val} ${cfg.unit}`); showToast('resolve','✅ Resolved',`${pName} — ${cfg.label} normalized`); });
}
function addNotif(type,title,body) { _notifLog.unshift({id:Date.now()+Math.random(),type,title,body,time:new Date().toLocaleTimeString()}); _unreadCount++; updateBadge(); renderNotifPanel(); }
function updateBadge() { const badge=document.getElementById('notifBadge'),bell=document.getElementById('notifBellBtn'); if(!badge||!bell) return; if(_unreadCount>0){badge.textContent=_unreadCount>99?'99+':_unreadCount;badge.classList.add('show');bell.classList.add('has-alerts');}else{badge.classList.remove('show');bell.classList.remove('has-alerts');} }
function renderNotifPanel() { const list=document.getElementById('notifList'); if(!list) return; list.innerHTML=_notifLog.length===0?'<div class="notif-empty">No notifications yet.</div>':_notifLog.map(n=>`<div class="notif-item ${n.type==='resolve'?'resolved':''}"><div class="notif-item-title">${n.title}</div><div class="notif-item-body">${escapeHtml(n.body)}</div><div class="notif-item-time">${n.time}</div></div>`).join(''); }
function toggleNotifPanel() { const panel=document.getElementById('notifPanel'),ov=document.getElementById('notifOverlay'); if(panel.classList.contains('open')){closeNotifPanel();}else{panel.classList.add('open');ov.classList.add('open');_unreadCount=0;updateBadge();} }
function closeNotifPanel() { document.getElementById('notifPanel')?.classList.remove('open'); document.getElementById('notifOverlay')?.classList.remove('open'); }
function clearNotifications() { _notifLog.length=0;_unreadCount=0;updateBadge();renderNotifPanel(); }

function showToast(type,title,body) {
  const c=document.getElementById('toastContainer'); if(!c) return;
  const id='toast-'+Date.now(); const d=document.createElement('div');
  d.className='toast'+(type==='resolve'?' resolve':''); d.id=id;
  d.innerHTML=`<span class="toast-icon">${type==='resolve'?'✅':'⚠'}</span><div class="toast-content"><div class="toast-title">${title}</div><div class="toast-body">${escapeHtml(body)}</div></div><button class="toast-close" onclick="dismissToast('${id}')">&times;</button>`;
  c.appendChild(d); setTimeout(()=>dismissToast(id),5000);
}
function dismissToast(id) { const el=document.getElementById(id); if(!el) return; el.style.animation='toastOut 0.3s ease forwards'; setTimeout(()=>el.remove(),300); }

function playAlertSound() {
  try {
    const ctx=new(window.AudioContext||window.webkitAudioContext)();
    [0,200,400].forEach(delay=>{ const osc=ctx.createOscillator(),gain=ctx.createGain(); osc.connect(gain);gain.connect(ctx.destination); osc.type='sine';osc.frequency.value=880; gain.gain.setValueAtTime(0.3,ctx.currentTime+delay/1000); gain.gain.exponentialRampToValueAtTime(0.001,ctx.currentTime+delay/1000+0.18); osc.start(ctx.currentTime+delay/1000); osc.stop(ctx.currentTime+delay/1000+0.18); });
  } catch(e){}
}
</script>
</body>
</html>
