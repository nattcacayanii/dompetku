# dompetku
aplikasi pengelola keuangan
<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>DompetKu - Atur Uang + Wishlist</title>
<style>
  *{box-sizing:border-box;font-family:'Segoe UI',sans-serif}
  body{margin:0;background:#f2f7f2;padding:16px;padding-bottom:90px}
  h1{font-size:22px;margin:0} p{color:#666;font-size:13px}
  .card{background:#fff;border-radius:16px;padding:16px;margin-top:14px;box-shadow:0 2px 10px rgba(0,0,0,.06)}
  .saldo{font-size:32px;font-weight:800;margin:6px 0}
  .row{display:flex;gap:10px}
  .mini{flex:1;background:#f6f6f6;border-radius:12px;padding:10px;text-align:center}
  .mini b{display:block;font-size:16px}
  .mini span{font-size:12px;color:#666}
  input,select,button{width:100%;padding:12px;border-radius:10px;border:1px solid #ddd;margin-top:8px;font-size:14px}
  button{background:#16a34a;color:#fff;border:none;font-weight:700;cursor:pointer}
  button.secondary{background:#111}
  button.outline{background:#fff;color:#111;border:1px solid #ddd}
  button.small{padding:8px;font-size:12px;width:auto}
  .tabs{display:flex;gap:8px;margin-top:14px;flex-wrap:wrap}
  .tabs button{background:#e8f5e9;color:#111}
  .tabs button.active{background:#16a34a;color:#fff}
  .item{display:flex;justify-content:space-between;padding:10px 0;border-bottom:1px solid #eee;font-size:14px;gap:10px}
  .item small{color:#888;display:block}
  .del{color:red;cursor:pointer;font-size:12px}
  #preview{width:100%;border-radius:12px;margin-top:10px;display:none}
  .bar{height:12px;background:#eee;border-radius:20px;overflow:hidden;margin-top:8px}
  .bar div{height:100%;transition:.3s}
  .badge{display:inline-block;background:#dcfce7;color:#166534;padding:4px 10px;border-radius:20px;font-size:12px;font-weight:700}
  .wish{border:1px solid #e5e7eb;border-radius:14px;padding:12px;margin-top:10px;background:#fffdf5}
  .wish img{width:100%;height:140px;object-fit:cover;border-radius:10px;margin-top:8px;display:none}
  .wish-top{display:flex;justify-content:space-between;align-items:center}
</style>
</head>
<body>

<h1>💰 DompetKu</h1>
<p>Atur uang, foto struk, & nabung buat wishlist</p>

<div class="card">
  <span class="badge" id="tanggalHari"></span>
  <div>Saldo Saat Ini</div>
  <div class="saldo" id="saldo">Rp 0</div>
  <div class="row">
    <div class="mini"><span>🟢 Pemasukan</span><b id="totalMasuk">Rp 0</b></div>
    <div class="mini"><span>🔴 Pengeluaran</span><b id="totalKeluar">Rp 0</b></div>
    <div class="mini"><span>🎯 Total Nabung Wishlist</span><b id="totalWish">Rp 0</b></div>
  </div>
  <div style="margin-top:12px">
    <b>🎯 Ideal Nabung (50/30/20)</b>
    <div id="idealBox"></div>
    <div class="bar"><div id="barNabung" style="width:0%;background:#16a34a"></div></div>
    <small id="barText"></small>
  </div>
</div>

<div class="tabs">
  <button class="active" onclick="gantiTab('masuk',this)">+ Pemasukan</button>
  <button onclick="gantiTab('keluar',this)">- Pengeluaran</button>
  <button onclick="gantiTab('struk',this)">🧾 Foto Struk</button>
  <button onclick="gantiTab('wishlist',this)">🎁 Wishlist</button>
</div>

<div class="card" id="formCard">
  <h3 id="formJudul">Tambah Pemasukan</h3>
  <div id="formTransaksi">
    <input id="nama" placeholder="Contoh: Uang jajan / Gaji">
    <input id="nominal" type="number" placeholder="Nominal Rp - contoh: 50000">
    <select id="kategori">
      <option value="Kebutuhan">Kebutuhan (50%)</option>
      <option value="Keinginan">Keinginan / Jajan (30%)</option>
      <option value="Tabungan">Tabungan (20%)</option>
    </select>
    <div id="strukArea" style="display:none">
      <input type="file" id="fotoStruk" accept="image/*" capture="environment">
      <img id="preview">
      <p id="statusOCR">📸 Pilih foto struk, nanti nominalnya otomatis ke-detect.</p>
      <button class="outline" onclick="bacaStruk()">🔍 Baca Struk Otomatis</button>
    </div>
    <button onclick="simpan()">Simpan</button>
  </div>

  <div id="formWishlist" style="display:none">
    <input id="wishNama" placeholder="Mau nabung buat apa? Contoh: Sepatu, HP, Skincare">
    <input id="wishTarget" type="number" placeholder="Target harga Rp - contoh: 300000">
    <input id="wishFoto" type="file" accept="image/*">
    <input id="wishLink" placeholder="Link toko / Shopee (opsional)">
    <button onclick="tambahWishlist()">+ Tambah Wishlist</button>
    <p>Tips: Tiap dapet pemasukan, sisihin 20% langsung masuk ke wishlist ini!</p>
  </div>
</div>

<div class="card">
  <h3>🎁 Wishlist & Target Nabung Kamu</h3>
  <div id="listWish"></div>
</div>

<div class="card">
  <h3>Riwayat Transaksi</h3>
  <div id="list"></div>
  <button class="secondary" onclick="hapusSemua()">Hapus Semua Data</button>
</div>

<script src="https://cdn.jsdelivr.net/npm/tesseract.js@5/dist/tesseract.min.js"></script>
<script>
let data = JSON.parse(localStorage.getItem('dompetku')||'[]');
let wishlist = JSON.parse(localStorage.getItem('dompetku_wish')||'[]');
let mode = 'masuk';
document.getElementById('tanggalHari').innerText = new Date().toLocaleDateString('id-ID',{weekday:'long',day:'numeric',month:'long',year:'numeric'});

function gantiTab(m,btn){
  mode=m;
  document.querySelectorAll('.tabs button').forEach(b=>b.classList.remove('active'));
  btn.classList.add('active');
  document.getElementById('formTransaksi').style.display = m==='wishlist' ? 'none':'block';
  document.getElementById('formWishlist').style.display = m==='wishlist' ? 'block':'none';
  document.getElementById('strukArea').style.display = m==='struk' ? 'block':'none';
  document.getElementById('formJudul').innerText = m==='masuk' ? 'Tambah Pemasukan' : m==='keluar' ? 'Tambah Pengeluaran Manual' : m==='struk' ? 'Tambah Pengeluaran dari Struk' : 'Buat Target Nabung Baru';
}

document.getElementById('fotoStruk').addEventListener('change', e=>{
  const file=e.target.files[0]; if(!file) return;
  const url=URL.createObjectURL(file);
  const img=document.getElementById('preview');
  img.src=url; img.style.display='block';
  document.getElementById('nama').value='Belanja dari Struk';
});

async function bacaStruk(){
  const img=document.getElementById('preview');
  if(!img.src){ alert('Pilih foto struk dulu!'); return; }
  document.getElementById('statusOCR').innerText='⏳ Lagi baca struk...';
  try{
    const {data:{text}} = await Tesseract.recognize(img.src,'eng+ind');
    let angka = text.match(/[\d.,]+/g) || [];
    let bersih = angka.map(a=>parseInt(a.replace(/[.,]/g,''))).filter(n=>n>1000 && n<10000000);
    if(bersih.length){
      let total = Math.max(...bersih);
      document.getElementById('nominal').value = total;
      document.getElementById('statusOCR').innerText = '✅ Ketemu! Total: Rp '+total.toLocaleString('id-ID')+'. Bisa diedit manual kalau salah.';
    } else document.getElementById('statusOCR').innerText='❌ Nggak ketemu angka. Isi manual aja ya.';
  }catch(e){ document.getElementById('statusOCR').innerText='Gagal baca otomatis. Isi manual aja.'; }
}

function simpan(){
  let nama=document.getElementById('nama').value || 'Tanpa Nama';
  let nominal=parseInt(document.getElementById('nominal').value);
  let kategori=document.getElementById('kategori').value;
  if(!nominal){ alert('Isi nominal dulu!'); return; }
  let tipe = mode==='masuk' ? 'masuk' : 'keluar';
  data.unshift({nama, nominal, kategori, tipe, tanggal:new Date().toLocaleDateString('id-ID')});
  localStorage.setItem('dompetku',JSON.stringify(data));
  document.getElementById('nama').value=''; document.getElementById('nominal').value='';
  render();
}

function tambahWishlist(){
  let nama=document.getElementById('wishNama').value;
  let target=parseInt(document.getElementById('wishTarget').value);
  let link=document.getElementById('wishLink').value;
  if(!nama || !target){ alert('Isi nama wishlist & target harganya!'); return; }
  let file=document.getElementById('wishFoto').files[0];
  if(file){
    let reader=new FileReader();
    reader.onload=e=>{
      wishlist.unshift({nama,target,terkumpul:0,link,foto:e.target.result,tanggal:new Date().toLocaleDateString('id-ID')});
      saveWish(); resetWishForm();
    };
    reader.readAsDataURL(file);
  } else {
    wishlist.unshift({nama,target,terkumpul:0,link,foto:'',tanggal:new Date().toLocaleDateString('id-ID')});
    saveWish(); resetWishForm();
  }
}
function resetWishForm(){ document.getElementById('wishNama').value=''; document.getElementById('wishTarget').value=''; document.getElementById('wishLink').value=''; render(); }
function saveWish(){ localStorage.setItem('dompetku_wish',JSON.stringify(wishlist)); render(); }

function tambahTabungan(i){
  let jml=parseInt(prompt('Mau nabung berapa buat "'+wishlist[i].nama+'"? Contoh: 20000'));
  if(!jml || jml<=0) return;
  wishlist[i].terkumpul += jml;
  // Otomatis catat juga sebagai pengeluaran tabungan biar saldo kepotong
  data.unshift({nama:'Nabung: '+wishlist[i].nama, nominal:jml, kategori:'Tabungan', tipe:'keluar', tanggal:new Date().toLocaleDateString('id-ID')});
  localStorage.setItem('dompetku',JSON.stringify(data));
  saveWish();
  if(wishlist[i].terkumpul>=wishlist[i].target) alert('🎉 YEAY! Target "'+wishlist[i].nama+'" udah tercapai! Bisa dibeli sekarang!');
}
function hapusWish(i){ if(confirm('Hapus wishlist ini?')){ wishlist.splice(i,1); saveWish(); } }
function hapus(i){ data.splice(i,1); localStorage.setItem('dompetku',JSON.stringify(data)); render(); }
function hapusSemua(){ if(confirm('Yakin hapus semua?')){ data=[]; wishlist=[]; localStorage.removeItem('dompetku'); localStorage.removeItem('dompetku_wish'); render(); } }
function format(n){ return 'Rp '+n.toLocaleString('id-ID'); }

function render(){
  let masuk = data.filter(d=>d.tipe==='masuk').reduce((a,b)=>a+b.nominal,0);
  let keluar = data.filter(d=>d.tipe==='keluar').reduce((a,b)=>a+b.nominal,0);
  let saldo = masuk - keluar;
  let totalWishTerkumpul = wishlist.reduce((a,b)=>a+b.terkumpul,0);
  document.getElementById('saldo').innerText = format(saldo);
  document.getElementById('totalMasuk').innerText = format(masuk);
  document.getElementById('totalKeluar').innerText = format(keluar);
  document.getElementById('totalWish').innerText = format(totalWishTerkumpul);

  let idealKebutuhan = masuk*0.5, idealKeinginan = masuk*0.3, idealTabung = masuk*0.2;
  let persenTabung = masuk>0 ? Math.round((saldo/masuk)*100) : 0;
  document.getElementById('idealBox').innerHTML = `
    <div class="item"><span>🍚 Kebutuhan (50%)</span><b>${format(idealKebutuhan)}</b></div>
    <div class="item"><span>🍟 Keinginan (30%)</span><b>${format(idealKeinginan)}</b></div>
    <div class="item"><span>💰 Target Nabung (20%)</span><b style="color:#16a34a">${format(idealTabung)}</b></div>
    <p style="margin:8px 0 0">Saran: Dari <b>${format(masuk)}</b>, langsung pisahin <b>${format(idealTabung)}</b> ke wishlist!</p>`;
  document.getElementById('barNabung').style.width = Math.min(100,Math.max(0,persenTabung))+'%';
  document.getElementById('barText').innerText = `Kamu nyimpen sekitar ${persenTabung}% dari pemasukan. Idealnya 20%. ${persenTabung>=20?'Mantap, udah aman! 🎉':'Semangat, kurang dikit lagi!'}`;

  document.getElementById('listWish').innerHTML = wishlist.length ? wishlist.map((w,i)=>{
    let persen = Math.min(100, Math.round((w.terkumpul/w.target)*100));
    let sisa = w.target - w.terkumpul;
    let sudah = w.terkumpul>=w.target;
    return `<div class="wish">
      <div class="wish-top"><b>🎁 ${w.nama}</b><span class="del" onclick="hapusWish(${i})">hapus</span></div>
      ${w.foto?`<img src="${w.foto}" style="display:block">`:''}
      <small>${w.tanggal} ${w.link?`• <a href="${w.link}" target="_blank">Lihat di toko</a>`:''}</small>
      <div style="margin-top:8px;display:flex;justify-content:space-between"><span>Terkumpul: <b>${format(w.terkumpul)}</b></span><span>Target: <b>${format(w.target)}</b></span></div>
      <div class="bar"><div style="width:${persen}%;background:${sudah?'#16a34a':'#f59e0b'}"></div></div>
      <div style="margin-top:6px;font-size:13px">${sudah?'🎉 <b>UDAH KETCAPAI! Bisa dibeli!</b>':`Kurang <b>${format(sisa)}</b> lagi • ${persen}% • Saran nabung per minggu: <b>${format(Math.ceil(sisa/4))}</b>`}</div>
      ${!sudah?`<div class="row"><button class="small" onclick="tambahTabungan(${i})">+ Nabung Ke Sini</button><button class="small outline" onclick="tambahTabungan(${i})">Nabung 20% (${format(Math.round(masuk*0.2))})</button></div>`:''}
    </div>`;
  }).join('') : '<p>Belum ada wishlist. Yuk bikin target pertama kamu!</p>';

  document.getElementById('list').innerHTML = data.length ? data.map((d,i)=>`
    <div class="item"><div><b>${d.tipe==='masuk'?'🟢':'🔴'} ${d.nama}</b><small>${d.tanggal} • ${d.kategori}</small></div>
    <div style="text-align:right"><b>${d.tipe==='masuk'?'+':'-'} ${format(d.nominal)}</b><br><span class="del" onclick="hapus(${i})">hapus</span></div></div>`).join('') : '<p>Belum ada transaksi.</p>';
}
render();
</script>
</body>
</html>
