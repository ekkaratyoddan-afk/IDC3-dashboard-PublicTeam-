<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>IDC3 Master Infrastructure Dashboard & Editor</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
<style>
  :root{
    --bg:#f4f6f9; --card:#ffffff; --txt:#2d3748; --muted:#718096;
    --primary:#3182ce; --ok:#38a169; --warn:#d69e2e; --crit:#e53e3e; --border:#e2e8f0;
  }
  *{box-sizing:border-box;margin:0;padding:0}
  body{font-family:"Segoe UI",Tahoma,"Sarabun",sans-serif;background:var(--bg);color:var(--txt);padding:20px;line-height:1.5}
  
  header{background:var(--card);border:1px solid var(--border);border-radius:12px;padding:20px 24px;margin-bottom:20px;display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:12px;box-shadow:0 1px 3px rgba(0,0,0,.05)}
  h1{font-size:22px;color:#1a202c}
  .sub{color:var(--muted);font-size:13px;margin-top:2px}
  .header-actions{display:flex;gap:10px;align-items:center}
  .status-badge{background:#ebf8ff;color:var(--primary);padding:6px 14px;border-radius:99px;font-weight:600;font-size:13px;display:flex;align-items:center;gap:6px}
  .dot{width:8px;height:8px;background:var(--ok);border-radius:50%;display:inline-block}

  /* Navigation Tabs */
  .tabs{display:flex;gap:8px;margin-bottom:20px;flex-wrap:wrap}
  .tab-btn{background:var(--card);border:1px solid var(--border);padding:10px 14px;border-radius:8px;cursor:pointer;font-weight:600;color:var(--muted);font-size:12.5px;transition:all .2s}
  .tab-btn.active{background:var(--primary);color:#fff;border-color:var(--primary)}

  .tab-content{display:none}
  .tab-content.active{display:block}

  .section-title{font-size:16px;font-weight:700;margin:20px 0 12px;color:#2b6cb0;display:flex;align-items:center;justify-content:space-between}

  /* Summary Cards */
  .cards-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(200px,1fr));gap:16px;margin-bottom:20px}
  .card{background:var(--card);border:1px solid var(--border);border-radius:12px;padding:18px;box-shadow:0 1px 3px rgba(0,0,0,.05)}
  .card .label{color:var(--muted);font-size:11.5px;text-transform:uppercase;letter-spacing:.5px;font-weight:600}
  .card .value{font-size:24px;font-weight:700;margin-top:6px;color:#1a202c}
  .card .unit{font-size:13px;color:var(--muted);font-weight:normal}

  /* Tables */
  .table-box{background:var(--card);border:1px solid var(--border);border-radius:12px;overflow-x:auto;margin-bottom:20px;box-shadow:0 1px 3px rgba(0,0,0,.05)}
  table{width:100%;border-collapse:collapse;font-size:12.5px;white-space:nowrap}
  th{background:#f7fafc;color:var(--muted);text-align:left;padding:12px 14px;font-weight:600;border-bottom:1px solid var(--border)}
  td{padding:11px 14px;border-bottom:1px solid var(--border);color:#4a5568}
  tr:last-child td{border-bottom:none}
  tr:hover td{background:#fafbfc}

  .btn{background:var(--primary);color:#fff;border:none;padding:6px 12px;border-radius:6px;cursor:pointer;font-size:12px;font-weight:600;transition:opacity .2s}
  .btn:hover{opacity:.85}
  .btn-sm{padding:4px 8px;font-size:11px}
  .btn-danger{background:var(--crit)}
  .btn-secondary{background:var(--muted)}

  /* Clickable Link */
  .ups-link{color:var(--primary);cursor:pointer;font-weight:700;text-decoration:underline}
  .ups-link:hover{color:#2b6cb0}

  /* Modal Edit & Detail */
  .modal{display:none;position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.5);justify-content:center;align-items:center;z-index:1000}
  .modal-content{background:var(--card);padding:24px;border-radius:12px;width:100%;max-width:700px;box-shadow:0 4px 12px rgba(0,0,0,0.15);max-height:85vh;overflow-y:auto}
  .modal-content h3{margin-bottom:16px;color:#1a202c}
  .form-group{margin-bottom:12px}
  .form-group label{display:block;font-size:12px;color:var(--muted);margin-bottom:4px;font-weight:600}
  .form-group input{width:100%;padding:8px 12px;border:1px solid var(--border);border-radius:6px;font-size:13px}

  .grid-2{display:grid;grid-template-columns:repeat(auto-fit,minmax(450px,1fr));gap:20px}
  .chart-container{background:var(--card);border:1px solid var(--border);border-radius:12px;padding:20px;box-shadow:0 1px 3px rgba(0,0,0,.05)}
  .chart-box{height:240px;margin-top:10px}

  footer{text-align:center;color:var(--muted);font-size:12px;margin-top:30px;padding:15px}
</style>
</head>
<body>

<header>
  <div>
    <h1>⚡ IDC3 Master Infrastructure Dashboard &amp; Editor</h1>
    <div class="sub">ระบบฐานข้อมูลสารสนเทศและบริหารจัดการวิศวกรรม (คลิกที่ชื่อระบบใน Spare Parts เพื่อดูประวัติได้ทันที)</div>
  </div>
  <div class="header-actions">
    <button class="btn btn-secondary btn-sm" onclick="resetData()">🔄 รีเซ็ตข้อมูลเริ่มต้น</button>
    <div class="status-badge"><span class="dot"></span> ระบบออนไลน์</div>
  </div>
</header>

<!-- Navigation Tabs -->
<div class="tabs">
  <button class="tab-btn active" onclick="switchTab(event, 't1')">📊 1. ภาพรวมระบบ</button>
  <button class="tab-btn" onclick="switchTab(event, 't2')">🔋 2. รอบเปลี่ยนแบต</button>
  <button class="tab-btn" onclick="switchTab(event, 't3')">📅 3. Spare Parts</button>
  <button class="tab-btn" onclick="switchTab(event, 't4')">📋 4. NC Records</button>
  <button class="tab-btn" onclick="switchTab(event, 't5')">🚨 5. EC Records</button>
  <button class="tab-btn" onclick="switchTab(event, 't6')">📞 6. Vendor &amp; PM</button>
  <button class="tab-btn" onclick="switchTab(event, 't7')">📦 7. PO แบต 2025</button>
  <button class="tab-btn" onclick="switchTab(event, 't8')">📦 8. PO แบต 2026</button>
</div>

<!-- ================= TAB 1: OVERVIEW ================= -->
<div id="t1" class="tab-content active">
  <div class="cards-grid">
    <div class="card"><div class="label">Total UPS Units</div><div class="value">11 <span class="unit">ยูนิต</span></div></div>
    <div class="card"><div class="label">CRAH Total Units</div><div class="value">62 <span class="unit">เครื่อง</span></div></div>
    <div class="card"><div class="label">Total IT Load (Sim)</div><div class="value">321.4 <span class="unit">kW</span></div></div>
    <div class="card"><div class="label">สถานะแจ้งเตือนวิกฤต</div><div class="value" style="color:var(--ok)">0 <span class="unit">ระบบ</span></div></div>
  </div>

  <div class="section-title">⚡ สรุปจำนวนและประเภท UPS แยกตามอาคาร (รวม 11 ยูนิต)</div>
  <div class="table-box">
    <table>
      <thead>
        <tr><th>อาคาร / โซน</th><th>UPS 400 kVA</th><th>UPS 30 kVA</th><th>UPS 60 kVA</th><th>รวมแต่ละอาคาร</th></tr>
      </thead>
      <tbody>
        <tr><td><b>Phase 1 (อาคาร UT Building)</b></td><td>8 ตัว (UPS-1A..1D, 2A..2D)</td><td>2 ตัว</td><td>-</td><td><b>10 เครื่อง</b></td></tr>
        <tr><td><b>อาคาร Office</b></td><td>-</td><td>-</td><td>1 ตัว</td><td><b>1 เครื่อง</b></td></tr>
        <tr><td><b>รวมทั้งหมด</b></td><td><b>8 ตัว</b></td><td><b>2 ตัว</b></td><td><b>1 ตัว</b></td><td><b style="color:var(--primary)">11 เครื่อง</b></td></tr>
      </tbody>
    </table>
  </div>

  <div class="section-title">❄️ สรุปจำนวนแอร์ CRAH แยกตามอาคารและเฟส (รวม 62 เครื่อง)</div>
  <div class="table-box">
    <table>
      <thead>
        <tr><th>โซน / เฟส</th><th>อาคาร DC Hall</th><th>อาคาร UT Building</th><th>รวมแต่ละเฟส</th></tr>
      </thead>
      <tbody>
        <tr><td><b>Phase 1</b></td><td>30 ตัว</td><td>8 ตัว</td><td><b>38 เครื่อง</b></td></tr>
        <tr><td><b>Phase 2</b></td><td>16 ตัว</td><td>8 ตัว</td><td><b>24 เครื่อง</b></td></tr>
        <tr><td><b>รวมทั้งหมดแยกตามอาคาร</b></td><td><b>46 ตัว</b></td><td><b>16 ตัว</b></td><td><b style="color:var(--primary)">62 เครื่อง</b></td></tr>
      </tbody>
    </table>
  </div>

  <div class="section-title">🔌 รายชื่ออุปกรณ์ระบบไฟฟ้าหลัก (คลิกที่ชื่อ UPS เพื่อดูประวัติการซ่อม)
    <button class="btn btn-sm" onclick="openAddModal('ups')">+ เพิ่ม UPS</button>
  </div>
  <div class="table-box">
    <table>
      <thead>
        <tr><th>สถานที่ (Location)</th><th>ชื่อย่อ (UPS/Unit)</th><th>ยี่ห้อ (Brand)</th><th>รุ่น (Model)</th><th>Serial Number</th><th>รอบ PM/ปี</th><th>จัดการ</th></tr>
      </thead>
      <tbody id="tbl-ups"></tbody>
    </table>
  </div>

  <div class="grid-2">
    <div class="chart-container">
      <h3 style="font-size:14px;color:#2d3748">📈 แนวโน้มโหลด UPS หลัก (kW)</h3>
      <div class="chart-box"><canvas id="loadChart"></canvas></div>
    </div>
    <div class="chart-container">
      <h3 style="font-size:14px;color:#2d3748">🌡️ อุณหภูมิระบบแอร์ภาพรวม (Supply / Return)</h3>
      <div class="chart-box"><canvas id="tempChart"></canvas></div>
    </div>
  </div>
</div>

<!-- ================= TAB 2: BATTERY DEEP DIVE ================= -->
<div id="t2" class="tab-content">
  <div class="section-title">🔋 เกณฑ์ความต้านทาน (Impedance) แยกตามขนาด UPS</div>
  <div class="cards-grid" style="margin-bottom:20px">
    <div class="card" style="border-left:4px solid var(--crit)">
      <div class="label">UPS 400kVA (Baseline 3.6 mΩ)</div>
      <div class="value" style="font-size:16px;margin-top:4px;color:var(--crit)">เปลี่ยน: ≥ 4.68 mΩ (+30%)</div>
      <div class="value" style="font-size:16px;margin-top:2px;color:var(--warn)">รับได้: ≤ 4.32 mΩ (+20%)</div>
      <div class="value" style="font-size:16px;margin-top:2px;color:var(--ok)">ปลอดภัย: ≤ 3.96 mΩ (+10%)</div>
    </div>
    <div class="card" style="border-left:4px solid var(--warn)">
      <div class="label">UPS 30kVA (Standard 9Ah)</div>
      <div class="value" style="font-size:16px;margin-top:4px;color:var(--crit)">เปลี่ยน: ตามค่า IR ผู้ผลิต / ค่าความจุลดลง >20%</div>
      <div class="value" style="font-size:16px;margin-top:2px;color:var(--ok)">ปลอดภัย: ตรวจวัด IR รายไตรมาสปกติ</div>
    </div>
    <div class="card" style="border-left:4px solid var(--primary)">
      <div class="label">UPS 60kVA Office (Standard 56Ah)</div>
      <div class="value" style="font-size:16px;margin-top:4px;color:var(--crit)">เปลี่ยน: ค่า IR สูงผิดปกติ / สำรองไฟไม่ได้ตามเกณฑ์</div>
      <div class="value" style="font-size:16px;margin-top:2px;color:var(--ok)">ปลอดภัย: ตรวจวัด IR รายไตรมาสปกติ</div>
    </div>
  </div>
  
  <div class="section-title">📋 ตารางรอบเปลี่ยนแบตเตอรี่ราย String
    <button class="btn btn-sm" onclick="openAddModal('battery')">+ เพิ่มข้อมูลแบต</button>
  </div>
  <div class="table-box">
    <table>
      <thead>
        <tr><th>กลุ่มอุปกรณ์ / String</th><th>จำนวนแบตเตอรี่</th><th>Vendor รอบ 1</th><th>เลขอ้างอิง NC (รอบ 1)</th><th>วันที่ติดตั้งรอบ 1</th><th>วันที่ครบรอบ 5 ปี (รอบ 2)</th><th>Vendor รอบ 2</th><th>เลขอ้างอิง NC (รอบ 2)</th><th>จัดการ</th></tr>
      </thead>
      <tbody id="tbl-battery"></tbody>
    </table>
  </div>
</div>

<!-- ================= TAB 3: SPARE PARTS 20 YEARS ================= -->
<div id="t3" class="tab-content">
  <div class="section-title">📅 แผนทดแทนอะไหล่ตามรอบอายุการใช้งาน (20 Years Lifecycle Planning - คลิกที่ชื่อระบบเพื่อดูประวัติ)
    <button class="btn btn-sm" onclick="openAddModal('spare')">+ เพิ่มอะไหล่</button>
  </div>
  <div class="table-box">
    <table>
      <thead>
        <tr><th>ระบบ / รุ่นอุปกรณ์</th><th>รายการอะไหล่</th><th>ตำแหน่ง</th><th>จำนวน</th><th>รอบเปลี่ยน</th><th>ปีรอบหลัก</th><th>จัดการ</th></tr>
      </thead>
      <tbody id="tbl-spare"></tbody>
    </table>
  </div>
</div>

<!-- ================= TAB 4: NORMAL CHANGE (NC) ================= -->
<div id="t4" class="tab-content">
  <div class="section-title">📋 ประวัติใบงานการเปลี่ยนแปลงตามแผนปกติ (Normal Change - NC Records)
    <button class="btn btn-sm" onclick="openAddModal('nc')">+ เพิ่มใบงาน NC</button>
  </div>
  <div class="table-box">
    <table>
      <thead>
        <tr><th>Ticket No.</th><th>Ticket Date</th><th>Type</th><th>Configuration Item</th><th>Subject / รายละเอียด</th><th>Priority</th><th>จัดการ</th></tr>
      </thead>
      <tbody id="tbl-nc"></tbody>
    </table>
  </div>
</div>

<!-- ================= TAB 5: EMERGENCY CHANGE (EC) ================= -->
<div id="t5" class="tab-content">
  <div class="section-title">🚨 ประวัติใบงานการเปลี่ยนแปลงฉุกเฉิน (Emergency Change - EC Records)
    <button class="btn btn-sm" onclick="openAddModal('ec')">+ เพิ่มใบงาน EC</button>
  </div>
  <div class="table-box">
    <table>
      <thead>
        <tr><th>Ticket No.</th><th>Date</th><th>Time</th><th>Configuration Item</th><th>Subject / รายละเอียดเหตุการณ์</th><th>Impact</th><th>จัดการ</th></tr>
      </thead>
      <tbody id="tbl-ec"></tbody>
    </table>
  </div>
</div>

<!-- ================= TAB 6: VENDOR & PM ================= -->
<div id="t6" class="tab-content">
  <div class="section-title">📞 รายชื่อ Vendor และตารางแผนบำรุงรักษาประจำปี 2026</div>
  <div class="table-box">
    <table>
      <thead>
        <tr><th>รายการระบบ / งาน</th><th>สถานที่ (Site)</th><th>Vendor ผู้ดูแล</th><th>ผู้ติดต่อหลัก</th><th>เบอร์โทรศัพท์</th><th>อีเมล / หมายเหตุ</th></tr>
      </thead>
      <tbody>
        <tr><td><b>CRAH &amp; Water Leak</b></td><td>SRB Phase 1-2 (62 Unit)</td><td>Vertiv</td><td>Call Center / ชาตรี / วรชัย</td><td>081-928-2157, 081-928-2156</td><td>callcenter.th@vertivco.com</td></tr>
        <tr><td><b>UPS 400kVA &amp; 30kVA</b></td><td>SRB Phase 1</td><td>Socomec</td><td>Hotline / กนก บัวกอ / อัครวินท์</td><td>086-043-1155, 065-716-1497</td><td>kanok.buako@socomec.com</td></tr>
        <tr><td><b>UPS 60kVA (Office)</b></td><td>SRB Office Shaft</td><td>MAXI / ดีอาร์เค</td><td>ทนงศักดิ์ / อัศนีย์ / ณัฐพงศ์</td><td>094-560-7802, 098-791-2885</td><td>ausanee.s@maxipowerplus.co.th</td></tr>
      </tbody>
    </table>
  </div>
</div>

<!-- ================= TAB 7: PO เปลี่ยนแบต 2025 ================= -->
<div id="t7" class="tab-content">
  <div class="section-title">📦 ติดตามสถานะใบสั่งซื้อ (PO เปลี่ยนแบตเตอรี่ปี 2025)
    <button class="btn btn-sm" onclick="openAddModal('po25')">+ เพิ่ม PO 2025</button>
  </div>
  <div class="table-box">
    <table>
      <thead>
        <tr><th>ลำดับ</th><th>วันที่ออก PO</th><th>เลข PO</th><th>รายการ</th><th>วันจัดส่งสินค้า</th><th>วันที่ความคืบหน้า</th><th>วันติดตั้ง</th><th>หมายเหตุ</th><th>จัดการ</th></tr>
      </thead>
      <tbody id="tbl-po25"></tbody>
    </table>
  </div>
</div>

<!-- ================= TAB 8: PO เปลี่ยนแบต 2026 ================= -->
<div id="t8" class="tab-content">
  <div class="section-title">📦 ติดตามสถานะใบสั่งซื้อ (PO เปลี่ยนแบตเตอรี่ปี 2026)
    <button class="btn btn-sm" onclick="openAddModal('po26')">+ เพิ่ม PO 2026</button>
  </div>
  <div class="table-box">
    <table>
      <thead>
        <tr><th>ลำดับ</th><th>วันที่ออก PO</th><th>เลข PO</th><th>รายการ</th><th>วันจัดส่งสินค้า</th><th>วันที่ความคืบหน้า</th><th>วันติดตั้ง</th><th>หมายเหตุ</th><th>จัดการ</th></tr>
      </thead>
      <tbody id="tbl-po26"></tbody>
    </table>
  </div>
</div>

<!-- Modal สำหรับแก้ไขข้อมูล -->
<div id="editModal" class="modal">
  <div class="modal-content">
    <h3 id="modalTitle">แก้ไขข้อมูล</h3>
    <div id="modalFormFields"></div>
    <div style="display:flex;justify-content:flex-end;gap:8px;margin-top:16px">
      <button class="btn btn-secondary" onclick="closeModal()">ยกเลิก</button>
      <button class="btn" onclick="saveModalData()">บันทึกข้อมูล</button>
    </div>
  </div>
</div>

<!-- Modal สำหรับดูรายละเอียดประวัติการซ่อมของแต่ละระบบ -->
<div id="detailModal" class="modal">
  <div class="modal-content">
    <h3 id="detailTitle">รายละเอียดประวัติการซ่อมระบบ</h3>
    <div id="detailContent"></div>
    <div style="display:flex;justify-content:flex-end;margin-top:16px">
      <button class="btn btn-secondary" onclick="closeDetailModal()">ปิดหน้าต่าง</button>
    </div>
  </div>
</div>

<footer>IDC3 Master Infrastructure Dashboard & Editor — เพิ่มระบบคลิกดูประวัติจากหน้า Spare Parts เรียบร้อย</footer>

<script>
  const defaultData = {
    ups: [
      {loc:"Saraburi Phase 1 (อาคาร UT)", name:"UPS1A", brand:"SOCOMEC", model:"DELPHYS-GP 2.0", sn:"16100385770001", pm:"4 ครั้ง"},
      {loc:"Saraburi Phase 1 (อาคาร UT)", name:"UPS1B", brand:"SOCOMEC", model:"DELPHYS-GP 2.0", sn:"16100385981001", pm:"4 ครั้ง"},
      {loc:"Saraburi Phase 1 (อาคาร UT)", name:"UPS1C", brand:"SOCOMEC", model:"DELPHYS-GP 2.0", sn:"161003859
