
<script setup>
import { ref, computed, watch, onMounted } from 'vue'

// Data tugas
const tugas = ref([])
const tugasBaru = ref('')
const kategori = ref('Sekolah')
const prioritas = ref('Sedang')
const tenggat = ref('')
const cari = ref('')
const filter = ref('Semua')
const detail = ref(false)
const prioritasSaja = ref(false)

// Ambil tugas tersimpan
onMounted(() => {
  try {
    tugas.value = JSON.parse(localStorage.getItem('taskify') || '[]')
  } catch {
    tugas.value = []
  }
})

// Simpan tugas otomatis
watch(tugas, data => {
  localStorage.setItem('taskify', JSON.stringify(data))
}, { deep: true })

// Hitung progres
const selesai = computed(() => tugas.value.filter(t => t.selesai).length)
const sisa = computed(() => tugas.value.length - selesai.value)
const progres = computed(() => tugas.value.length
  ? Math.round(selesai.value / tugas.value.length * 100) : 0)

// Filter dan pencarian
const daftar = computed(() => tugas.value.filter(t =>
  t.nama.toLowerCase().includes(cari.value.toLowerCase()) &&
  (filter.value === 'Semua' ||
    (filter.value === 'Selesai' && t.selesai) ||
    (filter.value === 'Belum Selesai' && !t.selesai)) &&
  (!prioritasSaja.value || t.prioritas === 'Tinggi')
))

// Tambah tugas
function tambahTugas() {
  if (!tugasBaru.value.trim()) return

  tugas.value.unshift({
    id: Date.now(),
    nama: tugasBaru.value.trim(),
    kategori: kategori.value,
    prioritas: prioritas.value,
    tenggat: tenggat.value,
    selesai: false,
    catatan: '',
    bukaCatatan: false
  })

  tugasBaru.value = ''
  tenggat.value = ''
  detail.value = false
}

// Hapus tugas
function hapusTugas(id) {
  if (confirm('Hapus tugas ini?')) {
    tugas.value = tugas.value.filter(t => t.id !== id)
  }
}

// Format tanggal
function formatTanggal(tanggal) {
  return tanggal ? new Date(tanggal + 'T00:00:00')
    .toLocaleDateString('id-ID', {
      day: 'numeric', month: 'short', year: 'numeric'
    }) : ''
}
</script>

