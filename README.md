<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>PDF → JPG Converter</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
<style>
  :root {
    --bg-dark: #09090b;
    --surface: rgba(255, 255, 255, 0.03);
    --surface-hover: rgba(255, 255, 255, 0.06);
    --border: rgba(255, 255, 255, 0.08);
    --border-active: rgba(255, 255, 255, 0.2);
    --accent: #6366f1;
    --accent-glow: rgba(99, 102, 241, 0.4);
    --text-main: #fafafa;
    --text-muted: #a1a1aa;
    --radius: 16px;
    --transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  }

  * { margin: 0; padding: 0; box-sizing: border-box; }
  
  body {
    font-family: 'Inter', system-ui, sans-serif;
    background-color: var(--bg-dark);
    background-image: 
      radial-gradient(circle at 15% 50%, rgba(99, 102, 241, 0.08) 0%, transparent 25%),
      radial-gradient(circle at 85% 30%, rgba(236, 72, 153, 0.06) 0%, transparent 25%);
    min-height: 100vh;
    color: var(--text-main);
    padding: 40px 20px;
    line-height: 1.5;
  }

  .container { max-width: 1000px; margin: 0 auto; position: relative; z-index: 1; }

  header { text-align: center; margin-bottom: 48px; animation: fadeDown 0.6s ease-out; }
  header h1 { 
    font-size: 2.5rem; font-weight: 700; letter-spacing: -0.03em; margin-bottom: 12px;
    background: linear-gradient(135deg, #fff 0%, #a1a1aa 100%);
    -webkit-background-clip: text; background-clip: text; color: transparent;
  }
  header p { color: var(--text-muted); font-size: 1.05rem; max-width: 500px; margin: 0 auto; }

  .glass {
    background: var(--surface);
    backdrop-filter: blur(12px); -webkit-backdrop-filter: blur(12px);
    border: 1px solid var(--border);
    border-radius: var(--radius);
  }

  #dropZone {
    padding: 64px 32px; text-align: center; cursor: pointer;
    transition: var(--transition); position: relative; overflow: hidden;
    border-style: dashed; border-width: 2px;
  }
  #dropZone:hover, #dropZone.dragover {
    background: var(--surface-hover); border-color: var(--accent);
    box-shadow: 0 0 40px var(--accent-glow); transform: translateY(-2px);
  }
  #dropZone .icon-wrap {
    width: 72px; height: 72px; margin: 0 auto 20px;
    background: linear-gradient(135deg, var(--accent), #ec4899);
    border-radius: 20px; display: flex; align-items: center; justify-content: center;
    font-size: 2rem; box-shadow: 0 8px 24px var(--accent-glow);
  }
  #dropZone h3 { font-size: 1.25rem; font-weight: 600; margin-bottom: 8px; }
  #dropZone p { color: var(--text-muted); font-size: 0.9rem; }
  #dropZone strong { color: var(--accent); font-weight: 600; }

  .controls {
    display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 24px; padding: 24px; margin: 24px 0; align-items: end;
  }
  .control-group label { 
    display: block; font-size: 0.8rem; font-weight: 600; 
    text-transform: uppercase; letter-spacing: 0.05em; 
    color: var(--text-muted); margin-bottom: 10px; 
  }
  select {
    width: 100%; padding: 12px 16px; border-radius: 10px;
    background: rgba(0,0,0,0.3); border: 1px solid var(--border);
    color: var(--text-main); font-size: 0.95rem; outline: none;
    transition: var(--transition); appearance: none;
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='12' fill='%23a1a1aa' viewBox='0 0 16 16'%3E%3Cpath d='M8 11L3 6h10z'/%3E%3C/svg%3E");
    background-repeat: no-repeat; background-position: right 14px center;
  }
  select:focus { border-color: var(--accent); box-shadow: 0 0 0 3px var(--accent-glow); }
  
  input[type=range] {
    width: 100%; height: 6px; border-radius: 99px; outline: none;
    background: rgba(255,255,255,0.1); appearance: none; margin-top: 8px;
  }
  input[type=range]::-webkit-slider-thumb {
    appearance: none; width: 20px; height: 20px; border-radius: 50%;
    background: var(--accent); cursor: pointer; border: 3px solid var(--bg-dark);
    box-shadow: 0 0 10px var(--accent-glow); transition: var(--transition);
  }
  input[type=range]::-webkit-slider-thumb:hover { transform: scale(1.2); }
  .range-value { float: right; color: var(--accent); font-weight: 600; font-size: 0.85rem; }

  #progressWrap { display: none; margin: 24px 0; padding: 20px; }
  .bar-track { height: 6px; background: rgba(255,255,255,0.06); border-radius: 99px; overflow: hidden; }
  .bar-fill { 
    height: 100%; width: 0%; border-radius: 99px; 
    background: linear-gradient(90deg, var(--accent), #ec4899);
    transition: width 0.3s ease; position: relative;
  }
  .bar-fill::after {
    content: ''; position: absolute; inset: 0;
    background: linear-gradient(90deg, transparent, rgba(255,255,255,0.3), transparent);
    animation: shimmer 1.5s infinite;
  }
  @keyframes shimmer { 0%{transform:translateX(-100%)} 100%{transform:translateX(100%)} }
  #progressText { font-size: 0.85rem; color: var(--text-muted); margin-top: 10px; display: flex; justify-content: space-between; }

  .actions { display: flex; gap: 12px; margin: 24px 0; flex-wrap: wrap; }
  button {
    border: none; border-radius: 12px; padding: 14px 28px; 
    font-size: 0.95rem; font-weight: 600; cursor: pointer; 
    transition: var(--transition); font-family: inherit;
    display: inline-flex; align-items: center; gap: 8px;
  }
  .btn-primary { 
    background: var(--accent); color: #fff; 
    box-shadow: 0 4px 20px var(--accent-glow);
  }
  .btn-primary:hover { filter: brightness(1.15); transform: translateY(-1px); box-shadow: 0 8px 30px var(--accent-glow); }
  .btn-secondary { background: var(--surface); color: var(--text-muted); border: 1px solid var(--border); }
  .btn-secondary:hover { background: var(--surface-hover); color: var(--text-main); border-color: var(--border-active); }
  button:disabled { opacity: 0.4; cursor: not-allowed; transform: none !important; filter: none !important; }

  #gallery { display: grid; grid-template-columns: repeat(auto-fill, minmax(220px, 1fr)); gap: 20px; margin-top: 8px; }
  .card {
    border-radius: var(--radius); overflow: hidden; 
    background: var(--surface); border: 1px solid var(--border);
    transition: var(--transition); animation: cardIn 0.4s ease-out both;
  }
  .card:hover { border-color: var(--border-active); transform: translateY(-4px); box-shadow: 0 12px 40px rgba(0,0,0,0.3); }
  @keyframes cardIn { from { opacity:0; transform: translateY(20px) scale(0.97); } to { opacity:1; transform:none; } }
  
  .card-img-wrap { 
    height: 220px; background: #fff; display: flex; align-items: center; justify-content: center;
    position: relative; overflow: hidden;
  }
  .card-img-wrap img { max-width: 100%; max-height: 100%; object-fit: contain; }
  .card-body { padding: 14px 16px; display: flex; justify-content: space-between; align-items: center; }
  .card-info span { display: block; }
  .card-info .page-num { font-weight: 600; font-size: 0.9rem; color: var(--text-main); }
  .card-info .file-size { font-size: 0.75rem; color: var(--text-muted); margin-top: 2px; }
  
  .dl-btn {
    width: 36px; height: 36px; border-radius: 10px; display: flex; align-items: center; justify-content: center;
    background: var(--surface-hover); border: 1px solid var(--border); color: var(--text-muted);
    text-decoration: none; transition: var(--transition); font-size: 1rem;
  }
  .dl-btn:hover { background: var(--accent); color: #fff; border-color: var(--accent); }

  footer { text-align: center; color: #52525b; font-size: 0.8rem; margin-top: 60px; padding-bottom: 20px; }

  @keyframes fadeDown { from { opacity:0; transform: translateY(-20px); } to { opacity:1; transform: none; } }

  @media (max-width: 600px) {
    header h1 { font-size: 1.8rem; }
    #dropZone { padding: 40px 20px; }
    .controls { grid-template-columns: 1fr; gap: 16px; padding: 18px; }
    #gallery { grid-template-columns: repeat(auto-fill, minmax(160px, 1fr)); gap: 12px; }
    .card-img-wrap { height: 160px; }
  }
</style>
</head>
<body>
<div class="container">
  <header>
    <h1>PDF to Image</h1>
    <p>Convert PDF pages to high-quality JPGs instantly. Private, fast, and entirely in your browser.</p>
  </header>

  <div id="dropZone" class="glass">
    <div class="icon-wrap">📄</div>
    <h3>Drop your PDF here</h3>
    <p>or <strong>click to browse</strong> · Max recommended 50MB</p>
  </div>
  <input type="file" id="fileInput" accept="application/pdf" hidden />

  <div class="controls glass">
    <div class="control-group">
      <label>Output Quality <span class="range-value" id="qualityVal">90%</span></label>
      <input type="range" id="quality" min="0.3" max="1" step="0.05" value="0.9" />
    </div>
    <div class="control-group">
      <label>Resolution Scale</label>
      <select id="resolution">
        <option value="1">Standard (1×)</option>
        <option value="1.5">High (1.5×)</option>
        <option value="2" selected>Retina (2×)</option>
        <option value="3">Ultra HD (3×)</option>
      </select>
    </div>
  </div>

  <div id="progressWrap" class="glass">
    <div class="bar-track"><div class="bar-fill" id="barFill"></div></div>
    <div id="progressText"><span id="progLabel">Preparing…</span><span id="progPercent">0%</span></div>
  </div>

  <div class="actions" id="actionBar" style="display:none">
    <button class="btn-primary" id="downloadAllBtn">⬇ Download All as ZIP</button>
    <button class="btn-secondary" id="resetBtn">↺ Convert Another</button>
  </div>

  <div id="gallery"></div>
  <footer>No files are uploaded. Conversion happens locally via PDF.js & JSZip.</footer>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
<script>
// FIX: Set worker source properly for GitHub Pages
pdfjsLib.GlobalWorkerOptions.workerSrc =
  'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.worker.min.js';

const $ = id => document.getElementById(id);
const dropZone = $('dropZone'), fileInput = $('fileInput'), gallery = $('gallery');
const progressWrap = $('progressWrap'), barFill = $('barFill');
const progLabel = $('progLabel'), progPercent = $('progPercent');
const actionBar = $('actionBar');

// ✅ FIX: Changed $('qualityEl') to $('quality') to match the HTML ID
const qualityEl = $('quality');
const qualityVal = $('qualityVal'), resEl = $('resolution');

let pdfDoc = null, results = [], baseName = 'document';

// Quality slider live update
qualityEl.addEventListener('input', () =>
  qualityVal.textContent = Math.round(qualityEl.value * 100) + '%');
qualityEl.addEventListener('change', () => { if (pdfDoc) convert(); });
resEl.addEventListener('change', () => { if (pdfDoc) convert(); });

// File selection
dropZone.addEventListener('click', () => fileInput.click());
fileInput.addEventListener('change', e => { if (e.target.files[0]) loadPDF(e.target.files[0]); });

// Drag & Drop
['dragenter','dragover'].forEach(ev => dropZone.addEventListener(ev, e => {
  e.preventDefault(); dropZone.classList.add('dragover');
}));
['dragleave','drop'].forEach(ev => dropZone.addEventListener(ev, e => {
  e.preventDefault(); dropZone.classList.remove('dragover');
}));
dropZone.addEventListener('drop', e => {
  const file = e.dataTransfer.files[0];
  if (file) loadPDF(file);
});
window.addEventListener('dragover', e => e.preventDefault());
window.addEventListener('drop', e => e.preventDefault());

// Load PDF
async function loadPDF(file) {
  if (file.type !== 'application/pdf' && !file.name.toLowerCase().endsWith('.pdf')) {
    alert('Please select a valid PDF file.'); return;
  }
  baseName = file.name.replace(/\.pdf$/i, '');
  try {
    const buf = await file.arrayBuffer();
    pdfDoc = await pdfjsLib.getDocument({ data: buf }).promise;
    convert();
  } catch (err) {
    if (err?.name === 'PasswordException') {
      const pw = prompt('This PDF is password protected.\nEnter password:');
      if (pw) {
        const buf2 = await file.arrayBuffer();
        pdfDoc = await pdfjsLib.getDocument({ data: buf2, password: pw }).promise;
        convert();
      }
    } else {
      alert('Could not read this PDF.\n' + err.message);
    }
  }
}

// Convert all pages
async function convert() {
  gallery.innerHTML = ''; results = [];
  actionBar.style.display = 'none';
  progressWrap.style.display = 'block';
  const scale = parseFloat(resEl.value);
  const quality = parseFloat(qualityEl.value);
  const total = pdfDoc.numPages;

  for (let i = 1; i <= total; i++) {
    progLabel.textContent = `Converting page ${i} of ${total}`;
    const pct = Math.round((i / total) * 100);
    progPercent.textContent = pct + '%';
    barFill.style.width = pct + '%';

    const blob = await renderPageToJPG(i, scale, quality);
    const name = `${baseName}-page-${i}.jpg`;
    results.push({ name, blob });
    addCard(name, blob, i);
  }

  progLabel.textContent = `✅ Complete — ${total} page${total > 1 ? 's' : ''} converted`;
  progPercent.textContent = '100%';
  actionBar.style.display = 'flex';
}

// Render single page
async function renderPageToJPG(pageNum, scale, quality) {
  const page = await pdfDoc.getPage(pageNum);
  const viewport = page.getViewport({ scale });
  const canvas = document.createElement('canvas');
  canvas.width = viewport.width;
  canvas.height = viewport.height;
  const ctx = canvas.getContext('2d');
  ctx.fillStyle = '#ffffff';
  ctx.fillRect(0, 0, canvas.width, canvas.height);
  await page.render({ canvasContext: ctx, viewport }).promise;
  return new Promise(res => canvas.toBlob(res, 'image/jpeg', quality));
}

// Add card to gallery
function addCard(name, blob, pageNum) {
  const url = URL.createObjectURL(blob);
  const card = document.createElement('div');
  card.className = 'card';
  card.style.animationDelay = `${(pageNum - 1) * 0.05}s`;
  card.innerHTML = `
    <div class="card-img-wrap">
      <img src="${url}" alt="Page ${pageNum}" loading="lazy">
    </div>
    <div class="card-body">
      <div class="card-info">
        <span class="page-num">Page ${pageNum}</span>
        <span class="file-size">${formatSize(blob.size)}</span>
      </div>
      <a href="${url}" download="${name}" class="dl-btn" title="Download JPG">⬇</a>
    </div>`;
  gallery.appendChild(card);
}

function formatSize(bytes) {
  if (bytes < 1024) return bytes + ' B';
  if (bytes < 1048576) return (bytes / 1024).toFixed(1) + ' KB';
  return (bytes / 1048576).toFixed(1) + ' MB';
}

// Download All ZIP
$('downloadAllBtn').addEventListener('click', async () => {
  const btn = $('downloadAllBtn');
  btn.disabled = true; btn.innerHTML = '⏳ Zipping…';
  const zip = new JSZip();
  results.forEach(r => zip.file(r.name, r.blob));
  const zipBlob = await zip.generateAsync({ type: 'blob' });
  saveBlob(zipBlob, baseName + '-images.zip');
  btn.disabled = false; btn.innerHTML = '⬇ Download All as ZIP';
});

// Reset
$('resetBtn').addEventListener('click', () => {
  pdfDoc = null; results = [];
  gallery.innerHTML = '';
  fileInput.value = '';
  actionBar.style.display = 'none';
  progressWrap.style.display = 'none';
  barFill.style.width = '0%';
});

function saveBlob(blob, filename) {
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = filename;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  setTimeout(() => URL.revokeObjectURL(a.href), 5000);
}
</script>
</body>
</html>
