<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="UTF-8">
<title>Statement Viewer</title>
<style>
  * { box-sizing: border-box; }
  body {
    margin: 0;
    font-family: -apple-system, "Helvetica Neue", "Thonburi", "Sukhumvit Set", sans-serif;
    background: #ffffff;
    color: #1c1c1e;
  }

  .topbar {
    display: flex;
    align-items: center;
    padding: 14px 16px;
    border-bottom: 1px solid #e5e5e5;
  }
  .topbar .close {
    font-size: 22px;
    color: #8e8e93;
    margin-right: 16px;
    line-height: 1;
    cursor: pointer;
    background: none;
    border: none;
  }
  .topbar .fname {
    font-size: 15px;
    color: #1c1c1e;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  /* Lock screen */
  #lockScreen {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: flex-start;
    padding-top: 130px;
    min-height: 70vh;
    transition: opacity 0.35s ease;
  }
  .lock-icon { width: 34px; height: 34px; margin-bottom: 14px; }
  .lock-icon svg { width: 100%; height: 100%; }
  .lock-title { font-size: 15px; color: #1c1c1e; margin-bottom: 22px; }
  #pwField {
    width: 260px;
    padding: 10px 2px;
    font-size: 16px;
    border: none;
    border-bottom: 1px solid #c6c6c8;
    outline: none;
    background: transparent;
    color: #1c1c1e;
  }
  #pwField::placeholder { color: #b0b0b3; }
  #pwField.error { border-bottom-color: #ff3b30; }
  .submit-row { width: 260px; text-align: right; margin-top: 18px; }
  #submitBtn { background: none; border: none; color: #007aff; font-size: 15px; cursor: pointer; padding: 4px 0; }
  .error-msg { width: 260px; color: #ff3b30; font-size: 12px; margin-top: 10px; visibility: hidden; }
  .error-msg.show { visibility: visible; }

  /* Content */
  #content {
    display: none;
    padding: 16px 14px 60px;
    max-width: 760px;
    margin: 0 auto;
    animation: fadeIn 0.4s ease;
  }
  @keyframes fadeIn {
    from { opacity: 0; transform: translateY(6px); }
    to { opacity: 1; transform: translateY(0); }
  }
  .stmt-header { text-align: center; margin-bottom: 16px; }
  .stmt-header h1 { font-size: 16px; margin: 0 0 6px; font-weight: 600; }
  .stmt-header p { font-size: 11.5px; color: #6e6e73; margin: 2px 0; line-height: 1.5; }

  /* Tabs */
  .tabs {
    display: flex;
    gap: 6px;
    margin-bottom: 14px;
    border-bottom: 1px solid #e5e5e5;
  }
  .tab-btn {
    flex: 1;
    background: none;
    border: none;
    padding: 10px 4px;
    font-size: 13.5px;
    font-weight: 600;
    color: #8e8e93;
    cursor: pointer;
    border-bottom: 2px solid transparent;
  }
  .tab-btn.active { color: #1c1c1e; border-bottom-color: #007aff; }

  .tab-panel { display: none; }
  .tab-panel.active { display: block; }

  /* Summary cards */
  .cards {
    display: flex;
    gap: 8px;
    margin-bottom: 16px;
  }
  .card {
    flex: 1;
    border: 1px solid #e5e5e5;
    border-radius: 10px;
    padding: 10px;
    text-align: center;
  }
  .card .label { font-size: 10.5px; color: #6e6e73; margin-bottom: 4px; }
  .card .value { font-size: 15px; font-weight: 700; font-variant-numeric: tabular-nums; }
  .card.income .value { color: #1e8e5a; }
  .card.expense .value { color: #d1453b; }
  .card.net .value { color: #1c1c1e; }

  table { width: 100%; border-collapse: collapse; font-size: 12px; }
  thead th {
    text-align: left;
    padding: 7px 5px;
    border-bottom: 1px solid #d1d1d6;
    color: #6e6e73;
    font-weight: 600;
    font-size: 10.5px;
  }
  tbody td { padding: 7px 5px; border-bottom: 1px solid #f0f0f2; vertical-align: top; }
  tbody tr:last-child td { border-bottom: none; }
  td.num { text-align: right; white-space: nowrap; font-variant-numeric: tabular-nums; }
  td.date { white-space: nowrap; color: #3a3a3c; }
  td.withdraw { color: #d1453b; }
  td.deposit { color: #1e8e5a; }
  td.balance { color: #1c1c1e; font-weight: 500; }

  .channel-cards {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 8px;
    margin-bottom: 16px;
  }
  .channel-card {
    border: 1px solid #e5e5e5;
    border-radius: 10px;
    padding: 9px 6px;
    text-align: center;
  }
  .channel-card .ch-label { font-size: 10.5px; color: #6e6e73; margin-bottom: 4px; }
  .channel-card .ch-value { font-size: 14px; font-weight: 700; color: #1c1c1e; font-variant-numeric: tabular-nums; }
  .channel-card .ch-count { font-size: 10px; color: #8e8e93; margin-top: 2px; }

  .summary {
    margin-top: 16px;
    padding-top: 12px;
    border-top: 1px solid #d1d1d6;
    font-size: 13px;
  }
  .summary-row { display: flex; justify-content: space-between; padding: 4px 6px; }
  .summary-row.total { font-weight: 600; border-top: 1px solid #e5e5e5; margin-top: 4px; padding-top: 8px; }
</style>
</head>
<body>


<div class="topbar">
  <button class="close" aria-label="close">&times;</button>
  <div class="fname">Statement_31AUG2026_341ee4c8-e17a-49cb-9fb8-6a5c57e1d114.pdf</div>
</div>

<div id="lockScreen">
  <div class="lock-icon">
    <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
      <rect x="5" y="10" width="14" height="10" rx="2" stroke="#007aff" stroke-width="1.6"/>
      <path d="M8 10V7a4 4 0 0 1 8 0v3" stroke="#007aff" stroke-width="1.6"/>
      <circle cx="12" cy="14.5" r="1.3" fill="#007aff"/>
    </svg>
  </div>
  <div class="lock-title">ต้องมีรหัสผ่าน</div>
  <input type="password" id="pwField" placeholder="รหัสผ่าน" autocomplete="off">
  <div class="submit-row"><button id="submitBtn">ส่ง</button></div>
  <div class="error-msg" id="errorMsg">รหัสผ่านไม่ถูกต้อง กรุณาลองอีกครั้ง</div>
</div>

<div id="content">
  <div class="stmt-header">
    <h1>Fresh Yogurt — สรุปรายรับ-รายจ่าย เดือนสิงหาคม 2569</h1>
    <p>Fresh yogurt (ซุ้มหน้าธนาคารกรุงเทพ สาขาศาลายา) หน้ามหาวิทยาลัยมหิดล ศาลายา</p>
  </div>

  <div class="cards">
    <div class="card income">
      <div class="label">รายรับ (ใบเสร็จปกติ)</div>
      <div class="value" id="totalIncome">-</div>
    </div>
    <div class="card expense">
      <div class="label">รายจ่าย (ถอนจากบัญชี)</div>
      <div class="value" id="totalExpense">-</div>
    </div>
    <div class="card net">
      <div class="label">คงเหลือสุทธิ</div>
      <div class="value" id="netTotal">-</div>
    </div>
  </div>

  <div class="tabs">
    <button class="tab-btn active" data-tab="income">รายรับ</button>
    <button class="tab-btn" data-tab="expense">รายจ่าย (ถอน)</button>
  </div>

  <!-- INCOME TAB -->
  <div class="tab-panel active" id="tab-income">
    <div class="channel-cards" id="channelCards"></div>

    <table>
      <thead>
        <tr>
          <th>วันที่</th>
          <th>ช่องทางชำระ</th>
          <th style="text-align:right">ยอดสุทธิ</th>
        </tr>
      </thead>
      <tbody id="incomeBody"></tbody>
    </table>
    <div class="summary">
      <div class="summary-row"><span>เงินสด</span><span id="sumCash">-</span></div>
      <div class="summary-row"><span>ไทยช่วยไทย</span><span id="sumThaiChuayThai">-</span></div>
      <div class="summary-row"><span>โอน</span><span id="sumTransfer">-</span></div>
      <div class="summary-row"><span>จำนวนใบเสร็จ (ไม่รวมที่ยกเลิก)</span><span id="incomeCount">-</span></div>
      <div class="summary-row total"><span>รวมรายรับ</span><span id="incomeSum">-</span></div>
    </div>
  </div>

  <!-- EXPENSE TAB -->
  <div class="tab-panel" id="tab-expense">
    <table>
      <thead>
        <tr>
          <th>วันที่ / เวลา</th>
          <th>รายการ</th>
          <th>รายละเอียด</th>
          <th style="text-align:right">ถอน</th>
          <th style="text-align:right">คงเหลือ</th>
        </tr>
      </thead>
      <tbody id="expenseBody"></tbody>
    </table>
    <div class="summary">
      <div class="summary-row"><span>จำนวนรายการถอน</span><span id="expenseCount">-</span></div>
      <div class="summary-row total"><span>รวมรายจ่าย</span><span id="expenseSum">-</span></div>
    </div>
  </div>

</div>

<script>
  const CORRECT_PASSWORD = "greekzly888";

  /* ---------- EXPENSE DATA (from bank statement) ---------- */
  const expenses = [
    ["02/08/69, 14.58น.", "ฉลาก Yogurt drink", "Dollaya Printing", 95.00, 95.00],
    ["03/08/69, 15.54น.", "น้ำผึ้ง", "7-Eleven", 100.00, 290.00],
    ["04/08/69, 12.48น.", "Cup (130 cc)", "Shoppee", 79.00, 369.00],
    ["05/08/69, 15.03น.", "Thermometer", "Shoppee", 42.00, 411.00],
    ["05/08/69, 15.03น.", "Spoon", "Shoppee", 52.00, 463.00],
    ["07/08/69, 11.33น.", "นมโค(รสจืด) 2ลิตร", "บจก.ปโยสิน พันธมิตรร่วมค้า 999", 516.00, 979.00],
    ["07/08/69, 16.37น.", "ค่าน้ำมัน", "-", 100.00, 1079.00],
    ["10/08/69, 16.52น.", "Bottle 150 g", "Shoppee", 292.00, 1371.00],
    ["12/08/69, 10.12น.", "TikTok", "TikTok", 135.50, 1506.50],
    ["13/08/69, 07.33น.", "น้ำผึ้ง", "7-Eleven", 70.00, 1576.50],
    ["14/08/69, 08.06น.", "นมโค(รสจืด) 2ลิตร", "บจก.ปโยสิน พันธมิตรร่วมค้า 999", 516.00, 2092.50],
    ["14/08/69, 17.40น.", "Meji milk fat 0%(2,000g), Yolida (450g)", "7-Eleven", 262.00, 2354.50],
    ["18/08/69, 16.30น.", "ค่าน้ำมัน", "-", 100.00, 2454.50],
    ["20/08/69, 12.19น.", "Cup (take away)", "TikTok", 121.00, 2575.50],
    ["20/08/69, 12.25น.", "TikTok", "TikTok", 132.10, 2707.60],
    ["21/08/69, 12.25น.", "ถุงแกลลอน", "TikTok", 57.70, 2765.30],
    ["21/08/69, 13.28น.", "โถจ่ายน้ำผึ้ง", "TikTok", 135.50, 2900.80],
    ["22/08/69, 00.29น.", "Stamp EXP", "TikTok", 72.55, 2973.35],
    ["22/08/69, 07.34น.", "ถุงกระดาษ", "มงคลภัณฑ์", 150.00, 3123.35],
    ["22/08/69, 13.37น.", "Cooler bag", "TikTok", 74.40, 3197.75],
    ["26/08/69, 19.05น.", "Honey granola & Chocolate granola", "Tops", 502.00, 3699.75],
    ["26/08/69, 19.36น.", "นมโค(รสจืด) 2ลิตร", "บจก.ปโยสิน พันธมิตรร่วมค้า 999", 516.00, 4215.75],
    ["27/08/69, 13.42น.", "ค่าน้ำมัน", "-", 100.00, 4315.75],
    ["31/08/69, 13.36น.", "TikTok", "TikTok", 55.20, 4370.95],
    ["31/08/69, 14.10น.", "ฉลาก Yogurt drink", "Dollaya Printing", 95.00, 4465.95],
  ];

  /* ---------- INCOME DATA (from receipt history) ---------- */
  /* status: "ok" = ใบเสร็จปกติ, anything else = ยกเลิก/ผิดพลาด (ตัดออก ไม่นำมาคิด) */
  const receiptsRaw = [
    ["04/08", "พร้อมเพย์", 59, "void"],
    ["04/08", "ไทยช่วยไทย", 61, "ok"],
    ["04/08", "ไทยช่วยไทย", 94, "ok"],
    ["05/08", "ไทยช่วยไทย", 74, "ok"],
    ["05/08", "เงินสด", 94, "ok"],
    ["05/08", "เงินสด", 59, "ok"],
    ["05/08", "ไทยช่วยไทย", 70, "ok"],
    ["05/08", "-", 0, "ok"],
    ["06/08", "-", 0, "ok"],
    ["06/08", "เงินสด", 61, "ok"],
    ["06/08", "เงินสด", 94, "ok"],
    ["06/08", "ไทยช่วยไทย", 105, "void"],
    ["06/08", "ไทยช่วยไทย", 105, "void"],
    ["06/08", "ไทยช่วยไทย", 59, "ok"],
    ["06/08", "ไทยช่วยไทย", 100, "ok"],
    ["06/08", "ไทยช่วยไทย", 98, "ok"],
    ["06/08", "ไทยช่วยไทย", 20, "ok"],
    ["06/08", "พร้อมเพย์", 70, "ok"],
    ["06/08", "ไทยช่วยไทย", 49, "ok"],
    ["07/08", "เงินสด", 35, "void"],
    ["07/08", "ไทยช่วยไทย", 61, "ok"],
    ["07/08", "พร้อมเพย์", 59, "ok"],
    ["07/08", "ไทยช่วยไทย", 49, "ok"],
    ["08/08", "พร้อมเพย์", 23.6, "ok"],
    ["08/08", "พร้อมเพย์", 98, "ok"],
    ["09/08", "พร้อมเพย์", 59, "ok"],
    ["09/08", "พร้อมเพย์", 59, "ok"],
    ["09/08", "พร้อมเพย์", 49, "ok"],
    ["10/08", "-", 0, "void"],
    ["10/08", "-", 0, "void"],
    ["10/08", "ไทยช่วยไทย", 20, "void"],
    ["10/08", "พร้อมเพย์", 74, "ok"],
    ["10/08", "ไทยช่วยไทย", 59, "ok"],
    ["10/08", "พร้อมเพย์", 59, "ok"],
    ["10/08", "พร้อมเพย์", 64, "ok"],
    ["10/08", "พร้อมเพย์", 51, "ok"],
    ["10/08", "พร้อมเพย์", 61, "ok"],
    ["10/08", "เงินสด", 74, "void"],
    ["10/08", "ไทยช่วยไทย", 59, "void"],
    ["10/08", "ไทยช่วยไทย", 59, "void"],
    ["10/08", "ไทยช่วยไทย", 64, "void"],
    ["11/08", "พร้อมเพย์", 118, "void"],
    ["11/08", "พร้อมเพย์", 118, "void"],
    ["11/08", "เงินสด", 118, "ok"],
    ["11/08", "พร้อมเพย์", 35, "void"],
    ["11/08", "เงินสด", 118, "void"],
    ["11/08", "ไทยช่วยไทย", 118, "void"],
    ["11/08", "ไทยช่วยไทย", 35, "void"],
    ["11/08", "ไทยช่วยไทย", 35, "void"],
    ["11/08", "พร้อมเพย์", 35, "void"],
    ["11/08", "พร้อมเพย์", 118, "void"],
    ["11/08", "ไทยช่วยไทย", 35, "ok"],
    ["11/08", "ไทยช่วยไทย", 118, "ok"],
    ["11/08", "ไทยช่วยไทย", 35, "ok"],
    ["13/08", "เงินสด", 35, "ok"],
    ["13/08", "พร้อมเพย์", 59, "ok"],
    ["13/08", "ไทยช่วยไทย", 59, "ok"],
    ["13/08", "เงินสด", 35, "ok"],
    ["13/08", "ไทยช่วยไทย", 70, "ok"],
    ["14/08", "ไทยช่วยไทย", 59, "ok"],
    ["14/08", "ไทยช่วยไทย", 35, "ok"],
    ["16/08", "เงินสด", 59, "ok"],
    ["16/08", "เงินสด", 49, "ok"],
    ["16/08", "พร้อมเพย์", 61, "ok"],
    ["16/08", "พร้อมเพย์", 59, "ok"],
    ["16/08", "พร้อมเพย์", 59, "ok"],
    ["16/08", "พร้อมเพย์", 70, "ok"],
    ["17/08", "เงินสด", 64, "ok"],
    ["17/08", "พร้อมเพย์", 105, "ok"],
    ["18/08", "ไทยช่วยไทย", 69, "ok"],
    ["18/08", "พร้อมเพย์", 49, "ok"],
    ["18/08", "พร้อมเพย์", 35, "ok"],
    ["20/08", "พร้อมเพย์", 59, "ok"],
    ["21/08", "พร้อมเพย์", 71, "ok"],
    ["22/08", "ไทยช่วยไทย", 59, "ok"],
    ["22/08", "พร้อมเพย์", 69, "ok"],
    ["22/08", "พร้อมเพย์", 61, "ok"],
    ["23/08", "พร้อมเพย์", 106, "void"],
    ["23/08", "พร้อมเพย์", 49, "void"],
    ["23/08", "พร้อมเพย์", 59, "void"],
    ["23/08", "พร้อมเพย์", 105, "void"],
    ["23/08", "เงินสด", 35, "void"],
    ["23/08", "พร้อมเพย์", 59, "void"],
    ["23/08", "พร้อมเพย์", 59, "void"],
    ["23/08", "พร้อมเพย์", 71, "ok"],
    ["23/08", "พร้อมเพย์", 35, "ok"],
    ["23/08", "พร้อมเพย์", 59, "ok"],
    ["23/08", "เงินสด", 140, "ok"],
    ["23/08", "เงินสด", 35, "ok"],
    ["23/08", "พร้อมเพย์", 49, "ok"],
    ["23/08", "พร้อมเพย์", 35, "ok"],
    ["24/08", "เงินสด", 141, "ok"],
    ["24/08", "พร้อมเพย์", 59, "ok"],
    ["24/08", "พร้อมเพย์", 59, "ok"],
    ["24/08", "เงินสด", 35, "ok"],
    ["24/08", "เงินสด", 35, "ok"],
    ["25/08", "พร้อมเพย์", 59, "ok"],
    ["26/08", "พร้อมเพย์", 49, "ok"],
    ["27/08", "ไทยช่วยไทย", 59, "ok"],
    ["27/08", "พร้อมเพย์", 61, "ok"],
  ];

  /* ตัดรายการที่ยกเลิก/ผิดพลาดออกไปเลย ไม่นำมาคิดหรือแสดงผล */
  const receipts = receiptsRaw.filter(r => r[3] === "ok");

  /* จัดกลุ่มช่องทางชำระเงิน */
  const CHANNEL_MAP = {
    "เงินสด": "เงินสด",
    "ไทยช่วยไทย": "ไทยช่วยไทย",
    "พร้อมเพย์": "โอน",
    "-": "ไม่ระบุ/อื่นๆ",
  };

  function fmt(n) {
    if (n === null || n === undefined) return "-";
    return n.toLocaleString("en-US", { minimumFractionDigits: 2, maximumFractionDigits: 2 });
  }

  function renderExpenses() {
    const body = document.getElementById("expenseBody");
    body.innerHTML = expenses.map(t => `
      <tr>
        <td class="date">${t[0]}</td>
        <td>${t[1]}</td>
        <td>${t[2]}</td>
        <td class="num withdraw">${fmt(t[3])}</td>
        <td class="num balance">${fmt(t[4])}</td>
      </tr>
    `).join("");
    const total = expenses.reduce((s, t) => s + t[3], 0);
    document.getElementById("expenseCount").textContent = expenses.length + " รายการ";
    document.getElementById("expenseSum").textContent = fmt(total) + " บาท";
    return total;
  }

  function renderIncome() {
    const body = document.getElementById("incomeBody");
    body.innerHTML = receipts.map(r => `
      <tr>
        <td class="date">${r[0]}</td>
        <td>${CHANNEL_MAP[r[1]] || r[1]}</td>
        <td class="num deposit">${fmt(r[2])}</td>
      </tr>
    `).join("");
    const total = receipts.reduce((s, r) => s + r[2], 0);
    document.getElementById("incomeCount").textContent = receipts.length + " รายการ";
    document.getElementById("incomeSum").textContent = fmt(total) + " บาท";

    // Channel breakdown
    const groups = {};
    receipts.forEach(r => {
      const ch = CHANNEL_MAP[r[1]] || r[1];
      if (!groups[ch]) groups[ch] = { sum: 0, count: 0 };
      groups[ch].sum += r[2];
      groups[ch].count += 1;
    });
    const order = ["เงินสด", "ไทยช่วยไทย", "โอน", "ไม่ระบุ/อื่นๆ"];
    const cardsHtml = order
      .filter(ch => groups[ch])
      .map(ch => `
        <div class="channel-card">
          <div class="ch-label">${ch}</div>
          <div class="ch-value">${fmt(groups[ch].sum)}</div>
          <div class="ch-count">${groups[ch].count} รายการ</div>
        </div>
      `).join("");
    document.getElementById("channelCards").innerHTML = cardsHtml;

    document.getElementById("sumCash").textContent = fmt(groups["เงินสด"] ? groups["เงินสด"].sum : 0) + " บาท";
    document.getElementById("sumThaiChuayThai").textContent = fmt(groups["ไทยช่วยไทย"] ? groups["ไทยช่วยไทย"].sum : 0) + " บาท";
    document.getElementById("sumTransfer").textContent = fmt(groups["โอน"] ? groups["โอน"].sum : 0) + " บาท";

    return total;
  }

  function renderTotals(incomeTotal, expenseTotal) {
    document.getElementById("totalIncome").textContent = fmt(incomeTotal);
    document.getElementById("totalExpense").textContent = fmt(expenseTotal);
    document.getElementById("netTotal").textContent = fmt(incomeTotal - expenseTotal);
  }

  function setupTabs() {
    document.querySelectorAll(".tab-btn").forEach(btn => {
      btn.addEventListener("click", () => {
        document.querySelectorAll(".tab-btn").forEach(b => b.classList.remove("active"));
        document.querySelectorAll(".tab-panel").forEach(p => p.classList.remove("active"));
        btn.classList.add("active");
        document.getElementById("tab-" + btn.dataset.tab).classList.add("active");
      });
    });
  }

  function tryUnlock() {
    const field = document.getElementById("pwField");
    const errorMsg = document.getElementById("errorMsg");
    const val = field.value;

    if (val === CORRECT_PASSWORD) {
      document.getElementById("lockScreen").style.opacity = "0";
      setTimeout(() => {
        document.getElementById("lockScreen").style.display = "none";
        document.getElementById("content").style.display = "block";
      }, 300);
    } else {
      field.classList.add("error");
      errorMsg.classList.add("show");
      field.value = "";
      field.focus();
    }
  }

  document.getElementById("submitBtn").addEventListener("click", tryUnlock);
  document.getElementById("pwField").addEventListener("keydown", (e) => {
    if (e.key === "Enter") tryUnlock();
    document.getElementById("pwField").classList.remove("error");
    document.getElementById("errorMsg").classList.remove("show");
  });

  const expenseTotal = renderExpenses();
  const incomeTotal = renderIncome();
  renderTotals(incomeTotal, expenseTotal);
  setupTabs();
</script>

</body>
</html>