<template>
  <main class="halaman">
    <div class="aplikasi">

      <!-- Sidebar -->
      <aside class="sidebar">
        <div class="merek">
          <span>❧</span>
          <h1>Taskify</h1>
          <small>Small steps, big progress</small>
        </div>

        <nav>
          <button @click="filter = 'Semua'">⌂ Beranda</button>
          <button @click="filter = 'Belum Selesai'">☷ Belum Selesai</button>
          <button @click="filter = 'Selesai'">✓ Tugas Selesai</button>
        </nav>

        <div class="pesan">
          <span>✿</span>
          <p>Pelan-pelan saja.<br>Yang penting terus maju.</p>
        </div>

        <div class="profil">
          <span class="avatar">E</span>
          <div><b>Taskify User</b><small>Ruang produktifmu</small></div>
        </div>
      </aside>

      <!-- Konten utama -->
      <section class="konten">
        <header class="sambutan">
          <div>
            <small class="label">RUANG PRODUKTIVITAS</small>
            <h2>Hai, semangat hari ini ✿</h2>
            <p>Selesaikan satu per satu. Kamu pasti bisa.</p>
          </div>
          <span class="tanggal">{{ new Date().toLocaleDateString('id-ID', {
            day: 'numeric', month: 'short', year: 'numeric'
          }) }}</span>
        </header>

        <!-- Ringkasan -->
        <section class="ringkasan">
          <article><span>☷</span><div><small>Total tugas</small><strong>{{ tugas.length }}</strong></div></article>
          <article><span>✓</span><div><small>Selesai</small><strong>{{ selesai }}</strong></div></article>
          <article><span>◷</span><div><small>Belum selesai</small><strong>{{ sisa }}</strong></div></article>
        </section>

        <!-- Progres -->
        <section class="kartu">
          <div class="judul">
            <div><small class="label">LANGKAH KECILMU</small><h2>Progress tugas</h2></div>
            <strong class="angka">{{ progres }}%</strong>
          </div>
          <div class="jalur"><div :style="{ width: progres + '%' }"></div></div>
          <div class="keterangan"><span>{{ selesai }} selesai</span><span>{{ sisa }} tersisa</span></div>
        </section>

        <!-- Daftar tugas -->
        <section class="kartu">
          <div class="judul">
            <div><small class="label">DAFTAR HARIAN</small><h2>Daftar tugas</h2></div>
            <button class="ringan" :disabled="!selesai" @click="tugas = tugas.filter(t => !t.selesai)">Bersihkan selesai</button>
          </div>

          <form class="form" @submit.prevent="tambahTugas">
            <div class="input-baris">
              <span>＋</span>
              <input v-model="tugasBaru" placeholder="Apa yang ingin kamu selesaikan?" maxlength="140">
              <button class="tambah">Tambah ↗</button>
            </div>

            <button type="button" class="detail" @click="detail = !detail">
              {{ detail ? '− Tutup detail' : '+ Tambah detail tugas' }}
            </button>

            <div v-if="detail" class="detail-form">
              <label>Kategori
                <select v-model="kategori">
                  <option>Sekolah</option><option>Pekerjaan</option>
                  <option>Pribadi</option><option>Umum</option>
                </select>
              </label>
              <label>Prioritas
                <select v-model="prioritas">
                  <option>Rendah</option><option>Sedang</option><option>Tinggi</option>
                </select>
              </label>
              <label>Tenggat
                <input v-model="tenggat" type="date">
              </label>
            </div>
          </form>

          <!-- Pencarian dan filter -->
          <div class="alat">
            <div class="filter">
              <button v-for="f in ['Semua', 'Belum Selesai', 'Selesai']"
                :key="f" :class="{ aktif: filter === f }" @click="filter = f">{{ f }}</button>
            </div>
            <input v-model="cari" placeholder="⌕ Cari tugas...">
          </div>

          <label class="opsi">
            <input type="checkbox" v-model="prioritasSaja"> Prioritas tinggi saja
          </label>

          <div v-if="daftar.length" class="daftar">
            <article v-for="t in daftar" :key="t.id" class="item" :class="{ sudah: t.selesai }">
              <input type="checkbox" v-model="t.selesai">

              <div class="isi">
                <h3>{{ t.nama }}</h3>
                <div class="tag-baris">
                  <span class="tag">{{ t.kategori }}</span>
                  <span class="tag" :class="'p-' + t.prioritas.toLowerCase()">{{ t.prioritas }}</span>
                  <span v-if="t.tenggat" class="tag">{{ formatTanggal(t.tenggat) }}</span>
                </div>

                <button class="detail" @click="t.bukaCatatan = !t.bukaCatatan">
                  {{ t.bukaCatatan ? 'Tutup catatan' : '+ Catatan' }}
                </button>
                <textarea v-if="t.bukaCatatan" v-model="t.catatan" placeholder="Tulis catatan..." rows="2"></textarea>
              </div>

              <button class="hapus" @click="hapusTugas(t.id)">×</button>
            </article>
          </div>

          <div v-else class="kosong">
            <span>❀</span>
            <h3>Belum ada tugas di sini</h3>
            <p>Tambahkan tugas dan mulai langkah kecilmu.</p>
          </div>
        </section>

        <footer class="kutipan">❧ <span>“Sedikit demi sedikit, lama-lama menjadi bukit.”</span> ✧</footer>
      </section>
    </div>
  </main>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;600;700&family=Playfair+Display:ital,wght@0,500;0,600;1,500&display=swap');

