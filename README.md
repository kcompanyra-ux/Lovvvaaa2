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
