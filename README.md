<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>JP-COPILOT</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    :root {
      --bg: #0b1020; --panel: #131a2e; --panel-2: #1a2340;
      --border: #263255; --text: #e6ecff; --muted: #8b98bd;
      --accent: #4f7cff; --accent-2: #7c5cff;
    }
    html, body { height: 100%; }
    body {
      font-family: 'Segoe UI', system-ui, sans-serif;
      background: radial-gradient(1200px 600px at 20% -10%, #1b2a5c 0%, var(--bg) 55%);
      color: var(--text); display: flex; justify-content: center;
      padding: 16px; height: 100vh; overflow: hidden;
    }
    .app { width: 100%; max-width: 900px; display: flex; flex-direction: column;
      background: linear-gradient(180deg, var(--panel), #0f1528);
      border: 1px solid var(--border); border-radius: 18px; overflow: hidden;
      box-shadow: 0 20px 60px rgba(0,0,0,.5); }
    header { display: flex; align-items: center; gap: 12px; padding: 14px 18px;
      background: linear-gradient(90deg, rgba(79,124,255,.15), rgba(124,92,255,.08));
      border-bottom: 1px solid var(--border); }
    .logo { width: 40px; height: 40px; border-radius: 12px;
      background: linear-gradient(135deg, var(--accent), var(--accent-2));
      display: grid; place-items: center; font-weight: 800; font-size: 18px;
      box-shadow: 0 6px 16px rgba(79,124,255,.4); }
    .title h1 { font-size: 17px; }
    .title p { font-size: 12px; color: var(--muted); }
    .status { margin-left: auto; font-size: 12px; color: var(--muted);
      display: flex; align-items: center; gap: 6px; }
    .dot { width: 8px; height: 8px; border-radius: 50%; background: #f5a524;
      box-shadow: 0 0 8px #f5a524; }
    .dot.ready { background: #22c55e; box-shadow: 0 0 8px #22c55e; }
    .dot.error { background: #ef4444; box-shadow: 0 0 8px #ef4444; }
    #chat { flex: 1; overflow-y: auto; padding: 22px; display: flex;
      flex-direction: column; gap: 14px; scroll-behavior: smooth; }
    #chat::-webkit-scrollbar { width: 8px; }
    #chat::-webkit-scrollbar-thumb { background: #2a3760; border-radius: 8px; }
    .row { display: flex; gap: 10px; align-items: flex-end; }
    .row.user { flex-direction: row-reverse; }
    .avatar { width: 32px; height: 32px; border-radius: 10px; flex-shrink: 0;
      display: grid; place-items: center; font-size: 13px; font-weight: 700;
      background: var(--panel-2); border: 1px solid var(--border); }
    .row.user .avatar { background: linear-gradient(135deg, var(--accent), var(--accent-2)); border: none; }
    .bubble { max-width: 78%; padding: 12px 16px; border-radius: 16px;
      line-height: 1.55; white-space: pre-wrap; word-wrap: break-word;
      font-size: 15px; animation: pop .18s ease-out; }
    @keyframes pop { from { transform: translateY(4px); opacity: 0; } to { transform: none; opacity: 1; } }
    .row.user .bubble { background: linear-gradient(135deg, var(--accent), #3a63e0);
      border-bottom-right-radius: 4px; }
    .row.ai .bubble { background: var(--ai, var(--panel-2)); border: 1px solid var(--border);
      border-bottom-left-radius: 4px; }
    .typing { display: inline-flex; gap: 4px; }
    .typing span { width: 6px; height: 6px; border-radius: 50%; background: var(--muted);
      animation: blink 1.2s infinite; }
    .typing span:nth-child(2) { animation-delay: .2s; }
    .typing span:nth-child(3) { animation-delay: .4s; }
    @keyframes blink { 0%, 80%, 100% { opacity: .2; } 40% { opacity: 1; } }
    form { display: flex; gap: 10px; padding: 14px 16px;
      background: rgba(19,26,46,.9); border-top: 1px solid var(--border); }
    #input { flex: 1; padding: 13px 16px; border-radius: 12px;
      border: 1px solid var(--border); background: #0e1426; color: var(--text);
      font-size: 15px; outline: none; }
    #input:focus { border-color: var(--accent); }
    button { padding: 0 22px; border: none; border-radius: 12px;
      background: linear-gradient(135deg, var(--accent), var(--accent-2));
      color: white; font-weight: 700; font-size: 14px; cursor: pointer; }
    button:disabled { opacity: .45; cursor: not-allowed; }
    .reset { background: transparent; border: 1px solid var(--border);
      color: var(--muted); padding: 0 14px; }
    .search-tag { display: inline-block; font-size: 11px; color: #9fb3ff;
      background: rgba(79,124,255,.15); border: 1px solid rgba(79,124,255,.4);
      padding: 2px 8px; border-radius: 999px; margin-bottom: 8px; }
    .sources { margin-top: 10px; padding-top: 10px; border-top: 1px dashed var(--border);
      font-size: 12px; color: var(--muted); }
    .sources a { color: #9fb3ff; text-decoration: none; display: block;
      margin-top: 4px; word-break: break-all; }
    .sources a:hover { text-decoration: underline; }
  </style>
</head>
<body>
  <div class="app">
    <header>
      <div class="logo">JP</div>
      <div class="title"><h1>JP-COPILOT</h1><p>Offline AI + live web search</p></div>
      <div class="status"><span id="dot" class="dot"></span><span id="statusText">Booting...</span></div>
    </header>

    <div id="chat"></div>

    <form id="form">
      <input id="input" placeholder="Message JP-COPILOT... (try: 'search: latest AI news')" autocomplete="off" required />
      <button id="reset" type="button" class="reset">Clear</button>
      <button id="send" type="submit">Send</button>
    </form>
  </div>

  <script type="module">
    import { pipeline, env } from 'https://cdn.jsdelivr.net/npm/@xenova/transformers@2.17.2';
    env.allowLocalModels = false;

    const chat = document.getElementById('chat');
    const form = document.getElementById('form');
    const input = document.getElementById('input');
    const send = document.getElementById('send');
    const reset = document.getElementById('reset');
    const dot = document.getElementById('dot');
    const statusText = document.getElementById('statusText');

    const MAX_TURNS = 6;
    let history = [];
    let generator = null;
    let loading = false;

    const setStatus = (s, t) => { dot.className = 'dot ' + s; statusText.textContent = t; };

    function addMessage(role, text) {
      const row = document.createElement('div');
      row.className = `row ${role}`;
      row.innerHTML = `<div class="avatar">${role === 'user' ? 'You' : 'JP'}</div>
                       <div class="bubble"></div>`;
      row.querySelector('.bubble').textContent = text;
      chat.appendChild(row);
      chat.scrollTop = chat.scrollHeight;
      return row.querySelector('.bubble');
    }

    function addTyping() {
      const row = document.createElement('div');
      row.className = 'row ai';
      row.innerHTML = `<div class="avatar">JP</div>
        <div class="bubble"><span class="typing"><span></span><span></span><span></span></span></div>`;
      chat.appendChild(row);
      chat.scrollTop = chat.scrollHeight;
      return row;
    }

    async function loadModel() {
      if (generator) return generator;
      loading = true;
      setStatus('', 'Loading model...');
      generator = await pipeline('text2text-generation', 'Xenova/LaMini-Flan-T5-248M');
      setStatus('ready', 'Ready');
      loading = false;
      return generator;
    }

    // --- LIVE SEARCH via DuckDuckGo (free, no key) ---
    async function liveSearch(query) {
      const url = `https://api.duckduckgo.com/?q=${encodeURIComponent(query)}&format=json&no_html=1&skip_disambig=1&t=jp-copilot`;
      const res = await fetch(url);
      if (!res.ok) throw new Error('Search failed: ' + res.status);
      const data = await res.json();

      const results = [];
      if (data.AbstractText) {
        results.push({ title: data.Heading || query, text: data.AbstractText, url: data.AbstractURL });
      }
      if (Array.isArray(data.RelatedTopics)) {
        for (const topic of data.RelatedTopics.slice(0, 4)) {
          if (topic.Text && topic.FirstURL) {
            results.push({ title: topic.Text.slice(0, 80), text: topic.Text, url: topic.FirstURL });
          }
        }
      }
      return results.slice(0, 5);
    }

    function buildPrompt(question, searchResults = []) {
      const lines = [];
      const recent = history.slice(-MAX_TURNS * 2);
      for (const m of recent) {
        if (m.role === 'user') lines.push(`User: ${m.content}`);
        else lines.push(`Assistant: ${m.content}`);
      }
      if (searchResults.length) {
        lines.push('Context from live web search:');
        for (const r of searchResults) lines.push(`- ${r.text}`);
      }
      lines.push(`User: ${question}`);
      lines.push('Assistant:');
      return lines.join('\n');
    }

    function renderSources(bubble, results) {
      if (!results.length) return;
      const src = document.createElement('div');
      src.className = 'sources';
      src.innerHTML = '<strong>Sources:</strong>';
      for (const r of results) {
        const a = document.createElement('a');
        a.href = r.url; a.target = '_blank'; a.rel = 'noopener';
        a.textContent = '• ' + (r.title || r.url);
        src.appendChild(a);
      }
      bubble.appendChild(src);
    }

    form.addEventListener('submit', async (e) => {
      e.preventDefault();
      if (loading) return;
      const raw = input.value.trim();
      if (!raw) return;

      // detect "search:" prefix
      const isSearch = /^search:\s*/i.test(raw);
      const question = isSearch ? raw.replace(/^search:\s*/i, '') : raw;

      addMessage('user', raw);
      history.push({ role: 'user', content: raw });
      input.value = '';
      send.disabled = true; reset.disabled = true;

      const typingRow = addTyping();

      try {
        const gen = await loadModel();
        let results = [];

        if (isSearch) {
          setStatus('', 'Searching web...');
          try {
            results = await liveSearch(question);
            if (!results.length) {
              typingRow.remove();
              const b = addMessage('ai', '');
              const tag = document.createElement('div');
              tag.className = 'search-tag';
              tag.textContent = '🌐 Live search returned no results';
              b.appendChild(tag);
              b.appendChild(document.createTextNode("I couldn't find anything useful. Try rephrasing."));
              setStatus('ready', 'Ready');
              return;
            }
          } catch (err) {
            typingRow.remove();
            addMessage('ai', 'Live search failed: ' + err.message);
            setStatus('error', 'Search error');
            return;
          }
          setStatus('', 'Thinking...');
        }

        const prompt = buildPrompt(question, results);
        const out = await gen(prompt, {
          max_new_tokens: 200, temperature: 0.7, top_p: 0.9, do_sample: true,
        });

        let reply = out[0].generated_text.trim();
        const last = reply.lastIndexOf('Assistant:');
        if (last !== -1) reply = reply.slice(last + 10).trim();
        if (!reply) reply = "I'm not sure how to answer that. Could you rephrase?";

        typingRow.remove();
        const bubble = addMessage('ai', '');
        if (isSearch) {
          const tag = document.createElement('div');
          tag.className = 'search-tag';
          tag.textContent = '🌐 Live web search';
          bubble.appendChild(tag);
        }
        bubble.appendChild(document.createTextNode(reply));
        renderSources(bubble, results);

        history.push({ role: 'assistant', content: reply });
        if (history.length > MAX_TURNS * 2) history = history.slice(-MAX_TURNS * 2);
        setStatus('ready', 'Ready');
      } catch (err) {
        typingRow.remove();
        setStatus('error', 'Error');
        addMessage('ai', 'Something went wrong: ' + err.message);
      } finally {
        send.disabled = false; reset.disabled = false; input.focus();
      }
    });

    reset.addEventListener('click', () => {
      history = []; chat.innerHTML = ''; greet(); input.focus();
    });

    function greet() {
      addMessage('ai',
        "Hi! I'm JP-COPILOT 👋\n" +
        "• Type normally for offline AI answers\n" +
        "• Prefix with 'search:' to trigger a live web lookup\n" +
        "Example: search: who won the last F1 race");
    }

    greet();
    input.focus();
    loadModel().catch(() => setStatus('error', 'Load failed'));
  </script>
</body>
</html>