* { box-sizing: border-box; }
body { margin: 0; font-family: 'DM Sans', sans-serif; color: #2d3536; }
button, input, select, textarea { font: inherit; }
button { cursor: pointer; }

.halaman {
  min-height: 100vh; padding: 28px;
  background: radial-gradient(ellipse at 8% 8%, #dce4d8, transparent 30%),
  radial-gradient(ellipse at 90% 85%, #d5e1e3, transparent 28%),
  linear-gradient(135deg, #f7f5ef, #e9eee8 55%, #f5f1e8);
}
.aplikasi { max-width: 1500px; margin: auto; display: grid; grid-template-columns: 215px minmax(0, 1fr); gap: 22px; }
.sidebar { min-height: calc(100vh - 56px); padding: 26px 17px 18px; border-radius: 26px; color: #f8f7f0; background: linear-gradient(160deg, #697c70, #52675e 75%, #394943); display: flex; flex-direction: column; box-shadow: 0 16px 40px #31413912; }
.merek { padding: 3px 10px 30px; }
.merek span { font-size: 28px; color: #dce8d9; }
.merek h1 { margin: 0; font: 500 34px 'Playfair Display', serif; }
small { font-size: 10px; }
.merek small { color: #d8e0d9; }
nav { display: grid; gap: 7px; }
nav button { padding: 12px; border: 0; border-radius: 13px; background: none; color: #e5ebe5; text-align: left; font-size: 12px; }
nav button:hover { background: #ffffff30; color: white; }
.pesan { margin-top: auto; padding: 22px 10px; border-top: 1px solid #ffffff38; color: #e5ece5; }
.pesan span { font-size: 28px; }
.pesan p { font: 16px/1.5 'Playfair Display', serif; }
.profil { display: flex; align-items: center; gap: 10px; padding-top: 16px; border-top: 1px solid #ffffff38; }
.avatar { display: grid; place-items: center; width: 35px; height: 35px; border-radius: 50%; background: #d9e4d8; color: #52675e; font: 18px 'Playfair Display', serif; }
.profil b, .profil small { display: block; }
.profil b { font-size: 11px; }
.profil small { margin-top: 3px; color: #d1dbd2; font-size: 9px; }

.konten { min-width: 0; display: flex; flex-direction: column; gap: 17px; }
.sambutan, .ringkasan article, .kartu, .kutipan { border: 1px solid #ffffffd9; box-shadow: 0 16px 40px #31413912; }
.sambutan { display: flex; justify-content: space-between; align-items: center; gap: 15px; padding: 25px 27px; border-radius: 24px; background: linear-gradient(110deg, #ffffffe0, #e8efe5e0); }
.label { color: #81968a; font-size: 9px; font-weight: 700; letter-spacing: 1.7px; }
.sambutan h2 { margin: 8px 0; font: 500 clamp(23px, 2.3vw, 31px) 'Playfair Display', serif; }
.sambutan p { margin: 0; color: #788681; font-size: 12px; }
.tanggal { white-space: nowrap; padding: 10px 13px; border: 1px solid #e8ece6; border-radius: 30px; background: #ffffffbb; color: #52675e; font-size: 10px; }

.ringkasan { display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; }
.ringkasan article { display: flex; align-items: center; gap: 12px; padding: 16px 14px; border-radius: 19px; background: #ffffffd9; }
.ringkasan article > span { display: grid; place-items: center; flex: 0 0 36px; height: 36px; border-radius: 13px; background: #e4eee6; color: #587967; font-size: 18px; }
.ringkasan small, .ringkasan strong { display: block; }
.ringkasan small { color: #7b8982; }
.ringkasan strong { margin-top: 4px; font: 600 24px 'Playfair Display', serif; }

.kartu { padding: 21px; border-radius: 22px; background: #ffffffd9; }
.judul { display: flex; align-items: center; justify-content: space-between; gap: 10px; }
.judul h2 { margin: 5px 0 0; font: 500 21px 'Playfair Display', serif; }
.angka { color: #697c70; font: 25px 'Playfair Display', serif; }
.jalur { height: 8px; margin-top: 15px; border-radius: 20px; background: #e8ece5; overflow: hidden; }
.jalur div { height: 100%; border-radius: 20px; background: linear-gradient(90deg, #697c70, #98aa9d); transition: width .3s; }
.keterangan { display: flex; justify-content: space-between; margin-top: 8px; color: #8a9690; font-size: 10px; }
.ringan { padding: 8px 10px; border: 0; border-radius: 9px; background: #f2f5ef; color: #6a8174; font-size: 10px; }
.ringan:disabled { opacity: .4; cursor: not-allowed; }

.form { margin-top: 17px; padding: 10px; border: 1px solid #e7ebe5; border-radius: 17px; background: #fbfcf9; }
.input-baris { display: flex; align-items: center; gap: 10px; }
.input-baris > span { display: grid; place-items: center; width: 33px; height: 33px; border-radius: 50%; background: #edf2ec; color: #647c6e; font-size: 22px; }
.input-baris input { flex: 1; min-width: 0; padding: 11px 0; border: 0; outline: 0; background: none; font-size: 12px; }
.tambah { padding: 11px 15px; border: 0; border-radius: 12px; background: #697c70; color: white; font-size: 11px; }
.detail { margin-top: 8px; padding: 4px 0; border: 0; background: none; color: #83938a; font-size: 10px; }
.detail-form { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; padding-top: 12px; }
.detail-form label { display: grid; gap: 6px; color: #7e8d84; font-size: 10px; }
.detail-form select, .detail-form input { width: 100%; min-width: 0; padding: 9px; border: 1px solid #e1e8df; border-radius: 9px; background: white; color: #46574f; font-size: 10px; }

.alat { display: flex; justify-content: space-between; align-items: center; gap: 12px; margin: 21px 0 12px; }
.filter { display: flex; flex-wrap: wrap; gap: 5px; }
.filter button { padding: 8px 11px; border: 0; border-radius: 22px; background: #f3f5f0; color: #829087; font-size: 10px; }
.filter button.aktif { background: #697c70; color: white; }
.alat > input { width: 150px; min-width: 0; padding: 9px 11px; border: 1px solid #e8ece7; border-radius: 22px; outline-color: #98aa9d; font-size: 10px; }
.opsi { display: flex; align-items: center; gap: 7px; margin-bottom: 11px; color: #89958e; font-size: 10px; }
.opsi input { accent-color: #697c70; }

.daftar { display: grid; gap: 8px; }
.item { display: flex; align-items: flex-start; gap: 12px; padding: 15px 12px; border: 1px solid #edf0ea; border-radius: 15px; background: white; }
.item.sudah { background: #f4f8f3; }
.item > input { flex: 0 0 18px; width: 18px; height: 18px; margin-top: 2px; accent-color: #697c70; }
.isi { flex: 1; min-width: 0; }
.isi h3 { overflow-wrap: anywhere; margin: 0; color: #34413a; font-size: 12px; font-weight: 600; }
.sudah h3 { color: #8b9990; text-decoration: line-through; }
.tag-baris { display: flex; flex-wrap: wrap; gap: 6px; margin-top: 8px; }
.tag { padding: 4px 7px; border-radius: 6px; background: #eaf0e9; color: #6a8173; font-size: 9px; }
.p-tinggi { background: #fae9e7; color: #b45e58; }
.p-sedang { background: #f7f0df; color: #9c7d3f; }
.p-rendah { background: #edf4ed; color: #64836c; }
.hapus { width: 25px; height: 25px; border: 0; border-radius: 8px; background: none; color: #829187; font-size: 20px; }
.hapus:hover { background: #fae9e7; color: #b45e58; }
.isi textarea { display: block; width: 100%; margin-top: 8px; padding: 9px; border: 1px solid #e3eae2; border-radius: 9px; resize: vertical; font-size: 11px; }
.kosong { padding: 30px 12px; text-align: center; }
.kosong span { color: #9bb0a0; font-size: 35px; }
.kosong h3 { margin: 8px 0 5px; font: 18px 'Playfair Display', serif; }
.kosong p { color: #89958e; font-size: 11px; }
.kutipan { display: flex; align-items: center; gap: 12px; margin-top: auto; padding: 14px 17px; border-radius: 17px; background: #edf2e9; color: #6f8678; }
.kutipan span { flex: 1; font: italic 12px 'Playfair Display', serif; }

@media (max-width: 760px) {
  .halaman { padding: 12px; }
  .aplikasi { grid-template-columns: 1fr; gap: 13px; }
  .sidebar { min-height: 0; padding: 13px; border-radius: 18px; }
  .merek { padding: 0 5px 12px; }
  .merek h1 { font-size: 27px; }
  .pesan, .profil { display: none; }
  nav { grid-template-columns: repeat(3, minmax(0, 1fr)); gap: 4px; }
  nav button { padding: 9px 3px; text-align: center; font-size: 10px; }
  .konten { gap: 12px; }
  .sambutan { align-items: flex-start; flex-direction: column; padding: 19px; }
  .ringkasan { gap: 7px; }
  .ringkasan article { gap: 7px; padding: 12px 8px; }
  .ringkasan article > span { flex-basis: 29px; height: 29px; }
  .ringkasan small { font-size: 9px; }
  .ringkasan strong { font-size: 21px; }
  .kartu { padding: 16px; }
  .detail-form { grid-template-columns: 1fr; }
  .alat { align-items: stretch; flex-direction: column; }
  .alat > input { width: 100%; }
  .item { gap: 8px; padding: 12px 8px; }
}
</style>
