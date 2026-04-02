<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Gift Card Balance Lookup</title>
  <script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
  <style>
    * { box-sizing: border-box; }
    body {
      margin: 0;
      font-family: Arial, Helvetica, sans-serif;
      background: linear-gradient(135deg, #f7f7f7, #eceff1);
      color: #222;
      padding: 24px;
    }
    .container {
      max-width: 1200px;
      margin: 0 auto;
      display: grid;
      gap: 20px;
    }
    .card {
      background: #fff;
      border-radius: 22px;
      box-shadow: 0 10px 30px rgba(0,0,0,0.08);
      padding: 24px;
    }
    h1, h2, h3 { margin-top: 0; }
    .subtitle {
      color: #666;
      margin-top: -6px;
      margin-bottom: 18px;
    }
    .lookup-grid {
      display: grid;
      grid-template-columns: 2fr auto auto;
      gap: 12px;
      align-items: end;
    }
    .field {
      display: flex;
      flex-direction: column;
      gap: 8px;
    }
    label {
      font-size: 14px;
      font-weight: 700;
    }
    input {
      width: 100%;
      padding: 14px 16px;
      border: 1px solid #d0d5dd;
      border-radius: 12px;
      font-size: 16px;
    }
    input:focus {
      outline: none;
      border-color: #111;
      box-shadow: 0 0 0 4px rgba(17,17,17,0.08);
    }
    button {
      padding: 14px 18px;
      border: none;
      border-radius: 12px;
      font-size: 15px;
      font-weight: 700;
      cursor: pointer;
    }
    .primary { background: #111; color: #fff; }
    .secondary { background: #e9ecef; color: #111; }
    .danger { background: #c92a2a; color: #fff; }
    .muted {
      color: #666;
      font-size: 14px;
    }
    .result-grid {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 14px;
      margin-top: 14px;
    }
    .stat {
      padding: 16px;
      border: 1px solid #ececec;
      border-radius: 16px;
      background: #fafafa;
    }
    .stat-label {
      font-size: 12px;
      font-weight: 700;
      color: #666;
      text-transform: uppercase;
      letter-spacing: 0.4px;
      margin-bottom: 6px;
    }
    .stat-value {
      font-size: 22px;
      font-weight: 800;
      word-break: break-word;
    }
    .status {
      margin-top: 14px;
      min-height: 20px;
      font-size: 14px;
      font-weight: 700;
    }
    .toolbar {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
      margin-top: 16px;
      align-items: center;
    }
    .table-wrap {
      overflow-x: auto;
      margin-top: 10px;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      min-width: 900px;
    }
    th, td {
      padding: 12px;
      border-bottom: 1px solid #eee;
      text-align: left;
      font-size: 14px;
    }
    th {
      background: #fafafa;
    }
    .edit-grid {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 12px;
      margin-top: 16px;
    }
    .hidden { display: none; }
    .note {
      font-size: 13px;
      color: #666;
      line-height: 1.5;
      margin-top: 12px;
    }
    .file-input {
      padding: 10px;
      border: 1px dashed #c8ced6;
      border-radius: 12px;
      background: #fafafa;
      max-width: 340px;
    }
    @media (max-width: 900px) {
      .lookup-grid,
      .edit-grid,
      .result-grid {
        grid-template-columns: 1fr;
      }
      body {
        padding: 14px;
      }
    }
  </style>
</head>
<body>
  <div class="container">
    <div class="card">
      <h1>Gift Card Balance Lookup</h1>
      <div class="subtitle">Search by serial card, view balance, update the card record, or import your Excel file.</div>

      <div class="lookup-grid">
        <div class="field">
          <label for="lookupSerial">Serial Card</label>
          <input id="lookupSerial" type="text" placeholder="Type serial card number" />
        </div>
        <button id="searchBtn" class="primary">Check Balance</button>
        <button id="resetLookupBtn" class="secondary">Reset</button>
      </div>

      <div id="status" class="status"></div>

      <div id="resultCard" class="hidden">
        <div class="result-grid">
          <div class="stat">
            <div class="stat-label">GC From</div>
            <div id="resultGcFrom" class="stat-value">-</div>
          </div>
          <div class="stat">
            <div class="stat-label">Serial Card</div>
            <div id="resultSerial" class="stat-value">-</div>
          </div>
          <div class="stat">
            <div class="stat-label">Current Balance</div>
            <div id="resultCurrentBalance" class="stat-value">-</div>
          </div>
          <div class="stat">
            <div class="stat-label">Active Date</div>
            <div id="resultActiveDate" class="stat-value">-</div>
          </div>
          <div class="stat">
            <div class="stat-label">Last Activity Date</div>
            <div id="resultLastActivityDate" class="stat-value">-</div>
          </div>
          <div class="stat">
            <div class="stat-label">Original Amt</div>
            <div id="resultOriginalAmt" class="stat-value">-</div>
          </div>
        </div>

        <h3 style="margin-top:20px;">Update This Card</h3>
        <div class="edit-grid">
          <div class="field">
            <label for="editGcFrom">GC From</label>
            <input id="editGcFrom" type="text" />
          </div>
          <div class="field">
            <label for="editSerial">Serial Card</label>
            <input id="editSerial" type="text" />
          </div>
          <div class="field">
            <label for="editActiveDate">Active Date</label>
            <input id="editActiveDate" type="date" />
          </div>
          <div class="field">
            <label for="editLastActivityDate">Last Activity Date</label>
            <input id="editLastActivityDate" type="date" />
          </div>
          <div class="field">
            <label for="editOriginalAmt">Original Amt</label>
            <input id="editOriginalAmt" type="number" step="0.01" />
          </div>
          <div class="field">
            <label for="editCurrentBalance">Current Balance</label>
            <input id="editCurrentBalance" type="number" step="0.01" />
          </div>
        </div>

        <div class="edit-grid" style="margin-top:16px;">
          <div class="field">
            <label for="amountUsed">Amount Used</label>
            <input id="amountUsed" type="number" step="0.01" min="0" placeholder="Enter amount used" />
          </div>
          <div class="field">
            <label for="autoActivityDate">Activity Date</label>
            <input id="autoActivityDate" type="date" />
          </div>
        </div>

        <div class="toolbar">
          <button id="deductBtn" class="primary">Apply Amount Used</button>
          <button id="saveUpdateBtn" class="secondary">Save Manual Update</button>
        </div>
      </div>
    </div>

    <div class="card">
      <h2>Gift Card Database</h2>
      <div class="muted">Use this table for all cards. Search above by serial card number.</div>

      <div class="toolbar">
        <button id="addNewBtn" class="primary">Add New Gift Card</button>
        <button id="exportBtn" class="secondary">Export CSV</button>
        <button id="clearBtn" class="danger">Clear All</button>
      </div>

      <div class="toolbar">
        <div class="field" style="min-width:280px;">
          <label for="excelFile">Upload Excel or CSV</label>
          <input id="excelFile" class="file-input" type="file" accept=".xlsx,.xls,.csv" />
        </div>
        <button id="importBtn" class="primary" type="button">Import File</button>
      </div>

      <div class="note">
        Required columns: <strong>GC From</strong>, <strong>Serial Card</strong>, <strong>Active Date</strong>, <strong>Last Activity Date</strong>, <strong>Original Amt</strong>, <strong>Current Balance</strong>.
      </div>

      <div id="newCardPanel" class="hidden" style="margin-top:18px;">
        <h3>Add New Gift Card</h3>
        <div class="edit-grid">
          <div class="field">
            <label for="newGcFrom">GC From</label>
            <input id="newGcFrom" type="text" placeholder="Store / SPA / Online" />
          </div>
          <div class="field">
            <label for="newSerial">Serial Card</label>
            <input id="newSerial" type="text" placeholder="Enter serial" />
          </div>
          <div class="field">
            <label for="newActiveDate">Active Date</label>
            <input id="newActiveDate" type="date" />
          </div>
          <div class="field">
            <label for="newLastActivityDate">Last Activity Date</label>
            <input id="newLastActivityDate" type="date" />
          </div>
          <div class="field">
            <label for="newOriginalAmt">Original Amt</label>
            <input id="newOriginalAmt" type="number" step="0.01" />
          </div>
          <div class="field">
            <label for="newCurrentBalance">Current Balance</label>
            <input id="newCurrentBalance" type="number" step="0.01" />
          </div>
        </div>
        <div class="toolbar">
          <button id="saveNewBtn" class="primary">Save New Card</button>
          <button id="cancelNewBtn" class="secondary">Cancel</button>
        </div>
      </div>

      <div class="table-wrap">
        <table>
          <thead>
            <tr>
              <th>GC From</th>
              <th>Serial Card</th>
              <th>Active Date</th>
              <th>Last Activity Date</th>
              <th>Original Amt</th>
              <th>Current Balance</th>
            </tr>
          </thead>
          <tbody id="tableBody"></tbody>
        </table>
      </div>

      <div class="note">
        This version stores the database in the browser on the device you use. You can import from Excel and export back to CSV for Excel.
      </div>
    </div>
  </div>

  <script>
  const STORAGE_KEY = 'gift_card_balance_database';
  let selectedIndex = -1;

  function loadData() {
    try { return JSON.parse(localStorage.getItem(STORAGE_KEY)) || []; }
    catch { return []; }
  }

  function saveData(data) {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
  }

  function normalizeSerial(v){ return String(v||'').trim().toLowerCase(); }

  function formatMoney(v){ const n=Number(v||0); return `$${Number.isNaN(n)?0:n.toFixed(2)}`; }

  function setStatus(m){ document.getElementById('status').textContent = m || ''; }

  function getTodayLocalDate(){
    const now=new Date(); const off=now.getTimezoneOffset();
    const local=new Date(now.getTime()-off*60000);
    return local.toISOString().slice(0,10);
  }

  function renderTable(){
    const data=loadData(); const tbody=document.getElementById('tableBody');
    tbody.innerHTML='';
    data.forEach(it=>{
      tbody.innerHTML += `
        <tr>
          <td>${it.gcFrom||''}</td>
          <td>${it.serialCard||''}</td>
          <td>${it.activeDate||''}</td>
          <td>${it.lastActivityDate||''}</td>
          <td>${formatMoney(it.originalAmt)}</td>
          <td>${formatMoney(it.currentBalance)}</td>
        </tr>`;
    });
  }

  function fillResult(it, idx){
    selectedIndex = idx;
    document.getElementById('resultCard').classList.remove('hidden');
    document.getElementById('resultGcFrom').textContent = it.gcFrom || '-';
    document.getElementById('resultSerial').textContent = it.serialCard || '-';
    document.getElementById('resultCurrentBalance').textContent = formatMoney(it.currentBalance);
    document.getElementById('resultActiveDate').textContent = it.activeDate || '-';
    document.getElementById('resultLastActivityDate').textContent = it.lastActivityDate || '-';
    document.getElementById('resultOriginalAmt').textContent = formatMoney(it.originalAmt);

    document.getElementById('editGcFrom').value = it.gcFrom || '';
    document.getElementById('editSerial').value = it.serialCard || '';
    document.getElementById('editActiveDate').value = it.activeDate || '';
    document.getElementById('editLastActivityDate').value = it.lastActivityDate || '';
    document.getElementById('editOriginalAmt').value = it.originalAmt || '';
    document.getElementById('editCurrentBalance').value = it.currentBalance || '';

    document.getElementById('amountUsed').value='';
    document.getElementById('autoActivityDate').value = getTodayLocalDate();
  }

  function resetLookup(){
    selectedIndex=-1;
    document.getElementById('lookupSerial').value='';
    document.getElementById('resultCard').classList.add('hidden');
    setStatus('');
  }

  function searchBySerial(){
    const v = normalizeSerial(document.getElementById('lookupSerial').value);
    if(!v){ setStatus('Please type a serial card number.'); return; }
    const data=loadData();
    const idx=data.findIndex(x=>normalizeSerial(x.serialCard)===v);
    if(idx===-1){ setStatus('No gift card found.'); return; }
    fillResult(data[idx], idx);
    setStatus('Gift card found.');
  }

  function applyAmountUsed(){
    if(selectedIndex<0){ setStatus('Search first.'); return; }
    const amt = Number(document.getElementById('amountUsed').value);
    const cur = Number(document.getElementById('editCurrentBalance').value||0);
    if(Number.isNaN(amt) || amt<0){ setStatus('Invalid amount.'); return; }
    if(amt>cur){ setStatus('Amount too high.'); return; }

    const newBal = (cur-amt).toFixed(2);
    const date = document.getElementById('autoActivityDate').value || getTodayLocalDate();

    const data=loadData();
    data[selectedIndex].currentBalance = newBal;
    data[selectedIndex].lastActivityDate = date;

    saveData(data);
    renderTable();
    fillResult(data[selectedIndex], selectedIndex);
    setStatus(`New balance: ${formatMoney(newBal)}`);
  }

  function saveUpdate(){
    if(selectedIndex<0){ setStatus('Search first.'); return; }
    const data=loadData();
    const serial = document.getElementById('editSerial').value.trim();
    if(!serial){ setStatus('Serial required.'); return; }
    const dup = data.findIndex((x,i)=> i!==selectedIndex && normalizeSerial(x.serialCard)===normalizeSerial(serial));
    if(dup!==-1){ setStatus('Serial already exists.'); return; }

    data[selectedIndex] = {
      gcFrom: document.getElementById('editGcFrom').value.trim(),
      serialCard: serial,
      activeDate: document.getElementById('editActiveDate').value,
      lastActivityDate: document.getElementById('editLastActivityDate').value,
      originalAmt: document.getElementById('editOriginalAmt').value,
      currentBalance: document.getElementById('editCurrentBalance').value
    };

    saveData(data);
    renderTable();
    fillResult(data[selectedIndex], selectedIndex);
    setStatus('Updated.');
  }

  function showNewPanel(){ document.getElementById('newCardPanel').classList.remove('hidden'); }
  function hideNewPanel(){ document.getElementById('newCardPanel').classList.add('hidden'); }

  function saveNewCard(){
    const data=loadData();
    const serial = document.getElementById('newSerial').value.trim();
    if(!serial){ setStatus('Serial required.'); return; }
    const exists = data.some(x=>normalizeSerial(x.serialCard)===normalizeSerial(serial));
    if(exists){ setStatus('Serial already exists.'); return; }

    data.unshift({
      gcFrom: document.getElementById('newGcFrom').value.trim(),
      serialCard: serial,
      activeDate: document.getElementById('newActiveDate').value,
      lastActivityDate: document.getElementById('newLastActivityDate').value,
      originalAmt: document.getElementById('newOriginalAmt').value,
      currentBalance: document.getElementById('newCurrentBalance').value
    });

    saveData(data);
    renderTable();
    hideNewPanel();
    setStatus('New gift card added.');
  }

  function exportCsv(){
    const data=loadData();
    if(!data.length){ setStatus('No records to export.'); return; }
    let csv='GC From,Serial Card,Active Date,Last Activity Date,Original Amt,Current Balance\n';
    data.forEach(it=>{
      csv += `"${(it.gcFrom||'').replace(/"/g,'""')}","${(it.serialCard||'').replace(/"/g,'""')}","${(it.activeDate||'').replace(/"/g,'""')}","${(it.lastActivityDate||'').replace(/"/g,'""')}","${(it.originalAmt||'').replace(/"/g,'""')}","${(it.currentBalance||'').replace(/"/g,'""')}"\n`;
    });
    const blob=new Blob([csv],{type:'text/csv;charset=utf-8;'});
    const url=URL.createObjectURL(blob);
    const a=document.createElement('a'); a.href=url; a.download='gift_card_database.csv'; a.click();
    URL.revokeObjectURL(url);
    setStatus('CSV exported.');
  }

  function clearAll(){ if(!confirm('Delete all gift card records?')) return; localStorage.removeItem(STORAGE_KEY); renderTable(); resetLookup(); setStatus('All records deleted.'); }

  function mapRow(row){
    const k = Object.keys(row).reduce((a,key)=>{ a[key.trim().toLowerCase()] = row[key]; return a; },{});
    return {
      gcFrom: k['gc from'] || '',
      serialCard: (k['serial card'] || k['serial'] || '').toString().trim(),
      activeDate: k['active date'] || k['activedate'] || '',
      lastActivityDate: k['last activity date'] || k['lastactivitydate'] || '',
      originalAmt: k['original amt'] || k['original activity amt'] || '',
      currentBalance: k['current balance'] || ''
    };
  }

  function importWorkbook(file){
    const reader=new FileReader();
    reader.onload=function(e){
      try{
        const data=new Uint8Array(e.target.result);
        const wb=XLSX.read(data,{type:'array'});
        const sh=wb.Sheets[wb.SheetNames[0]];
        const rows=XLSX.utils.sheet_to_json(sh,{defval:''});
        const mapped = rows.map(mapRow).filter(x=>x.serialCard);
        const uniq=new Map(); mapped.forEach(x=>uniq.set(normalizeSerial(x.serialCard),x));
        const finalData = Array.from(uniq.values());
        saveData(finalData);
        renderTable();
        resetLookup();
        setStatus(`Imported ${finalData.length} gift cards.`);
      }catch{
        setStatus('Could not import file. Check format.');
      }
    };
    reader.readAsArrayBuffer(file);
  }

  function importSelectedFile(){
    const f=document.getElementById('excelFile').files[0];
    if(!f){ setStatus('Choose a file first.'); return; }
    importWorkbook(f);
  }

  document.getElementById('searchBtn').addEventListener('click', searchBySerial);
  document.getElementById('lookupSerial').addEventListener('keydown', e=>{ if(e.key==='Enter') searchBySerial(); });
  document.getElementById('resetLookupBtn').addEventListener('click', resetLookup);
  document.getElementById('deductBtn').addEventListener('click', applyAmountUsed);
  document.getElementById('saveUpdateBtn').addEventListener('click', saveUpdate);
  document.getElementById('addNewBtn').addEventListener('click', showNewPanel);
  document.getElementById('cancelNewBtn').addEventListener('click', hideNewPanel);
  document.getElementById('saveNewBtn').addEventListener('click', saveNewCard);
  document.getElementById('exportBtn').addEventListener('click', exportCsv);
  document.getElementById('clearBtn').addEventListener('click', clearAll);
  document.getElementById('importBtn').addEventListener('click', importSelectedFile);

  document.getElementById('autoActivityDate').value = getTodayLocalDate();
  renderTable();
</script>
</body>
</html>
