from pathlib import Path
import zipfile

# Prepare directory
out_dir = Path("/mnt/data/bozkurtfk_site")
out_dir.mkdir(parents=True, exist_ok=True)

index_html = """<!doctype html>
<html lang="tr">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Bozkurt FK - Kayıt</title>
  <style>
    :root{--blue:#2a6fb3;--grey:#6b6f76;--bg:#f4f6f8}
    body{font-family:Inter,ui-sans-serif,system-ui,Segoe UI,Roboto,'Helvetica Neue',Arial; background:var(--bg); margin:0; color:#222}
    header{background:linear-gradient(90deg,var(--grey),var(--blue)); color:white; padding:24px 16px; display:flex; align-items:center; gap:12px}
    .logo{width:64px;height:64px;border-radius:12px;background:rgba(255,255,255,0.12);display:flex;align-items:center;justify-content:center;font-weight:700}
    main{max-width:900px;margin:28px auto;padding:16px}
    .card{background:white;border-radius:12px;box-shadow:0 6px 18px rgba(31,41,55,0.06);padding:20px;margin-bottom:16px}
    h1{margin:0 0 8px 0}
    form .row{display:flex;gap:12px;margin-bottom:10px}
    label{font-size:13px;color:#444;margin-bottom:6px;display:block}
    input,select,textarea{width:100%;padding:10px;border-radius:8px;border:1px solid #e6e9ee;font-size:14px}
    button{background:var(--blue);color:white;padding:10px 14px;border-radius:8px;border:none;font-weight:600;cursor:pointer}
    .muted{color:#666;font-size:13px}
    footer{max-width:900px;margin:16px auto;text-align:center;color:#666;font-size:13px}
    .note{background:#fff7e6;padding:10px;border-radius:8px;color:#7a4b00;border:1px solid #ffecb3;margin-bottom:12px}
  </style>
</head>
<body>
  <header>
    <div class="logo">BK</div>
    <div>
      <div style="font-weight:700">Bozkurt FK</div>
      <div style="font-size:13px;opacity:0.9">Resmi Kayıt Sayfası</div>
    </div>
  </header>

  <main>
    <div class="card">
      <h1>Oyuncu / Üye Kayıt Formu</h1>
      <p class="muted">Takıma katılmak için formu doldurun. Bilgiler tarayıcınızda saklanacak ve yöneticiler <strong>admin sayfasından</strong> görüntüleyip CSV olarak indirebilir.</p>

      <div class="note">Not: Bu temel sürüm verileri sunucuda saklamaz. İsterseniz ben bu formu bir Google Form veya sunucu tarafı (PHP/MySQL) ile entegre edilmiş hâle getiririm.</div>

      <form id="regForm">
        <div class="row">
          <div style="flex:1">
            <label>İsim</label>
            <input type="text" id="isim" required>
          </div>
          <div style="flex:1">
            <label>Soyisim</label>
            <input type="text" id="soyisim" required>
          </div>
        </div>
        <div class="row">
          <div style="flex:1">
            <label>Telefon</label>
            <input type="tel" id="telefon" placeholder="+90 5xx xxx xx xx" required>
          </div>
          <div style="flex:1">
            <label>Doğum Yılı</label>
            <input type="number" id="dyil" min="1900" max="2025" placeholder="2000">
          </div>
        </div>
        <div class="row">
          <div style="flex:1">
            <label>Pozisyon</label>
            <select id="pozisyon">
              <option>Kaleci</option>
              <option>Defans</option>
              <option>Orta Saha</option>
              <option>Forvet</option>
              <option>Yedek</option>
            </select>
          </div>
          <div style="flex:1">
            <label>Forma Numarası (isteğe bağlı)</label>
            <input type="number" id="forma" min="1" max="99">
          </div>
        </div>
        <div style="margin-bottom:12px">
          <label>Kısa Bilgi / Kendini Tanıt</label>
          <textarea id="bio" rows="4" placeholder="Kısa bir açıklama..."></textarea>
        </div>

        <div style="display:flex;gap:8px;align-items:center">
          <button type="submit">Kayıt Ol</button>
          <a href="admin.html" style="margin-left:8px;align-self:center">Yönetici Sayfası</a>
        </div>
      </form>
      <div id="msg" style="margin-top:12px"></div>
    </div>

    <div class="card">
      <h2>Takım Hakkında</h2>
      <p>Bozkurt FK — gri &amp; mavi renkleriyle mücadele eden amatör futbol takımı. Formunuzu doldurduktan sonra yöneticiler sizi arayabilir.</p>
    </div>
  </main>

  <footer>© Bozkurt FK — Hazır site · Lokal kayıt sistemi</footer>

<script>
const form = document.getElementById('regForm');
const msg = document.getElementById('msg');

function loadRegs(){ return JSON.parse(localStorage.getItem('bozkurt_regs')||'[]'); }
function saveRegs(arr){ localStorage.setItem('bozkurt_regs', JSON.stringify(arr)); }

form.addEventListener('submit', e=>{
  e.preventDefault();
  const reg = {
    isim: document.getElementById('isim').value.trim(),
    soyisim: document.getElementById('soyisim').value.trim(),
    telefon: document.getElementById('telefon').value.trim(),
    dyil: document.getElementById('dyil').value.trim(),
    pozisyon: document.getElementById('pozisyon').value,
    forma: document.getElementById('forma').value,
    bio: document.getElementById('bio').value.trim(),
    timestamp: new Date().toISOString()
  };
  const arr = loadRegs();
  arr.push(reg);
  saveRegs(arr);
  msg.innerHTML = '<strong style="color:green">Kayıt başarıyla alındı!</strong>';
  form.reset();
  setTimeout(()=>msg.innerHTML='',3000);
});
</script>
</body>
</html>
"""

