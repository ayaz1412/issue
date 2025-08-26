<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>ImagifyPro Suite - Dashboard</title>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/1.5.13/cropper.min.css">
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body {
      font-family: "Segoe UI", Tahoma, sans-serif;
      background: #f4f7fb;
      color: #333;
      display: flex;
      flex-direction: column;
      height: 100vh;
    }
    header {
      height: 60px;
      background: linear-gradient(90deg,#1a73e8,#155fc1);
      color: #fff;
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 0 20px;
      font-size: 18px;
      font-weight: 600;
      box-shadow: 0 2px 6px rgba(0,0,0,0.2);
      position: sticky;
      top: 0;
      z-index: 1000;
    }
    header .brand { display:flex; align-items:center; gap:10px; }
    header .actions i {
      margin: 0 8px;
      cursor: pointer;
      transition: transform 0.2s, color 0.3s;
    }
    header .actions i:hover { color: #ffdf5d; transform: scale(1.2); }

    .container { display: flex; flex: 1; overflow: hidden; }
    .sidebar {
      width: 260px;
      background: #fff;
      border-right: 1px solid #ddd;
      display: flex;
      flex-direction: column;
      padding-top: 20px;
      transition: width 0.3s;
    }
    .sidebar button {
      background: none;
      border: none;
      text-align: left;
      padding: 14px 20px;
      font-size: 15px;
      cursor: pointer;
      color: #333;
      display: flex;
      align-items: center;
      gap:10px;
      transition: background 0.3s, color 0.3s;
    }
    .sidebar button:hover { background: #1a73e8; color: #fff; }

    .main { flex: 1; display: flex; flex-direction: column; padding: 20px; overflow-y: auto; }
    .toolbar { display: flex; align-items: center; margin-bottom: 15px; flex-wrap: wrap; gap: 10px; }
    .toolbar input[type="text"] { flex: 1; padding: 10px 14px; border: 1px solid #ccc; border-radius: 25px; outline: none; min-width: 200px; }
    .progress-bar { height: 8px; background: #ddd; border-radius: 4px; flex: 0.3; overflow: hidden; min-width: 120px; }
    .progress-bar span { display: block; height: 100%; width: 0%; background: linear-gradient(90deg,#1a73e8,#42a5f5); transition: width 0.3s; }

    .grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(220px, 1fr)); gap: 20px; }
    .card { background: #fff; border-radius: 15px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); padding: 12px; display: flex; flex-direction: column; align-items: center; transition: transform 0.2s, box-shadow 0.3s; }
    .card:hover { transform: translateY(-5px); box-shadow: 0 8px 20px rgba(0,0,0,0.15); }
    .card img { max-width: 100%; border-radius: 10px; cursor: pointer; }
    .card .actions { margin-top: 10px; display: flex; gap: 8px; flex-wrap: wrap; justify-content:center; }
    .card button { background: #1a73e8; color: #fff; border: none; border-radius: 8px; padding: 6px 10px; cursor: pointer; font-size: 13px; transition: background 0.3s; }
    .card button:hover { background: #155fc1; }
    .card .meta { width:100%; display:flex; align-items:center; justify-content:space-between; margin-top:8px; font-size:12px; color:#666; }

    .select-row { display:flex; width:100%; align-items:center; justify-content:space-between; gap:10px; margin-bottom:8px; }

    /* Modal cropper */
    .modal { position: fixed; inset:0; background: rgba(0,0,0,0.7); display: flex; justify-content: center; align-items: center; visibility: hidden; opacity: 0; transition: 0.3s; z-index: 2000; }
    .modal.active { visibility: visible; opacity: 1; }
    .modal-content { background: #fff; padding: 20px; border-radius: 12px; max-width: 95%; max-height: 95%; overflow: auto; }
    .modal-footer { margin-top:10px; text-align:right; display:flex; gap:10px; justify-content:flex-end; }

    /* Drag & drop */
    .dropzone { border: 2px dashed #1a73e8; border-radius: 12px; padding: 24px; text-align:center; color:#4b5563; background:#eef4ff; }
    .dropzone.dragover { background:#dbeafe; }

    .dark-mode { background: #121212; color: #eee; }
    .dark-mode .sidebar { background: #1f1f1f; border-color: #444; }
    .dark-mode .card { background: #1f1f1f; }
    .dark-mode header { background: linear-gradient(90deg,#333,#000); }

    /* Responsive */
    @media(max-width: 768px){ .sidebar { width: 64px; } .sidebar button span { display: none; } }
    @media(max-width: 600px){ .toolbar { flex-direction: column; align-items: stretch; } .progress-bar { flex: 1; } }
  </style>
</head>
<body>
  <header>
    <div class="brand"><span style="font-size:22px">📄</span> ImagifyPro Suite</div>
    <div class="actions">
      <i class="fas fa-upload" id="uploadBtn" title="Upload"></i>
      <i class="fas fa-file-export" id="exportPdfBtn" title="Export PDF"></i>
      <i class="fas fa-file-lines" id="exportTxtBtn" title="Export OCR Text"></i>
      <i class="fas fa-moon" id="modeToggle" title="Toggle Dark Mode"></i>
    </div>
  </header>

  <div class="container">
    <div class="sidebar">
      <button id="openUpload"><i class="fas fa-folder-open"></i><span> Upload Images</span></button>
      <button id="autoCropAll"><i class="fas fa-crop-alt"></i><span> Auto-Crop All (use last crop)</span></button>
      <button id="runOCR"><i class="fas fa-search"></i><span> OCR & Index Text</span></button>
      <button id="sideExportPdf"><i class="fas fa-file-pdf"></i><span> Export PDF</span></button>
      <button id="sideExportTxt"><i class="fas fa-file-lines"></i><span> Export Text</span></button>
    </div>

    <div class="main">
      <div class="toolbar">
        <input type="text" id="searchInput" placeholder="Search text in images...">
        <div class="progress-bar" title="Processing progress"><span id="progress"></span></div>
      </div>

      <div id="dropzone" class="dropzone">
        <p><strong>Drag & drop</strong> images here, or click <em>Upload Images</em>.</p>
        <p style="font-size:12px; opacity:0.8;">PNG, JPG, JPEG — multiple files supported</p>
      </div>

      <div class="grid" id="imageGrid" style="margin-top:20px"></div>
    </div>
  </div>

  <!-- Crop Modal -->
  <div class="modal" id="cropModal">
    <div class="modal-content">
      <img id="cropImage" src="" style="max-width:100%; display:block;">
      <div class="modal-footer">
        <button id="cropApplyAll" title="Apply this crop to all selected cards">Apply to Selected</button>
        <button id="cropSave">Save</button>
        <button id="cropCancel">Cancel</button>
      </div>
    </div>
  </div>

  <!-- Hidden inputs -->
  <input type="file" id="fileInput" multiple accept="image/*" style="display:none;">

  <script src="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/1.5.13/cropper.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/tesseract.js/5.0.3/tesseract.min.js"></script>
  <script>
    // ==== State ====
    const state = {
      images: [], // {id, dataUrl, fileName, ocrText:"", cropData:null}
      lastCropData: null
    };

    // ==== Elements ====
    const modeToggle = document.getElementById('modeToggle');
    const fileInput = document.getElementById('fileInput');
    const uploadBtn = document.getElementById('uploadBtn');
    const openUpload = document.getElementById('openUpload');
    const imageGrid = document.getElementById('imageGrid');
    const progressBar = document.getElementById('progress');
    const searchInput = document.getElementById('searchInput');
    const exportPdfBtn = document.getElementById('exportPdfBtn');
    const exportTxtBtn = document.getElementById('exportTxtBtn');
    const sideExportPdf = document.getElementById('sideExportPdf');
    const sideExportTxt = document.getElementById('sideExportTxt');
    const runOCRBtn = document.getElementById('runOCR');
    const autoCropAllBtn = document.getElementById('autoCropAll');

    const cropModal = document.getElementById('cropModal');
    const cropImg = document.getElementById('cropImage');
    const cropSave = document.getElementById('cropSave');
    const cropCancel = document.getElementById('cropCancel');
    const cropApplyAll = document.getElementById('cropApplyAll');
    let cropper = null; let activeCropId = null;

    const dropzone = document.getElementById('dropzone');

    // ==== Helpers ====
    const uid = () => Math.random().toString(36).slice(2,9);
    function setProgress(p){ progressBar.style.width = `${p}%`; }

    function dataUrlToImage(dataUrl){ return new Promise(res=>{ const img = new Image(); img.onload=()=>res(img); img.src=dataUrl; }); }

    async function cropDataUrl(dataUrl, cropData){
      const img = await dataUrlToImage(dataUrl);
      const canvas = document.createElement('canvas');
      canvas.width = cropData.width; canvas.height = cropData.height;
      const ctx = canvas.getContext('2d');
      ctx.drawImage(img, cropData.x, cropData.y, cropData.width, cropData.height, 0,0, cropData.width, cropData.height);
      return canvas.toDataURL('image/png');
    }

    function createCard(imgObj){
      const card = document.createElement('div');
      card.className = 'card';
      card.dataset.id = imgObj.id;
      card.innerHTML = `
        <div class="select-row">
          <label style="display:flex; align-items:center; gap:8px;">
            <input type="checkbox" class="selectChk"> Select
          </label>
          <span class="file-name" title="File name">${imgObj.fileName || 'image'}</span>
        </div>
        <img src="${imgObj.dataUrl}" alt="image" class="preview">
        <div class="actions">
          <button class="btnCrop"><i class="fas fa-crop-alt"></i> Crop</button>
          <button class="btnOCR"><i class="fas fa-search"></i> OCR</button>
          <button class="btnDownload"><i class="fas fa-download"></i> Save</button>
          <button class="btnRemove" title="Remove image"><i class="fas fa-trash"></i></button>
        </div>
        <div class="meta"><span class="ocr-status">OCR: pending</span><span class="dim"></span></div>
      `;
      const imgEl = card.querySelector('img');
      imgEl.addEventListener('click', ()=> openCrop(imgObj.id));
      card.querySelector('.btnCrop').addEventListener('click', ()=> openCrop(imgObj.id));
      card.querySelector('.btnRemove').addEventListener('click', ()=> removeImage(imgObj.id));
      card.querySelector('.btnDownload').addEventListener('click', ()=> downloadDataUrl(imgObj.dataUrl, (imgObj.fileName||'image')+'.png'));
      card.querySelector('.btnOCR').addEventListener('click', ()=> runOCRForIds([imgObj.id]));

      // Set dimensions meta
      const temp = new Image(); temp.onload = ()=>{ card.querySelector('.dim').textContent = `${temp.naturalWidth}x${temp.naturalHeight}`; }; temp.src = imgObj.dataUrl;

      imageGrid.appendChild(card);
    }

    function getSelectedIds(){ return [...document.querySelectorAll('.selectChk:checked')].map(chk => chk.closest('.card').dataset.id); }

    function removeImage(id){ state.images = state.images.filter(x=>x.id!==id); document.querySelector(`.card[data-id="${id}"]`)?.remove(); }

    function refreshCardImage(id, newDataUrl){
      const card = document.querySelector(`.card[data-id="${id}"]`);
      if(!card) return;
      card.querySelector('img').src = newDataUrl;
      const obj = state.images.find(x=>x.id===id); if(obj){ obj.dataUrl = newDataUrl; }
    }

    function downloadDataUrl(dataUrl, filename){
      const a = document.createElement('a'); a.href = dataUrl; a.download = filename; a.click();
    }

    function openCrop(id){
      const obj = state.images.find(x=>x.id===id); if(!obj) return;
      activeCropId = id; cropImg.src = obj.dataUrl; cropModal.classList.add('active');
      cropImg.onload = ()=>{ if(cropper) cropper.destroy(); cropper = new Cropper(cropImg, { viewMode:1 }); };
    }

    function closeCrop(){ cropModal.classList.remove('active'); if(cropper){ cropper.destroy(); cropper=null; } activeCropId=null; }

    // ==== Events ====
    modeToggle.addEventListener('click', ()=> document.body.classList.toggle('dark-mode'));
    uploadBtn.addEventListener('click', ()=> fileInput.click());
    openUpload.addEventListener('click', ()=> fileInput.click());

    fileInput.addEventListener('change', (e)=> handleFiles(e.target.files));

    async function handleFiles(fileList){
      const files = [...fileList];
      if(!files.length) return;
      setProgress(0);
      let idx = 0;
      for(const file of files){
        const reader = new FileReader();
        const p = Math.round(((idx)/files.length)*100); setProgress(p);
        await new Promise(res=>{ reader.onload = (ev)=>{ const id = uid(); const imgObj = { id, dataUrl: ev.target.result, fileName: file.name, ocrText:"", cropData:null }; state.images.push(imgObj); createCard(imgObj); res(); }; reader.readAsDataURL(file); });
        idx++;
      }
      setProgress(100); setTimeout(()=> setProgress(0), 600);
    }

    // Drag & drop
    ;['dragenter','dragover','dragleave','drop'].forEach(evt => dropzone.addEventListener(evt, e=>{ e.preventDefault(); e.stopPropagation(); }));
    ;['dragenter','dragover'].forEach(evt => dropzone.addEventListener(evt, ()=> dropzone.classList.add('dragover')));
    ;['dragleave','drop'].forEach(evt => dropzone.addEventListener(evt, ()=> dropzone.classList.remove('dragover')));
    dropzone.addEventListener('drop', (e)=>{ const dt = e.dataTransfer; const files = dt.files; handleFiles(files); });

    // Crop modal buttons
    cropCancel.addEventListener('click', closeCrop);

    cropSave.addEventListener('click', async ()=>{
      if(!cropper || !activeCropId) return;
      const data = cropper.getData(true); // {x,y,width,height}
      state.lastCropData = { x:Math.round(data.x), y:Math.round(data.y), width:Math.round(data.width), height:Math.round(data.height) };
      const obj = state.images.find(x=>x.id===activeCropId);
      const newUrl = await cropDataUrl(obj.dataUrl, state.lastCropData);
      obj.dataUrl = newUrl; obj.cropData = state.lastCropData;
      refreshCardImage(activeCropId, newUrl);
      closeCrop();
    });

    cropApplyAll.addEventListener('click', async ()=>{
      if(!cropper) return;
      const data = cropper.getData(true);
      state.lastCropData = { x:Math.round(data.x), y:Math.round(data.y), width:Math.round(data.width), height:Math.round(data.height) };
      const ids = getSelectedIds();
      if(ids.length===0){ alert('Select some images first.'); return; }
      let i=0; for(const id of ids){ const obj=state.images.find(x=>x.id===id); if(!obj) continue; const newUrl = await cropDataUrl(obj.dataUrl, state.lastCropData); obj.dataUrl=newUrl; obj.cropData=state.lastCropData; refreshCardImage(id, newUrl); setProgress(Math.round(((i+1)/ids.length)*100)); i++; }
      setTimeout(()=> setProgress(0), 600);
      alert('Applied crop to selected images.');
    });

    // Sidebar Auto-Crop All uses last crop rect
    autoCropAllBtn.addEventListener('click', async ()=>{
      if(!state.lastCropData){ alert('No crop set yet. Open any image, crop it, and Save once to set the crop.'); return; }
      const ids = getSelectedIds().length ? getSelectedIds() : state.images.map(x=>x.id);
      let i=0; for(const id of ids){ const obj=state.images.find(x=>x.id===id); if(!obj) continue; const newUrl = await cropDataUrl(obj.dataUrl, state.lastCropData); obj.dataUrl=newUrl; obj.cropData=state.lastCropData; refreshCardImage(id, newUrl); setProgress(Math.round(((i+1)/ids.length)*100)); i++; }
      setTimeout(()=> setProgress(0), 600);
    });

    // OCR
    async function runOCRForIds(ids){
      if(ids.length===0){ alert('No images selected.'); return; }
      const workerOpt = { logger: m => { if(m.progress) setProgress(Math.round(m.progress*100)); } };
      for(const id of ids){
        const card = document.querySelector(`.card[data-id="${id}"]`);
        const status = card?.querySelector('.ocr-status');
        status && (status.textContent = 'OCR: processing...');
        const obj = state.images.find(x=>x.id===id);
        try {
          const { data } = await Tesseract.recognize(obj.dataUrl, 'eng', workerOpt);
          obj.ocrText = (data.text || '').trim();
          status && (status.textContent = obj.ocrText ? 'OCR: done' : 'OCR: no text');
        } catch(err){ console.error(err); status && (status.textContent = 'OCR: error'); }
      }
      setTimeout(()=> setProgress(0), 600);
    }

    runOCRBtn.addEventListener('click', ()=>{
      const ids = getSelectedIds().length ? getSelectedIds() : state.images.map(x=>x.id);
      runOCRForIds(ids);
    });

    // Search filtering by OCR text
    searchInput.addEventListener('input', ()=>{
      const q = searchInput.value.toLowerCase();
      document.querySelectorAll('.card').forEach(card=>{
        const id = card.dataset.id; const obj = state.images.find(x=>x.id===id);
        const hay = (obj?.ocrText || '').toLowerCase();
        card.style.display = (!q || hay.includes(q)) ? '' : 'none';
      });
    });

    // Export PDF
    async function exportPDF(ids){
      if(ids.length===0){ alert('No images selected.'); return; }
      setProgress(0);
      const { jsPDF } = window.jspdf;
      const doc = new jsPDF({ unit:'pt', format:'a4' });
      const pageW = doc.internal.pageSize.getWidth();
      const pageH = doc.internal.pageSize.getHeight();
      const margin = 36; // 0.5in

      let first = true; let i=0;
      for(const id of ids){
        const obj = state.images.find(x=>x.id===id); if(!obj) continue;
        const img = await dataUrlToImage(obj.dataUrl);
        const scale = Math.min((pageW-2*margin)/img.width, (pageH-2*margin)/img.height);
        const w = img.width * scale; const h = img.height * scale;
        const x = (pageW - w)/2; const y = (pageH - h)/2;
        if(!first) doc.addPage(); first=false;
        doc.addImage(obj.dataUrl, 'PNG', x, y, w, h);
        setProgress(Math.round(((i+1)/ids.length)*100)); i++;
      }
      doc.save('imagifypro.pdf');
      setTimeout(()=> setProgress(0), 600);
    }

    exportPdfBtn.addEventListener('click', ()=>{ const ids = getSelectedIds().length ? getSelectedIds() : state.images.map(x=>x.id); exportPDF(ids); });
    sideExportPdf.addEventListener('click', ()=>{ const ids = getSelectedIds().length ? getSelectedIds() : state.images.map(x=>x.id); exportPDF(ids); });

    // Export OCR text
    function exportText(ids){
      if(ids.length===0){ alert('No images selected.'); return; }
      let out = '';
      ids.forEach((id, idx)=>{ const obj = state.images.find(x=>x.id===id); out += `#${idx+1} ${obj.fileName || 'image'}\n${(obj.ocrText||'').trim()}\n\n`; });
      const blob = new Blob([out], {type:'text/plain'}); const url = URL.createObjectURL(blob);
      const a = document.createElement('a'); a.href=url; a.download='imagifypro_ocr.txt'; a.click(); URL.revokeObjectURL(url);
    }
    exportTxtBtn.addEventListener('click', ()=>{ const ids = getSelectedIds().length ? getSelectedIds() : state.images.map(x=>x.id); exportText(ids); });
    sideExportTxt.addEventListener('click', ()=>{ const ids = getSelectedIds().length ? getSelectedIds() : state.images.map(x=>x.id); exportText(ids); });
  </script>
</body>
</html>

