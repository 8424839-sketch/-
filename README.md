<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover">
<title>Заказ через WhatsApp</title>
<meta name="color-scheme" content="light dark">
<meta name="theme-color" content="#0b0c0f">
<style>
  :root{ --bg:#0b0c0f; --card:#12141a; --text:#e8eaf0; --muted:#a9b0bd; --line:#1f2330; --brand:#25D366; }
  @media (prefers-color-scheme: light){ :root{ --bg:#f6f7fb; --card:#ffffff; --text:#111827; --muted:#596173; --line:#e6e8ef; } }
  *{box-sizing:border-box}
  body{margin:0; background:var(--bg); color:var(--text); font-family:-apple-system,system-ui,Segoe UI,Roboto,Helvetica,Arial; line-height:1.45}
  .wrap{max-width:760px; margin:0 auto; padding:16px 14px 80px}
  h1{font-size:26px; margin:8px 0 12px}
  .card{background:var(--card); border:1px solid var(--line); border-radius:16px; padding:12px; margin-top:12px}
  .row{display:grid; grid-template-columns:1fr auto; gap:12px; align-items:center; padding:14px 0; border-top:1px solid var(--line)}
  .row:first-child{border-top:none}
  .title{font-weight:800; letter-spacing:0.2px}
  .muted{color:var(--muted); font-size:12px}
  input, select{
    width:100%; padding:12px 14px; border-radius:12px; border:1px solid var(--line);
    background:transparent; color:var(--text); font-size:16px; -webkit-appearance:none; appearance:none;
  }
  input[type="number"]::-webkit-outer-spin-button,input[type="number"]::-webkit-inner-spin-button{-webkit-appearance:none;margin:0}
  .btn{display:inline-flex; align-items:center; justify-content:center; gap:8px; background:var(--brand); color:#0b2b1b; font-weight:800; padding:16px 18px; border-radius:14px; border:none; width:100%; cursor:pointer; font-size:18px}
  .totals{display:flex; flex-wrap:wrap; gap:10px; margin-top:10px}
  .pill{background:var(--card); border:1px solid var(--line); border-radius:999px; padding:10px 12px; font-size:14px}
  .hint{font-size:12px; color:var(--muted); text-align:center; margin-top:10px}
  .bar{position:fixed; left:0; right:0; bottom:0; z-index:10; padding:12px; background:color-mix(in hsl, var(--bg) 70%, transparent); border-top:1px solid var(--line); display:none}
  .warn{background:#7a1b1b; color:#fff; padding:10px 12px; border-radius:10px; margin-top:8px; display:none}
</style>
</head>
<body>
  <div class="wrap">
    <h1>Заказ через WhatsApp</h1>

    <div class="card">
      <div class="row">
        <div>
          <div class="title">Колетчики</div>
          <div class="muted">60 г • 12 шт/короб • 25 ₽/шт</div>
        </div>
        <div style="width:130px"><input type="number" class="boxes" value="0" min="0" step="1" aria-label="Коробов — Колетчики"></div>
      </div>
      <div class="row">
        <div>
          <div class="title">Шоколадные шарики</div>
          <div class="muted">60 г • 12 шт/короб • 25 ₽/шт</div>
        </div>
        <div style="width:130px"><input type="number" class="boxes" value="0" min="0" step="1" aria-label="Коробов — Шоколадные шарики"></div>
      </div>

      <div class="totals">
        <div class="pill">Коробов: <b id="tBoxes">0</b></div>
        <div class="pill">Штучно: <b id="tUnits">0</b> шт</div>
        <div class="pill">Объём: <b id="tVol">0.00000</b> м³</div>
        <div class="pill">Вес: <b id="tWeight">0.0</b> кг</div>
        <div class="pill">Сумма: <b id="tSum">0</b> ₽</div>
      </div>
    </div>

    <div class="card">
      <div style="display:grid; grid-template-columns:1fr 1fr; gap:12px;">
        <input id="clientName" placeholder="Имя">
        <input id="clientPhone" placeholder="Телефон (необязательно)" inputmode="tel">
      </div>
      <div style="display:grid; gap:12px; margin-top:12px;">
        <input id="clientCity" placeholder="Город / адрес">
        <select id="delivery">
          <option value="ПЭК">ПЭК</option>
          <option value="СДЭК">СДЭК</option>
          <option value="Деловые линии">Деловые линии</option>
          <option value="Самовывоз">Самовывоз</option>
        </select>
      </div>
      <div id="warn" class="warn"></div>
    </div>

    <div class="card">
      <button class="btn" id="sendBtn">Отправить в WhatsApp</button>
      <div class="hint">Откроется WhatsApp с сообщением на +7 928 842‑48‑39.</div>
    </div>

    <div class="hint">Получатель: +7 928 842‑48‑39</div>
  </div>

  <div class="bar" id="filesBar">
    <div style="display:flex; gap:8px; align-items:center; justify-content:space-between; flex-wrap:wrap;">
      <div class="muted">Если открыто из «Файлов», нажмите «Поделиться → Открыть в Safari».</div>
      <button class="btn" id="openSafari" style="width:auto; background:#3d4453; color:#eef2ff;">Как открыть в Safari</button>
    </div>
  </div>

<script>
(function(){
  // Настройки
  const PRICE_RUB = 25;
  const UNITS_PER_BOX = 12;
  const BOX_M3 = 0.02204;
  const BOX_KG = 0.70;
  const WHATSAPP_PHONE = '79288424839';

  // Детектор «Файлы»
  function inFilesPreview(){
    try { return location.protocol === 'file:' || location.origin === 'null'; } catch { return true; }
  }
  if (inFilesPreview()){
    const bar = document.getElementById('filesBar');
    bar.style.display = 'block';
    document.getElementById('openSafari').addEventListener('click', ()=>{
      alert('Откройте этот файл так: Поделиться → Открыть в Safari. В Safari отправка в WhatsApp работает стабильно.');
    });
  }

  const $ = (s,p=document)=>p.querySelector(s);
  const $$ = (s,p=document)=>Array.prototype.slice.call(p.querySelectorAll(s));

  const tB=$('#tBoxes'), tU=$('#tUnits'), tS=$('#tSum'), tV=$('#tVol'), tW=$('#tWeight');
  const warn = $('#warn');

  function recalc(){
    let boxes = 0;
    $$('.boxes').forEach(i=>{
      const v = Math.max(0, parseInt(i.value||'0',10) || 0);
      i.value = v;
      boxes += v;
    });
    const units = boxes * UNITS_PER_BOX;
    const sum   = units * PRICE_RUB;
    const vol   = boxes * BOX_M3;
    const kg    = boxes * BOX_KG;
    tB.textContent = boxes;
    tU.textContent = units;
    tS.textContent = sum.toLocaleString('ru-RU');
    tV.textContent = vol.toFixed(5);
    tW.textContent = kg.toFixed(1);
  }

  function buildMessage(){
    const names = ['Колетчики','Шоколадные шарики'];
    const lines = ['Заказ',''];
    let totalBoxes = 0;
    $$('.row').forEach((r,idx)=>{
      const qty = parseInt(r.querySelector('.boxes')?.value||'0',10)||0;
      if(!qty) return;
      totalBoxes += qty;
      lines.push(`• ${names[idx]} — коробов: ${qty} (шт: ${qty*UNITS_PER_BOX})`);
    });
    const units = totalBoxes * UNITS_PER_BOX;
    const sum = units * PRICE_RUB;
    const vol = (totalBoxes * BOX_M3).toFixed(5);
    const kg  = (totalBoxes * BOX_KG).toFixed(1);
    lines.push('', `Итого: коробов ${totalBoxes}, штук ${units}, объём ${vol} м³, вес ~${kg} кг`);
    lines.push(`Сумма к оплате: ${sum} ₽`, '');
    const name = ($('#clientName').value||'').trim();
    const phone= ($('#clientPhone').value||'').trim();
    const city = ($('#clientCity').value||'').trim();
    const del  = ($('#delivery').value||'').trim();
    if (name)  lines.push(`Имя: ${name}`);
    if (phone) lines.push(`Телефон: ${phone}`);
    if (city)  lines.push(`Город/адрес: ${city}`);
    if (del)   lines.push(`Доставка: ${del}`);
    return lines.join('\n');
  }

  function sendWA(){
    warn.style.display='none';
    const boxes = parseInt(tB.textContent||'0',10)||0;
    if (!boxes){
      warn.textContent = 'Добавьте хотя бы 1 короб.';
      warn.style.display='block';
      return;
    }
    const msg = encodeURIComponent(buildMessage());
    const to = WHATSAPP_PHONE.replace(/[^\d]/g,'');
    const url = `https://wa.me/${to}?text=${msg}`;
    try{ location.href = url; }catch{ window.open(url, '_blank'); }
  }

  $$('.boxes').forEach(i=>{
    i.addEventListener('input', recalc, {passive:true});
    i.addEventListener('change', recalc, {passive:true});
  });
  document.getElementById('sendBtn').addEventListener('click', sendWA, {passive:true});
  recalc();
})();
</script>
</body>
</html>