admin_html = """<!doctype html>
<html lang="tr">
<head>
  <meta charset="utf-8"/>
  <meta name="viewport" content="width=device-width,initial-scale=1"/>
  <title>Bozkurt FK - Admin</title>
  <style>
    body{font-family:Inter,system-ui,Arial;background:#f4f6f8;padding:20px;color:#222}
    .card{background:white;padding:18px;border-radius:10px;max-width:1000px;margin:0 auto;box-shadow:0 6px 18px rgba(0,0,0,0.06)}
    table{width:100%;border-collapse:collapse;font-size:14px}
    th,td{padding:8px;border-bottom:1px solid #eee;text-align:left}
    th{background:#fafafa}
    button{padding:8px 12px;border-radius:8px;border:none;background:#2a6fb3;color:#fff;cursor:pointer}
    .small{font-size:13px;color:#666;margin-bottom:8px}
  </style>
</head>
<body>
  <div class="card">
    <h1>Yönetici - Kayıtlar</h1>
    <p class="small">Bu sayfa tarayıcınızın <code>localStorage</code> içindeki kayıtları gösterir. Dışa aktarmak için "CSV indir" butonunu kullanın.</p>
    <div style="display:flex;gap:8px;margin-bottom:12px">
      <button id="refresh">Yenile</button>
      <button id="clear">Bütün Kayıtları Sil</button>
      <button id="download">CSV İndir</button>
    </div>
    <div id="tableWrap"></div>
  </div>

<script>
function loadRegs(){ return JSON.parse(localStorage.getItem('bozkurt_regs')||'[]'); }
function render(){
  const regs = loadRegs();
  if(!regs.length){
    document.getElementById('tableWrap').innerHTML = '<div style="padding:10px;color:#666">Kayıt bulunamadı.</div>'; return;
  }
  let html = '<table><thead><tr><th>#</th><th>İsim</th><th>Soyisim</th><th>Telefon</th><th>Doğum Yılı</th><th>Pozisyon</th><th>Forma</th><th>Not</th><th>Zaman</th></tr></thead><tbody>';
  regs.forEach((r,i)=>{
    html += `<tr><td>${i+1}</td><td>${escapeHtml(r.isim)}</td><td>${escapeHtml(r.soyisim)}</td><td>${escapeHtml(r.telefon)}</td><td>${escapeHtml(r.dyil)}</td><td>${escapeHtml(r.pozisyon)}</td><td>${escapeHtml(r.forma)}</td><td>${escapeHtml(r.bio)}</td><td>${new Date(r.timestamp).toLocaleString()}</td></tr>`;
  });
  html += '</tbody></table>';
  document.getElementById('tableWrap').innerHTML = html;
}

function escapeHtml(s){ return (s||'').replace(/[&<>"']/g, c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',\"'\":'&#39;'})[c]); }

document.getElementById('refresh').addEventListener('click', render);
document.getElementById('clear').addEventListener('click', ()=>{ if(confirm('Tüm kayıtlar silinsin mi?')){ localStorage.removeItem('bozkurt_regs'); render(); }});
document.getElementById('download').addEventListener('click', ()=>{
  const regs = loadRegs();
  if(!regs.length){ alert('Kayıt yok'); return; }
  const keys = ['isim','soyisim','telefon','dyil','pozisyon','forma','bio','timestamp'];
  const header = keys.join(',') + '\\n';
  const csv = regs.map(r=> keys.map(k=> '\"'+ ( (r[k]||'').toString().replace(/\"/g,'\"\"') ) +'\"').join(',')).join('\\n');
  const blob = new Blob([header+csv], {type:'text/csv;charset=utf-8;'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a'); a.href = url; a.download = 'bozkurt_regs.csv'; document.body.appendChild(a); a.click(); a.remove(); URL.revokeObjectURL(url);
});

// initial render
render();
</script>
</body>
</html>
"""

# Write files
(index := out_dir / "index.html").write_text(index_html, encoding="utf-8")
(admin := out_dir / "admin.html").write_text(admin_html, encoding="utf-8")

# Create README
readme = out_dir / "README.txt"
readme.write_text("Bozkurt FK - Basit kayıtlı web sitesi\\n\\nKullanım:\\n- index.html ana kayıt formu.\\n- admin.html ile kayıtları görüntüleyip CSV indirebilirsiniz.\\n\\nHosting önerileri:\\n- GitHub Pages\\n- Netlify\\n- Vercel\\n\\nNot: Bu sürüm yerel storage (tarayıcı) kullanır. Sunucu ile saklama isterseniz söyle, PHP/MySQL veya Google Forms entegrasyonu yaparım.\\n", encoding="utf-8")

# Zip files
zip_path = Path("/mnt/data/bozkurtfk_site.zip")
with zipfile.ZipFile(zip_path, "w", zipfile.ZIP_DEFLATED) as zf:
    for f in out_dir.iterdir():
        zf.write(f, arcname=f.name)

