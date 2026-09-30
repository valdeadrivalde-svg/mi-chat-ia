<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Mi Chat Orquestador (ChatGPT + Gemini)</title>
  <style>
    * { box-sizing: border-box; font-family: 'Segoe UI', system-ui, sans-serif; }
    body { background-color: #0f172a; color: #f8fafc; margin: 0; padding: 20px; display: flex; justify-content: center; }
    .card { background-color: #1e293b; width: 100%; max-width: 800px; padding: 24px; border-radius: 16px; box-shadow: 0 10px 25px rgba(0,0,0,0.5); }
    h1 { margin-top: 0; font-size: 1.5rem; color: #38bdf8; text-align: center; }
    p.sub { text-align: center; color: #94a3b8; font-size: 0.9rem; margin-bottom: 20px; }
    .config-box { background: #0f172a; padding: 15px; border-radius: 10px; border: 1px solid #334155; margin-bottom: 20px; }
    .config-box h3 { margin-top: 0; font-size: 1rem; color: #e2e8f0; }
    label { font-size: 0.8rem; color: #94a3b8; display: block; margin-top: 8px; }
    input[type="text"] { width: 100%; padding: 8px 12px; margin-top: 4px; background: #1e293b; border: 1px solid #475569; border-radius: 6px; color: #fff; font-size: 0.85rem; }
    textarea { width: 100%; height: 100px; padding: 12px; background: #0f172a; border: 1px solid #334155; border-radius: 8px; color: #fff; font-size: 0.95rem; resize: vertical; }
    button { width: 100%; padding: 12px; margin-top: 10px; background: #2563eb; color: white; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; font-size: 1rem; }
    button:hover { background: #1d4ed8; }
    .loading { display: none; text-align: center; margin: 15px 0; color: #38bdf8; font-weight: bold; }
    .output-box { display: none; margin-top: 20px; padding: 16px; background: #0f172a; border-left: 4px solid #38bdf8; border-radius: 6px; white-space: pre-wrap; line-height: 1.6; font-size: 0.95rem; }
  </style>
</head>
<body>

  <div class="card">
    <h1>🤖 Orquestador IA</h1>
    <p class="sub">Pregunta una vez y obtén la respuesta combinada de ChatGPT y Gemini</p>

    <div class="config-box">
      <h3>🔑 Claves API (se guardan solo en tu navegador)</h3>
      <label>Google Gemini API Key:</label>
      <input type="text" id="geminiKey" placeholder="Pega tu clave de Gemini aquí">
      
      <label>OpenAI ChatGPT API Key:</label>
      <input type="text" id="openaiKey" placeholder="Pega tu clave de OpenAI (sk-...) aquí">
    </div>

    <textarea id="promptInput" placeholder="Escribe tu pregunta aquí..."></textarea>
    <button onclick="ejecutarOrquestador()">Enviar Pregunta</button>

    <div id="loading" class="loading">⏳ Consultando a ChatGPT y Gemini en paralelo...</div>
    <div id="output" class="output-box"></div>
  </div>

<script>
  document.getElementById('geminiKey').value = localStorage.getItem('gemini_key') || '';
  document.getElementById('openaiKey').value = localStorage.getItem('openai_key') || '';

  async function ejecutarOrquestador() {
    const geminiKey = document.getElementById('geminiKey').value.trim();
    const openaiKey = document.getElementById('openaiKey').value.trim();
    const prompt = document.getElementById('promptInput').value.trim();
    const loading = document.getElementById('loading');
    const output = document.getElementById('output');

    if (!geminiKey || !openaiKey) {
      alert("Por favor, introduce ambas claves API antes de continuar.");
      return;
    }
    if (!prompt) {
      alert("Por favor escribe una pregunta.");
      return;
    }

    localStorage.setItem('gemini_key', geminiKey);
    localStorage.setItem('openai_key', openaiKey);

    loading.style.display = 'block';
    output.style.display = 'none';

    try {
      const [resGemini, resGPT] = await Promise.all([
        consultarGemini(geminiKey, prompt),
        consultarOpenAI(openaiKey, prompt)
      ]);

      const promptSintesis = `
Pregunta original: ${prompt}

--- RESPUESTA A (ChatGPT) ---
${resGPT}

--- RESPUESTA B (Gemini) ---
${resGemini}

INSTRUCCIONES:
Lee ambas respuestas y genera una única respuesta combinada en español. 
Coge lo mejor de cada una, elimina duplicados y da una respuesta clara y estructurada.
      `;

      loading.innerText = "✨ Combinando y mejorando lo mejor de ambas respuestas...";
      const respuestaFinal = await consultarGemini(geminiKey, promptSintesis);

      output.innerText = respuestaFinal;
      output.style.display = 'block';
    } catch (err) {
      alert("Ocurrió un error: " + err.message);
    } finally {
      loading.style.display = 'none';
      loading.innerText = "⏳ Consultando a ChatGPT y Gemini en paralelo...";
    }
  }

  async function consultarGemini(apiKey, text) {
    const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-3.8-flash:generateContent?key=${apiKey}`;
    const res = await fetch(url, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ contents: [{ parts: [{ text: text }] }] })
    });
    const data = await res.json();
    if (data.error) throw new Error("Gemini Error: " + data.error.message);
    return data.candidates[0].content.parts[0].text;
  }

  async function consultarOpenAI(apiKey, text) {
    const url = 'https://api.openai.com/v1/chat/completions';
    const res = await fetch(url, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${apiKey}`
      },
      body: JSON.stringify({
        model: 'gpt-4o-mini',
        messages: [{ role: 'user', content: text }]
      })
    });
    const data = await res.json();
    if (data.error) throw new Error("OpenAI Error: " + data.error.message);
    return data.choices[0].message.content;
  }
</script>

</body>
</html>
