<!DOCTYPE html>
<html lang="it">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>WorkTrack – Guadagno Reale (MVP)</title>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800&display=swap" rel="stylesheet">
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    :root { --ring: 0 0% 100%; }
    html, body { height: 100%; }
    body { font-family: 'Inter', system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, 'Helvetica Neue', Arial, sans-serif; }
  </style>
  <!-- PWA hooks -->
  <link rel="manifest" href="/manifest.json">
  <meta name="theme-color" content="#0f172a">
  <script>
    if ('serviceWorker' in navigator) {
      window.addEventListener('load', () => {
        navigator.serviceWorker.register('/sw.js').catch(console.error);
      });
    }
  </script>
</head>
<body class="min-h-screen bg-slate-950 text-slate-100">
  <!-- HEADER -->
  <header class="sticky top-0 z-30 backdrop-blur supports-[backdrop-filter]:bg-slate-950/70 bg-slate-900/60 border-b border-slate-800">
    <div class="max-w-4xl mx-auto px-4 sm:px-6 py-4 flex items-center justify-between">
      <div class="flex items-center gap-3">
        <div class="text-2xl">📊</div>
        <h1 class="text-xl sm:text-2xl font-extrabold tracking-tight">WorkTrack – Guadagno Reale</h1>
      </div>
      <div class="flex items-center gap-2">
        <button id="shareBtn" class="hidden sm:inline-flex px-3 py-2 rounded-xl border border-slate-800 hover:bg-slate-800/60 transition text-sm">Condividi</button>
        <button id="settingsBtn" class="px-3 py-2 rounded-xl bg-indigo-600 hover:bg-indigo-500 transition text-sm font-semibold">Impostazioni</button>
      </div>
    </div>
  </header>

  <!-- MAIN -->
  <main class="max-w-4xl mx-auto px-4 sm:px-6 py-8 grid gap-6">
    <!-- HERO / STATUS BAR -->
    <section class="grid gap-4">
      <div class="flex flex-col sm:flex-row items-start sm:items-center justify-between gap-3">
        <p class="text-slate-300">Vedi crescere il tuo stipendio in <span class="font-semibold text-white">tempo reale</span>.</p>
        <div class="inline-flex items-center gap-2 text-xs text-slate-400">
          <span class="h-2 w-2 rounded-full bg-emerald-500 animate-pulse"></span>
          <span id="statusText">Pronto</span>
        </div>
      </div>
      <div class="grid sm:grid-cols-2 gap-4">
        <div class="rounded-2xl border border-slate-800 bg-slate-900/60 p-5">
          <div class="text-sm text-slate-400">Guadagno Oggi</div>
          <div id="earnToday" class="text-4xl sm:text-5xl font-extrabold mt-2 tracking-tight">£0.00</div>
          <div class="mt-3 text-xs text-slate-400" id="earnTodayNet">Netto stimato: £0.00</div>
        </div>
        <div class="rounded-2xl border border-slate-800 bg-slate-900/60 p-5">
          <div class="text-sm text-slate-400">Questa Settimana</div>
          <div id="earnWeek" class="text-4xl sm:text-5xl font-extrabold mt-2 tracking-tight">£0.00</div>
          <div class="mt-3 text-xs text-slate-400" id="earnWeekNet">Netto stimato: £0.00</div>
        </div>
      </div>
    </section>

    <!-- CONTROLS -->
    <section class="rounded-2xl border border-slate-800 bg-slate-900/60 p-5 grid gap-4">
      <div class="grid sm:grid-cols-3 gap-4">
        <div>
          <label class="block text-xs text-slate-400 mb-1">Paga oraria</label>
          <div class="flex items-center rounded-xl border border-slate-800 bg-slate-950">
            <span class="px-3 text-slate-400" id="currencySymbol">£</span>
            <input id="hourlyInput" type="number" min="0" step="0.01" class="w-full bg-transparent p-3 outline-none" placeholder="12.35" />
          </div>
        </div>
        <div>
          <label class="block text-xs text-slate-400 mb-1">Tasse/Detrazioni (%)</label>
          <div class="flex items-center rounded-xl border border-slate-800 bg-slate-950">
            <input id="taxInput" type="number" min="0" max="70" step="0.1" class="w-full bg-transparent p-3 outline-none" placeholder="20" />
            <span class="px-3 text-slate-400">%</span>
          </div>
        </div>
        <div>
          <label class="block text-xs text-slate-400 mb-1">Valuta</label>
          <select id="currencySelect" class="w-full rounded-xl border border-slate-800 bg-slate-950 p-3">
            <option value="GBP">GBP – £</option>
            <option value="EUR">EUR – €</option>
            <option value="USD">USD – $</option>
          </select>
        </div>
      </div>

      <div class="flex items-center gap-3">
        <button id="startBtn" class="px-4 py-3 rounded-xl bg-emerald-600 hover:bg-emerald-500 font-semibold">Start turno</button>
        <button id="pauseBtn" class="px-4 py-3 rounded-xl bg-amber-600 hover:bg-amber-500 font-semibold hidden">Pausa</button>
        <button id="resumeBtn" class="px-4 py-3 rounded-xl bg-emerald-600 hover:bg-emerald-500 font-semibold hidden">Riprendi</button>
        <button id="stopBtn" class="px-4 py-3 rounded-xl bg-rose-600 hover:bg-rose-500 font-semibold hidden">Stop</button>
        <button id="resetTodayBtn" class="ml-auto px-3 py-3 rounded-xl border border-slate-800 hover:bg-slate-800/60 text-sm">Azzera Oggi</button>
      </div>

      <div class="text-xs text-slate-400">
        <span id="sessionInfo">Nessun turno attivo.</span>
      </div>
    </section>

    <!-- HISTORY (compact) -->
    <section class="rounded-2xl border border-slate-800 bg-slate-900/60 p-5">
      <div class="flex items-center justify-between mb-3">
        <h2 class="text-sm font-semibold text-slate-300">Turni recenti</h2>
        <button id="clearHistoryBtn" class="text-xs text-slate-400 hover:text-white">Pulisci</button>
      </div>
      <div id="historyList" class="grid gap-2 text-sm text-slate-300"></div>
    </section>

    <footer class="py-8 text-center text-xs text-slate-500">
      MVP v0.1 • Dati salvati in locale • Prossimo: login + sync
    </footer>
  </main>

  <!-- SETTINGS MODAL -->
  <dialog id="settingsModal" class="backdrop:bg-black/60 rounded-2xl p-0 w-[92vw] max-w-lg border border-slate-800 bg-slate-900 text-slate-100">
    <form method="dialog" class="grid">
      <div class="px-5 py-4 flex items-center justify-between border-b border-slate-800">
        <h3 class="font-semibold">Impostazioni</h3>
        <button class="text-slate-400 hover:text-white" id="closeSettings">✕</button>
      </div>
      <div class="p-5 grid gap-4">
        <div class="grid sm:grid-cols-2 gap-4">
          <label class="grid gap-1 text-sm">Inizio settimana
            <select id="weekStartSelect" class="rounded-xl border border-slate-800 bg-slate-950 p-3">
              <option value="1">Lunedì</option>
              <option value="0">Domenica</option>
            </select>
          </label>
          <label class="grid gap-1 text-sm">Auto-pausa inattività (min)
            <input id="autoPauseInput" type="number" min="0" step="1" class="rounded-xl border border-slate-800 bg-slate-950 p-3" placeholder="0 = off" />
          </label>
        </div>
        <p class="text-xs text-slate-400">Tutto è salvato in locale (localStorage). Nessun dato esce dal tuo dispositivo.</p>
      </div>
      <div class="px-5 py-4 border-t border-slate-800 flex items-center justify-end gap-2">
        <button id="saveSettingsBtn" value="save" class="px-4 py-2 rounded-xl bg-indigo-600 hover:bg-indigo-500 font-semibold">Salva</button>
        <button value="cancel" class="px-4 py-2 rounded-xl border border-slate-800 hover:bg-slate-800/60">Chiudi</button>
      </div>
    </form>
  </dialog>

  <script>
    // ------------------ STATE & STORAGE ------------------
    const SKEY = {
      hourly: 'wt_hourly',
      tax: 'wt_tax',
      currency: 'wt_currency',
      weekStart: 'wt_weekStart',
      sessions: 'wt_sessions', // JSON array of {start,end,durationMs,earned}
      autoPauseMin: 'wt_autoPauseMin'
    };

    const state = {
      running: false,
      paused: false,
      startTs: null,
      pauseAccumMs: 0,
      lastTick: null,
      hourly: parseFloat(localStorage.getItem(SKEY.hourly) ?? '12.35'),
      tax: parseFloat(localStorage.getItem(SKEY.tax) ?? '20'),
      currency: localStorage.getItem(SKEY.currency) || 'GBP',
      weekStart: parseInt(localStorage.getItem(SKEY.weekStart) ?? '1', 10),
      autoPauseMin: parseInt(localStorage.getItem(SKEY.autoPauseMin) ?? '0', 10),
      sessions: JSON.parse(localStorage.getItem(SKEY.sessions) || '[]')
    };

    // ------------------ UTILS ------------------
    const currencyMap = { GBP: '£', EUR: '€', USD: '$' };
    const fmtMoney = (n) => (n || 0).toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 });
    const todayKey = () => new Date().toISOString().slice(0,10);

    function saveState() {
      localStorage.setItem(SKEY.hourly, String(state.hourly));
      localStorage.setItem(SKEY.tax, String(state.tax));
      localStorage.setItem(SKEY.currency, state.currency);
      localStorage.setItem(SKEY.weekStart, String(state.weekStart));
      localStorage.setItem(SKEY.autoPauseMin, String(state.autoPauseMin));
      localStorage.setItem(SKEY.sessions, JSON.stringify(state.sessions));
    }

    function getWeekRange(d = new Date()) {
      const day = d.getDay(); // 0=Sun..6=Sat
      const diff = (day - state.weekStart + 7) % 7; // days since week start
      const start = new Date(d);
      start.setHours(0,0,0,0);
      start.setDate(start.getDate() - diff);
      const end = new Date(start);
      end.setDate(end.getDate() + 7);
      return { start, end };
    }

    function earningsPerMs() {
      const perHour = state.hourly || 0;
      return perHour / 3600000; // £ per millisecond
    }

    function net(val) {
      const tax = Math.max(0, Math.min(70, state.tax || 0));
      return val * (1 - tax / 100);
    }

    // ------------------ DOM ------------------
    const el = (id) => document.getElementById(id);
    const earnTodayEl = el('earnToday');
    const earnTodayNetEl = el('earnTodayNet');
    const earnWeekEl = el('earnWeek');
    const earnWeekNetEl = el('earnWeekNet');
    const sessionInfoEl = el('sessionInfo');
    const historyListEl = el('historyList');

    const hourlyInput = el('hourlyInput');
    const taxInput = el('taxInput');
    const currencySelect = el('currencySelect');
    const currencySymbol = el('currencySymbol');

    const startBtn = el('startBtn');
    const pauseBtn = el('pauseBtn');
    const resumeBtn = el('resumeBtn');
    const stopBtn = el('stopBtn');
    const resetTodayBtn = el('resetTodayBtn');

    const clearHistoryBtn = el('clearHistoryBtn');

    const settingsBtn = el('settingsBtn');
    const settingsModal = el('settingsModal');
    const closeSettings = el('closeSettings');
    const saveSettingsBtn = el('saveSettingsBtn');
    const weekStartSelect = el('weekStartSelect');
    const autoPauseInput = el('autoPauseInput');

    const statusText = el('statusText');

    // ------------------ INIT ------------------
    function loadUIFromState() {
      hourlyInput.value = String(state.hourly ?? '');
      taxInput.value = String(state.tax ?? '');
      currencySelect.value = state.currency;
      currencySymbol.textContent = currencyMap[state.currency] || '£';
      weekStartSelect.value = String(state.weekStart);
      autoPauseInput.value = String(state.autoPauseMin);
      updateTotalsUI();
      renderHistory();
    }

    function updateTotalsUI() {
      const curr = currencyMap[state.currency] || '£';
      // Today
      const todayTotal = sumSessionsRange(dayStart(new Date()), dayEnd(new Date()));
      earnTodayEl.textContent = curr + fmtMoney(todayTotal);
      earnTodayNetEl.textContent = `Netto stimato: ${curr}${fmtMoney(net(todayTotal))}`;

      // Week
      const { start, end } = getWeekRange();
      const weekTotal = sumSessionsRange(start, end);
      earnWeekEl.textContent = curr + fmtMoney(weekTotal);
      earnWeekNetEl.textContent = `Netto stimato: ${curr}${fmtMoney(net(weekTotal))}`;
    }

    function dayStart(d) { const x = new Date(d); x.setHours(0,0,0,0); return x; }
    function dayEnd(d) { const x = new Date(d); x.setHours(23,59,59,999); return x; }

    function sumSessionsRange(start, end) {
      let total = 0;
      for (const s of state.sessions) {
        const st = new Date(s.start).getTime();
        const en = new Date(s.end).getTime();
        if (en >= start.getTime() && st <= end.getTime()) total += s.earned || 0;
      }
      // include running session partial for today/week if active
      if (state.running && state.startTs) {
        const now = Date.now();
        const earned = (now - state.startTs - state.pauseAccumMs) * earningsPerMs();
        const st = state.startTs; const en = now;
        const inRange = en >= start.getTime() && st <= end.getTime();
        if (inRange) total += Math.max(0, earned);
      }
      return total;
    }

    function renderHistory() {
      if (!state.sessions.length) { historyListEl.innerHTML = '<div class="text-slate-500">Nessun turno salvato.</div>'; return; }
      const items = state.sessions.slice().reverse().slice(0, 10).map(s => {
        const d1 = new Date(s.start); const d2 = new Date(s.end);
        const durH = (s.durationMs || 0) / 3600000;
        const curr = currencyMap[state.currency] || '£';
        return `<div class="flex items-center justify-between rounded-xl border border-slate-800 bg-slate-950 p-3">
          <div class="text-slate-300">${d1.toLocaleDateString()} <span class="text-slate-500">${d1.toLocaleTimeString()}–${d2.toLocaleTimeString()}</span></div>
          <div class="text-slate-400 text-xs">${durH.toFixed(2)} h · <span class="text-slate-200 font-semibold">${curr}${fmtMoney(s.earned || 0)}</span></div>
        </div>`;
      }).join('');
      historyListEl.innerHTML = items;
    }

    // ------------------ SESSION LOGIC ------------------
    let tickHandle = null;

    function startSession() {
      if (state.running) return;
      const h = parseFloat(hourlyInput.value);
      if (!h || h <= 0) { alert('Imposta una paga oraria valida.'); return; }
      state.hourly = h; state.tax = parseFloat(taxInput.value || '0'); state.currency = currencySelect.value;
      state.startTs = Date.now();
      state.pauseAccumMs = 0;
      state.running = true; state.paused = false; state.lastTick = Date.now();
      saveState();
      updateButtons();
      sessionInfoEl.textContent = 'Turno in corso…';
      statusText.textContent = 'In corso';
      tickHandle = requestAnimationFrame(tick);
    }

    function pauseSession() {
      if (!state.running || state.paused) return;
      state.paused = true;
      // accumulate pause from now
      state.lastTick = Date.now();
      updateButtons();
      statusText.textContent = 'In pausa';
    }

    function resumeSession() {
      if (!state.running || !state.paused) return;
      // add paused duration
      const now = Date.now();
      state.pauseAccumMs += (now - state.lastTick);
      state.paused = false;
      state.lastTick = now;
      updateButtons();
      statusText.textContent = 'In corso';
      tickHandle = requestAnimationFrame(tick);
    }

    function stopSession() {
      if (!state.running) return;
      const now = Date.now();
      const effectiveMs = Math.max(0, (now - state.startTs - (state.paused ? (now - state.lastTick) : 0) - state.pauseAccumMs));
      const earned = effectiveMs * earningsPerMs();
      const session = { start: new Date(state.startTs).toISOString(), end: new Date(now).toISOString(), durationMs: effectiveMs, earned: earned };
      state.sessions.push(session);
      state.running = false; state.paused = false; state.startTs = null; state.pauseAccumMs = 0; state.lastTick = null;
      saveState();
      updateButtons();
      updateTotalsUI();
      renderHistory();
      sessionInfoEl.textContent = 'Nessun turno attivo.';
      statusText.textContent = 'Pronto';
      if (tickHandle) cancelAnimationFrame(tickHandle);
    }

    function tick() {
      if (!state.running) return;
      const now = Date.now();
      if (!state.paused) {
        updateTotalsUI();
        // auto-pause if needed
        if (state.autoPauseMin > 0) {
          // (MVP semplice) – se finestra perde focus per X minuti, metti in pausa
          // Nota: soluzione minimale; per qualcosa di serio: Web Worker + user activity listeners
        }
      }
      state.lastTick = now;
      tickHandle = requestAnimationFrame(tick);
    }

    function updateButtons() {
      startBtn.classList.toggle('hidden', state.running);
      stopBtn.classList.toggle('hidden', !state.running);
      pauseBtn.classList.toggle('hidden', !state.running || state.paused);
      resumeBtn.classList.toggle('hidden', !state.running || !state.paused);
    }

    function resetToday() {
      const yes = confirm('Azzero i turni di oggi?');
      if (!yes) return;
      const start = dayStart(new Date()).getTime();
      const end = dayEnd(new Date()).getTime();
      state.sessions = state.sessions.filter(s => {
        const st = new Date(s.start).getTime();
        const en = new Date(s.end).getTime();
        return !(en >= start && st <= end);
      });
      saveState();
      updateTotalsUI();
      renderHistory();
    }

    function clearHistory() {
      const yes = confirm('Pulisci tutti i turni salvati?');
      if (!yes) return;
      state.sessions = [];
      saveState();
      updateTotalsUI();
      renderHistory();
    }

    // ------------------ EVENTS ------------------
    startBtn.addEventListener('click', startSession);
    pauseBtn.addEventListener('click', pauseSession);
    resumeBtn.addEventListener('click', resumeSession);
    stopBtn.addEventListener('click', stopSession);
    resetTodayBtn.addEventListener('click', resetToday);
    clearHistoryBtn.addEventListener('click', clearHistory);

    hourlyInput.addEventListener('change', () => { state.hourly = parseFloat(hourlyInput.value || '0'); saveState(); updateTotalsUI(); });
    taxInput.addEventListener('change', () => { state.tax = parseFloat(taxInput.value || '0'); saveState(); updateTotalsUI(); });
    currencySelect.addEventListener('change', () => { state.currency = currencySelect.value; currencySymbol.textContent = currencyMap[state.currency] || '£'; saveState(); updateTotalsUI(); });

    settingsBtn.addEventListener('click', () => settingsModal.showModal());
    closeSettings.addEventListener('click', (e) => { e.preventDefault(); settingsModal.close(); });
    saveSettingsBtn.addEventListener('click', (e) => {
      e.preventDefault();
      state.weekStart = parseInt(weekStartSelect.value, 10);
      state.autoPauseMin = parseInt(autoPauseInput.value || '0', 10);
      saveState();
      updateTotalsUI();
      settingsModal.close();
    });

    // keyboard shortcuts MVP
    window.addEventListener('keydown', (e) => {
      if (e.key.toLowerCase() === 's') startSession();
      if (e.key.toLowerCase() === 'p') (state.paused ? resumeSession() : pauseSession());
      if (e.key.toLowerCase() === 'x') stopSession();
    });

    // First paint
    loadUIFromState();
    updateButtons();
  </script>

  <!--
  =========================
  ROADMAP (TODO NEXT)
  =========================
  - [ ] Export CSV dei turni
  - [ ] Autopause vera (idle/mouse/visibilitychange)
  - [ ] Tariffe multiple (overtime, notturni, weekend)
  - [ ] Obiettivi: target mensili + progress bar
  - [ ] Login + sync (Firebase / Supabase)
  - [ ] Grafici (Recharts) e report settimanale
  - [ ] UI polishing + tema chiaro/scuro
  - [ ] PWA installabile (offline support)
  - [ ] Pay periods (settimanale/bi-sett./mensile) + busta paga
  -->
</body>
</html>
