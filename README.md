<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover">
  <title>翻訳アプリ+（音声/カメラ/履歴/辞書/チャット）</title>
  <link rel="manifest" href="data:application/json,{&quot;name&quot;:&quot;翻訳アプリ+&quot;,&quot;short_name&quot;:&quot;翻訳+&quot;,&quot;display&quot;:&quot;standalone&quot;,&quot;start_url&quot;:&quot;.&quot;,&quot;background_color&quot;:&quot;#0f1220&quot;,&quot;theme_color&quot;:&quot;#0f1220&quot;}">
  <style>
    :root { --bg:#0f1220; --card:#171a2a; --text:#e8ecf1; --muted:#93a1b3; --accent:#6ae3ff; --line:#22263a; }
    * { box-sizing:border-box; }
    body { margin:0; font-family: system-ui,-apple-system,Segoe UI,Roboto,"Noto Sans JP",sans-serif; background:var(--bg); color:var(--text); }
    header { position:sticky; top:0; background:var(--bg); border-bottom:1px solid var(--line); z-index:10; }
    .bar { max-width:900px; margin:0 auto; padding:10px 16px; display:flex; gap:8px; align-items:center; }
    h1 { font-size:1.1rem; margin:0; }
    .container { max-width:900px; margin:0 auto; padding:16px; }
    .card { background:var(--card); border:1px solid var(--line); border-radius:14px; padding:14px; margin-bottom:16px; }
    .row { display:grid; grid-template-columns:1fr; gap:12px; }
    @media (min-width:880px){ .row.two { grid-template-columns:1fr 1fr; } }
    textarea, select, input[type=text] {
      width:100%; background:#0d1020; border:1px solid var(--line); color:var(--text); border-radius:10px; padding:10px; outline:none;
    }
    textarea { min-height:120px; resize:vertical; }
    .actions { display:flex; flex-wrap:wrap; gap:8px; }
    button { background:var(--accent); color:#001018; border:none; border-radius:10px; padding:10px 14px; font-weight:600; }
    button.secondary { background:#273048; color:var(--text); }
    .tiny { font-size:0.85rem; color:var(--muted); }
    .status { margin-top:8px; font-size:0.9rem; color:var(--muted); }
    .list { display:flex; flex-direction:column; gap:8px; max-height:220px; overflow:auto; }
    .item { padding:8px; border:1px solid var(--line); border-radius:10px; background:#0d1020; }
    .inline { display:flex; gap:8px; align-items:center; }
    .pill { background:#273048; color:var(--text); padding:4px 8px; border-radius:999px; font-size:0.8rem; }
    video, canvas { width:100%; max-height:260px; border-radius:12px; background:#000; }
  </style>
</head>
<body>
  <header>
    <div class="bar">
      <h1>翻訳アプリ+</h1>
      <div class="tiny">音声・カメラ・履歴・辞書・チャット</div>
    </div>
  </header>

  <div class="container">
    <!-- Translation core -->
    <div class="card">
      <div class="row two">
        <div>
          <div class="inline">
            <label class="tiny">元の言語</label>
            <select id="from">
              <option value="auto">自動検出</option>
              <option value="ja">日本語</option><option value="en">英語</option><option value="zh">中国語</option>
              <option value="ko">韓国語</option><option value="es">スペイン語</option><option value="fr">フランス語</option>
              <option value="de">ドイツ語</option><option value="ru">ロシア語</option><option value="pt">ポルトガル語</option>
            </select>
            <button id="mic" title="音声入力">🎙</button>
          </div>
          <textarea id="source" placeholder="話す / 入力 / カメラOCR…"></textarea>
        </div>
        <div>
          <div class="inline">
            <label class="tiny">翻訳後の言語</label>
            <select id="to">
              <option value="ja">日本語</option><option value="en" selected>英語</option><option value="zh">中国語</option>
              <option value="ko">韓国語</option><option value="es">スペイン語</option><option value="fr">フランス語</option>
              <option value="de">ドイツ語</option><option value="ru">ロシア語</option><option value="pt">ポルトガル語</option>
            </select>
            <button id="speak" title="読み上げ">🔊</button>
          </div>
          <textarea id="result" placeholder="翻訳結果…" readonly></textarea>
        </div>
      </div>
      <div class="actions">
        <button id="translate">翻訳する</button>
        <button id="swap" class="secondary">言語入れ替え</button>
        <button id="copy" class="secondary">結果コピー</button>
        <button id="fav" class="secondary">お気に入りに追加 ★</button>
        <button id="clear" class="secondary">クリア</button>
      </div>
      <div id="status" class="status">準備完了</div>
    </div>

    <!-- Camera OCR -->
    <div class="card">
      <div class="inline">
        <div class="pill">カメラ翻訳（OCR）</div>
        <button id="camStart">カメラ開始</button>
        <button id="camSnap" class="secondary">撮影→文字抽出</button>
      </div>
      <video id="video" playsinline></video>
      <canvas id="canvas" hidden></canvas>
      <div class="tiny">初回はカメラ許可を求められます。明るい場所で撮影すると精度が上がります。</div>
    </div>

    <!-- History & favorites -->
    <div class="card">
      <div class="inline">
        <div class="pill">履歴 / お気に入り</div>
        <button id="histClear" class="secondary">履歴クリア</button>
      </div>
      <div id="history" class="list"></div>
    </div>

    <!-- Dictionary / details -->
    <div class="card">
      <div class="inline">
        <div class="pill">辞書・用例（タップで詳細）</div>
      </div>
      <div id="dict" class="list"></div>
    </div>

    <!-- Chat mode -->
    <div class="card">
      <div class="inline"><div class="pill">チャットモード（双方向の翻訳）</div></div>
      <div class="row two">
        <div>
          <label class="tiny">あなたの言語</label>
          <select id="chatMe"><option value="ja">日本語</option><option value="en" selected>英語</option></select>
          <textarea id="chatInput" placeholder="あなたのメッセージ…"></textarea>
        </div>
        <div>
          <label class="tiny">相手の言語</label>
          <select id="chatYou"><option value="en">英語</option><option value="ja" selected>日本語</option></select>
          <div id="chatLog" class="list"></div>
        </div>
      </div>
      <div class="actions">
        <button id="chatSend">送信して翻訳</button>
      </div>
    </div>

    <div class="tiny">注: 公開翻訳APIはレート制限があります。安定運用は自己ホストや有料APIの利用を推奨。</div>
  </div>

  <script>
    // Elements
    const el = id => document.getElementById(id);
    const from = el('from'), to = el('to'), src = el('source'), out = el('result'), status = el('status');
    const historyEl = el('history'), dictEl = el('dict');
    const video = el('video'), canvas = el('canvas');

    // Local storage
    const store = {
      load(key, def=[]) { try { return JSON.parse(localStorage.getItem(key)) ?? def; } catch { return def; } },
      save(key, val) { localStorage.setItem(key, JSON.stringify(val)); }
    };
    let history = store.load('history');
    let favorites = store.load('favorites');

    function setStatus(msg){ status.textContent = msg; }

    // --- Translation engine (replace with your API for production) ---
    const API_URL = 'https://libretranslate.com/translate'; // 変更可
    async function translateText(text, sourceLang, targetLang){
      // 自動検出対応
      const payload = { q: text, source: sourceLang === 'auto' ? 'auto' : sourceLang, target: targetLang, format:'text' };
      const res = await fetch(API_URL, { method:'POST', headers:{'Content-Type':'application/json'}, body:JSON.stringify(payload) });
      if(!res.ok){ throw new Error('翻訳APIエラー: '+res.status); }
      const data = await res.json();
      return data.translatedText || data.response || '';
    }

    async function onTranslate(){
      const text = src.value.trim();
      if(!text){ setStatus('テキストを入力してください'); return; }
      setStatus('翻訳中…');
      try {
        const translated = await translateText(text, from.value, to.value);
        out.value = translated;
        setStatus('完了');

        // 履歴登録
        const item = { ts: Date.now(), from: from.value, to: to.value, src: text, out: translated };
        history.unshift(item); history = history.slice(0, 200); store.save('history', history);
        renderHistory();

        // 簡易辞書（単語分割）
        renderDict(translated);
      } catch(e){
        setStatus('翻訳に失敗: '+e.message);
      }
    }

    function renderHistory(){
      historyEl.innerHTML = '';
      history.forEach((h,i)=>{
        const div = document.createElement('div'); div.className='item';
        div.innerHTML = `<div class="tiny">${new Date(h.ts).toLocaleString()}</div>
          <div><span class="pill">${h.from}→${h.to}</span></div>
          <div class="tiny">原文</div><div>${escapeHtml(h.src)}</div>
          <div class="tiny">訳文</div><div>${escapeHtml(h.out)}</div>
          <div class="actions"><button data-i="${i}" class="restore secondary">復元</button>
          <button data-i="${i}" class="fav secondary">★お気に入り</button></div>`;
        historyEl.appendChild(div);
      });
      historyEl.querySelectorAll('.restore').forEach(b=>b.onclick = ()=>{
        const h = history[+b.dataset.i]; src.value = h.src; out.value = h.out; from.value = h.from; to.value = h.to;
      });
      historyEl.querySelectorAll('.fav').forEach(b=>b.onclick = ()=>{
        const h = history[+b.dataset.i]; favorites.unshift(h); favorites = favorites.slice(0,200); store.save('favorites', favorites);
        setStatus('お気に入りに追加しました');
      });
    }
    function escapeHtml(s){ return s.replace(/[&<>"']/g, m=>({ '&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;' }[m])); }

    function renderDict(text){
      dictEl.innerHTML = '';
      const words = [...new Set(text.split(/\s+/).filter(w=>w.length))].slice(0,30);
      words.forEach(w=>{
        const div = document.createElement('div'); div.className='item';
        div.textContent = w;
        div.onclick = async ()=>{
          div.textContent = w + ' …読み込み中';
          // ここで辞書API（例: Free Dictionary API）に差し替え可能
          const meaning = await fakeDictionary(w);
          div.innerHTML = `<strong>${escapeHtml(w)}</strong><div class="tiny">${escapeHtml(meaning)}</div>`;
        };
        dictEl.appendChild(div);
      });
    }
    async function fakeDictionary(word){
      // ダミー：本番は辞書APIから定義や用例を取得
      return '定義/用例は辞書APIを設定してください（学習用ダミー表示）。';
    }

    // --- Voice input (Web Speech) & Speech synthesis ---
    const micBtn = el('mic'), speakBtn = el('speak');
    let recognition;
    micBtn.onclick = ()=>{
      try {
        const SR = window.SpeechRecognition || window.webkitSpeechRecognition;
        if(!SR){ setStatus('音声入力はこのブラウザで未対応です'); return; }
        recognition = new SR();
        recognition.lang = from.value === 'auto' ? 'ja-JP' : langToBCP47(from.value);
        recognition.interimResults = false; recognition.onresult = e=>{
          const t = Array.from(e.results).map(r=>r[0].transcript).join(' ');
          src.value = (src.value ? src.value+'\n' : '') + t;
          setStatus('音声入力完了');
        };
        recognition.onerror = e=> setStatus('音声入力エラー: '+e.error);
        recognition.start(); setStatus('音声入力中…話しかけてください');
      } catch(e){ setStatus('音声開始失敗: '+e.message); }
    };
    speakBtn.onclick = ()=>{
      const text = out.value.trim(); if(!text){ setStatus('読み上げるテキストがありません'); return; }
      const u = new SpeechSynthesisUtterance(text);
      u.lang = langToBCP47(to.value); speechSynthesis.cancel(); speechSynthesis.speak(u);
      setStatus('読み上げ中…');
    };
    function langToBCP47(code){
      const map = { ja:'ja-JP', en:'en-US', zh:'zh-CN', ko:'ko-KR', es:'es-ES', fr:'fr-FR', de:'de-DE', ru:'ru-RU', pt:'pt-PT' };
      return map[code] || 'en-US';
    }

    // --- Camera OCR (getUserMedia + Tesseract.js) ---
    let stream;
    el('camStart').onclick = async ()=>{
      try {
        stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode:'environment' }, audio:false });
        video.srcObject = stream; await video.play();
        setStatus('カメラ起動');
      } catch(e){ setStatus('カメラ起動失敗: '+e.message); }
    };
    el('camSnap').onclick = async ()=>{
      if(!video.srcObject){ setStatus('先にカメラ開始を押してください'); return; }
      canvas.hidden = false;
      const w = video.videoWidth, h = video.videoHeight;
      canvas.width = w; canvas.height = h;
      const ctx = canvas.getContext('2d'); ctx.drawImage(video, 0, 0, w, h);
      setStatus('OCR中…');
      try {
        // ここでTesseract.jsを利用（CDNを読み込む必要あり）
        // 例）<script src="https://cdn.jsdelivr.net/npm/tesseract.js@v4/dist/tesseract.min.js"></script>
        if(!window.Tesseract){ throw new Error('Tesseract.jsが読み込まれていません'); }
        const { data:{ text } } = await Tesseract.recognize(canvas, 'eng');
        src.value = text; setStatus('OCR完了 → テキストを翻訳してみましょう');
      } catch(e){ setStatus('OCR失敗: '+e.message); }
    };

    // --- Favorites & actions ---
    el('fav').onclick = ()=>{
      const item = { ts: Date.now(), from: from.value, to: to.value, src: src.value.trim(), out: out.value.trim() };
      favorites.unshift(item); favorites = favorites.slice(0,200); store.save('favorites', favorites);
      renderHistory(); setStatus('お気に入りに追加しました');
    };
    el('histClear').onclick = ()=>{ history = []; store.save('history', history); renderHistory(); setStatus('履歴をクリアしました'); };
    el('translate').onclick = onTranslate;
    el('swap').onclick = ()=>{
      const f = from.value; from.value = to.value; to.value = f==='auto' ? to.value : f;
      const s = src.value; src.value = out.value; out.value = s; setStatus('言語とテキストを入れ替えました');
    };
    el('copy').onclick = async ()=>{
      try { await navigator.clipboard.writeText(out.value); setStatus('結果をコピーしました'); } catch{ setStatus('コピーできませんでした'); }
    };
    el('clear').onclick = ()=>{ src.value=''; out.value=''; setStatus('クリアしました'); };

    // --- Chat mode ---
    const chatMe = el('chatMe'), chatYou = el('chatYou'), chatInput = el('chatInput'), chatLog = el('chatLog');
    el('chatSend').onclick = async ()=>{
      const msg = chatInput.value.trim(); if(!msg){ setStatus('メッセージを入力してください'); return; }
      const mine = document.createElement('div'); mine.className='item'; mine.innerHTML = `<div class="pill">あなた</div><div>${escapeHtml(msg)}</div>`;
      chatLog.prepend(mine);
      setStatus('チャット翻訳中…');
      try {
        const translated = await translateText(msg, chatMe.value, chatYou.value);
        const theirs = document.createElement('div'); theirs.className='item';
        theirs.innerHTML = `<div class="pill">相手(${chatYou.value})</div><div>${escapeHtml(translated)}</div>`;
        chatLog.prepend(theirs);
        setStatus('送信完了');
      } catch(e){ setStatus('チャット翻訳失敗: '+e.message); }
      chatInput.value = '';
    };

    // --- Render initial history ---
    renderHistory();

    // --- PWA: minimal service worker inline registration (optional offline cache UI) ---
    if('serviceWorker' in navigator){
      const swCode = `
        const CACHE = 'translate-plus-v1';
        const ASSETS = ['.', location.href];
        self.addEventListener('install', e=>{
          e.waitUntil(caches.open(CACHE).then(c=>c.addAll(ASSETS)));
        });
        self.addEventListener('fetch', e=>{
          e.respondWith(caches.match(e.request).then(r=> r || fetch(e.request)));
        });
      `;
      const blob = new Blob([swCode], { type: 'text/javascript' });
      const swUrl = URL.createObjectURL(blob);
      navigator.serviceWorker.register(swUrl).catch(()=>{});
    }
  </script>

  <!-- OCR用ライブラリ（本番は外部CDNを使用） -->
  <!-- 例: <script src="https://cdn.jsdelivr.net/npm/tesseract.js@v4/dist/tesseract.min.js"></script> -->
</body>
</html>
