---
name: safira
description: Panduan kerja aplikasi SAFIRA (Sistem Administrasi Fasilitasi Inspeksi, RPL, dan Anggaran Kapal; dulu "RPL Kapal") BKK Kelas I Tarakan (index.html satu berkas) - struktur kode, alur surat tugas/tagihan, data & migrasi, DOCX/PDF/QR, kontrol kustom, cara mengubah, menguji, dan push. Pakai setiap kali mengubah, memperbaiki, atau menjelaskan aplikasi ini.
---

# SAFIRA - BKK Kelas I Tarakan

**SAFIRA** = Sistem Administrasi Fasilitasi Inspeksi, RPL, dan Anggaran Kapal (nama resmi sejak 2026-11-05; sebelumnya "RPL Kapal", sempat diusulkan "ASTRAVA" tetapi tidak pernah dirilis). Nama bawaan di `NAMA_APP`/`KEPANJANGAN_APP`; admin masih bisa mengubah nama di pengaturan Aplikasi (`DB.settings.appName`). Pengenal teknis lama sengaja **tidak** diganti agar data & QR lama tetap berlaku: IndexedDB `rpl-bkk-tarakan`, kunci `rpl.theme`/`rpl.session`/`rpl.sort`, kanal `rpl-sync`, kunci QR `KUNCI_QR`. "RPL" di teks aplikasi tetap berarti Rekening Pemerintah Lainnya.

Prototipe aplikasi surat tugas pemeriksaan kapal dan biaya transportasinya lewat Rekening Pemerintah Lainnya (RPL).
Satu-satunya berkas aplikasi: `index.html` (~575 KB, HTML + CSS + JS tanpa pustaka luar, tanpa build).
Repo publik: https://github.com/chaidir10/rpltarakan (cabang `main`). Bahasa antarmuka, komentar, dan pesan commit: **Bahasa Indonesia**.

## Aturan kerja dengan pemilik repo

- **Wajib perbarui berkas ini setiap ada perubahan.** Setiap perubahan aplikasi (fitur, data/versi, aturan bisnis, cara uji) harus dicatat di SKILL.md ini pada commit yang sama: perbarui bagian terkait dan tambahkan satu baris di **Riwayat perubahan** (paling bawah). Berkas ini harus lengkap dan jelas agar sesi berikutnya tidak bingung.
- **Auto push**: setelah perubahan diuji dan lulus, langsung commit + push ke `main` (pemilik sudah meminta). Akhiri pesan commit dengan baris `Co-Authored-By` sesuai pengingat sistem.
- Pemilik kadang commit sendiri (mis. "rapikan titik", "baru"). Selalu cek `git log -3` dan `git status` sebelum commit; jangan menimpa kerja mereka.
- **Bahaya editor basi**: VS Code pernah menyimpan tab lama sehingga `index.html` mundur belasan commit. Sebelum commit, pastikan fitur terbaru masih ada (mis. `grep -c "function kkSelect" index.html`). Bila tertimpa: simpan salinan, `git checkout <commit-terbaru> -- index.html`, commit pemulihan (jangan force push).
- Repo publik berisi data nyata (nama/NIP petugas, agen) dan sandi bawaan. Pemilik memilih "publik apa adanya"; jangan menambah rahasia baru.
- Email pengguna hanya untuk identitas; jangan dikirim ke layanan lain.
- Jangan membuat ulang modal yang bisa ditutup klik luar: modal hanya tertutup lewat tombol X atau tombol Batal/Tutup.

## Peta kode (cari dengan Grep, nomor baris berubah-ubah)

| Bagian | Cari |
|---|---|
| Tema awal & CSS (variabel `--ink`, `--brand`, `--surface`, `[data-theme=dark]`) | `<style>` di atas |
| Data agen bawaan (teks `|`) | `id="seed-agen"` |
| jsQR 1.4.0 (Apache-2.0), dimuat lewat `(0,eval)` | `id="lib-jsqr"`, `const jsqr=` |
| Penyimpanan IndexedDB `rpl-bkk-tarakan` (store `kv`, `files`), sinkron antar-tab | `const store=`, `function save`, `flush`, `rpl-sync` |
| Jenis pemeriksaan | `const JENIS=`, `kunciJenis`, `urutJenis` |
| Wilker (5) | `const WILKER=` (akun wilker bersama sudah tidak dipakai) |
| Pegawai bawaan (30 nama) & akun | `PETUGAS_AWAL`, `akunPetugas`, `lengkapiAkunPetugas`, `akunDariPetugas`, `PW_PETUGAS` |
| Migrasi data | `function migrasi(d)` (versi data sekarang **17**) |
| Data awal | `function seedState()` (harus `v:17`) |
| Sesi, peran, hak akses | `loadSession`, `PERAN_BAWAAN`, `daftarPeran`, `punya`, `isAdmin`, `lihatSemua`, `can`, `stSaya`, `bolehLihat`, `milikSaya`, `terbatas`, `sayaPetugas` |
| Status surat | `const STAGE=`, `function stage(s)`, `tanpaTagihan`, `perluSatker` |
| Jarak & transport | `JARAK`, `PILIH_JARAK`, `jarakST`, `pilihJarak`, `MODA`, `barisTagihan`, `aturBarisTagihan` |
| Kontrol kustom (select/tanggal/saran/berkas) | `Kontrol kustom`, `kkSelect`, `kkTanggal`, `kkSaran`, `kkBerkas`, `kkTerapkan` |
| Modal, toast, field | `openModal`, `closeModal`, `confirmBox`, `toast`, `const field=` |
| Model surat & DOCX | `LAYOUT`, `docSTInti`, `docST`, `docSPD`, `spdDepan`, `spdBelakang`, `nominatifHalaman`, `buildDocx`, `xXml`, `pXml`, `zipStore` |
| Pratinjau HTML surat | `docHtml`, `pHtml`, `xHtml`, `fitSheets` |
| PDF (penulis sendiri, font base-14) | `pdfNominatif`, `pdfPernyataan`, `lebarTeks`, `bungkus` |
| QR tanda tangan & keaslian | `KUNCI_QR`, `kodeQrTtd`, `ttdData`, `kanvasQr`, `vVerif`, `cariQrSurat`, `cariQrPdf` |
| Portal agen (login agen, permohonan, pelacakan, SPD bercap) | `isAgen`, `renderAgenPortal`, `vAgenBeranda`, `vAgenPermohonanForm`, `vAgenST`, `langkahAgen`, `progresHtml`, `renderDaftarAgen`, `forms.daftarAgen/permohonan/spdCap/sandiAgen` |
| Permohonan agen (sisi satker) | `DB.permohonan`, `vPermohonan`, `permohonanModal`, `vAgenMenunggu`, `kartuDokAgen`, rute `#/permohonan`, `#/surat/dari/<id>` |
| Keuangan (dana masuk, penyaluran, laporan) | `rekeningOf`, `hakPetugas`, `salurOf`, `sisaSalur`, `statusSalur`, `dataLaporan`, `vKeuangan`, `vSalur`, `kartuRekeningSaya`, `csvLaporan`, `pdfLaporan`, `forms.salur/rekeningSaya`, rute `#/keuangan`, `#/keuangan/<stId>` |
| Monev RPL oleh SKI | `MONEV_ITEM`, `DB.monev`, `monevST`, `temuanMonev`, `vMonev`, `vMonevForm`, `kartuMonev`, `pdfMonev`, `logoSkiBawaan`, `forms.monev`, rute `#/monev`, `#/monev/<stId>` |
| Router & kerangka | `function route()`, `renderShell`, `NAV_SATKER`, `NAV_SET`, `renderLogin`, `doLogin` |
| Aksi klik & form | objek `actions` (atribut `data-act`), `forms` (`data-form`), handler `data-change`/`data-input` |

Pola UI: delegasi event. Tombol memakai `data-act="namaAksi"`, form `data-form="nama"` + `novalidate` (validasi manual dengan `toast(...,'bad')`), perubahan input `data-change`/`data-input`. Render ulang lewat `route()`.

## Pengguna, peran & rute (sejak data versi 13)

- **Satu orang satu catatan.** Data orang ada di `DB.petugas` (nama, NIP, pangkat, jabatan, wilker asal, `aktif` = bisa ditugaskan). Akun masuk (opsional) ada di `DB.users` dan terhubung lewat `petugasId` (`akunDariPetugas`). Keduanya dikelola dari satu menu **Pegawai & Akun** (`#/set/pegawai`, `vSetPegawai`, `pegawaiModal`, `forms.pegawai`, impor/ekspor CSV `imporPegawai`). Rute lama `set/users` dan `set/petugas` dialihkan ke sana.
- **Peran jamak per akun**: `user.peran = ['petugas','validator',...]` (tidak ada lagi `role`). Peran adalah **data** di `DB.settings.peran` (`{id,nama,warna,ket,lingkup,bawaan}`); admin bisa menambah/mengubah/menghapus peran tambahan di Hak Akses (`peranModal`, `forms.peran`). Bawaan: Petugas (lingkup `sendiri`), **Keuangan** (id `validator`, dulu "Validator keuangan"), Pimpinan, **Verifikator SKI** (sejak v16), Admin satker (lingkup `semua`). Pakai `daftarPeran()`, `cariPeran(id)`, `peranTupel()`, `peranOf(u)`, `teksPeran(u)`; jangan menulis daftar peran tetap di kode.
- **Hak per peran**: `DB.settings.permsPeran[peran][fitur]` (fitur di `PERMS`). `can(fitur)` = admin, atau salah satu peran akun punya hak itu. Lingkup data: `lihatSemua()` (ada peran berlingkup `semua`) atau hanya surat yang dibuat/diikuti (`milikSaya`, `stSaya`, `bolehLihat`).
- **Akun contoh**: `admin`/`admin123`; pegawai masuk dengan NIP (atau nama depan bila tanpa NIP) / `petugas123`. Akun dengan sandi bawaan diberi `gantiSandi` dan melihat pengingat sampai menggantinya. Akun wilker bersama lama (`malundung`, `berau`, ...) sudah dinonaktifkan (`bersama:true`); jangan dipakai untuk uji, pakai `[data-l=admin]` atau NIP.
- Pegawai baru otomatis punya akun (sakelar "Punya akun" menyala, peran Petugas); admin bisa mematikannya. "Bisa ditugaskan" (muncul di pilihan petugas surat) terpisah dari "Akun aktif" (boleh masuk).
- Harus selalu ada minimal satu admin aktif; admin tidak bisa mencabut peran admin/akun dirinya sendiri.
- **Keuangan** (peran `validator`, hak `validasi`, `export`, `laporan`, `salur`; Pimpinan bawaan punya `laporan` saja): menu **Keuangan** (`#/keuangan`) = dasbor dana RPL masuk (pembayaran agen yang **valid**, menurut tanggal transfer agen `s.bayar.tgl`), grafik bulanan (`kartuPendapatan`), dana masuk bulan terpilih, **perlu disalurkan**, penyaluran per petugas bulan itu, dan daftar petugas yang belum punya rekening. **Penyaluran** (`#/keuangan/<stId>`, `vSalur`): hak tiap petugas = rincian tagihannya (`biayaBaris`: transport + penginapan + uang harian); keuangan menandai "ditransfer" per petugas atau sekaligus (tanggal, bukti transfer opsional, catatan) ke `s.salur[petugasId] = {jumlah, tgl, at, oleh, rek, buktiId, catatan}` (snapshot rekening), bisa dibatalkan. Penyaluran **diblokir bila rekening petugas kosong**. **Laporan** bulanan/tahunan: CSV (`csvLaporan`) dan PDF A4 mendatar banyak halaman (`pdfLaporan`).
- **Rekening petugas** di data pegawai `petugas.rekening = {bank, nomor, nama}`: diisi petugas sendiri di **Akun** (`kartuRekeningSaya`, juga menampilkan dana diterima/menunggu) atau admin di form pegawai; ikut ekspor/impor CSV pegawai (`bank`, `no_rekening`, `nama_rekening`). Petugas tanpa rekening melihat pengingat di setiap halaman.
- **Verifikator SKI** (Satuan Kepatuhan Internal, hak `monev`): menu **Monev SKI** (`#/monev`) berisi daftar surat tugas (saring belum/sudah dimonev). **Satu formulir MONEV RPL per surat tugas** (`DB.monev[]` = `{stId, hasil:{rba,jumlah,phqc,cop,p3k,sbm,double,klaim,bukti: {s:sesuai|tidak, k:keterangan}}, kesimpulan:memenuhi|tidak, lokasi, tanggal, petugasTtd, verifikatorId/Nama/Nip}`). Halaman isian menampilkan data surat, data dukung (berkas TTE, SPD bercap, surat permohonan, bukti bayar, rincian klaim per petugas beserta bukti) dan **temuan otomatis** sebagai bahan (`temuanMonev`: jumlah petugas melebihi contoh, uang harian/transport tidak sesuai SBM, tagihan ganda kapal & tanggal sama, petugas diklaim di surat lain pada hari yang sama, bukti dukung kurang); keputusan tetap dipilih verifikator. Petugas penanda tangan dipilih dari petugas surat (bawaan petugas pertama); verifikator = akun yang mengisi (nama/NIP dari data pegawainya).
- **Cetakan monev** (`pdfMonev`) meniru lampiran SKI: kop "Satuan Kepatuhan Internal" (Times) + subjudul oranye, logo SKI kiri (unggah di halaman Monev SKI, `DB.settings.logoSki`; bawaan gambar pengganti `logoSkiBawaan`), logo kop Kemenkes kanan, garis toska & lime; tabel No/Rincian/Hasil/Keterangan dengan kotak dicentang; baris "Dalam monitoring dan evaluasi RPL di ..." mengecil/dibungkus agar muat; kesimpulan mencoret pilihan yang tidak berlaku; tanda tangan Petugas & Verifikator SKI. Hasil monev juga tampil di detail surat (kartu **Monev SKI**, bisa dicetak oleh yang bisa melihat surat).
- **Agen** punya dua jalan: (1) tautan WhatsApp tanpa login `#/t/<token>` (tetap berlaku), (2) **akun agen** (sejak data v15). Agen **mendaftar sendiri** di `#/daftar` (nama perusahaan, email, WhatsApp, kab/kota, sandi) lalu **disetujui admin** di Data Agen (`vAgenMenunggu`, aksi `setujuiAgen`/`tolakAgen`). Akun disimpan di `agen.akun = {password, status:menunggu|aktif|ditolak, daftar, alasan}`; username = email. Sesi agen `{type:'agen'}` dan `ME._type==='agen'` (`isAgen()`); agen hanya melihat surat dengan `s.agenId===ME.id` dan permohonannya sendiri. Akun contoh: email agen PT. BNE SAMUDERA AGENSI / `agen123`.
- **Portal agen** (`#/a`, `renderAgenPortal`): beranda daftar permohonan & surat dengan progress bar; `#/a/baru` ajukan permohonan (nomor & tanggal surat, hal, kapal, GT, posisi, rencana tanggal, wilker opsional, **unggah surat permohonan wajib**, catatan); `#/a/p/<id>` pelacakan permohonan (bisa dibatalkan selama `baru`); `#/a/s/<id>` pelacakan & dokumen bersebelahan di atas, lalu invoice/aksi lama selebar halaman di bawah (lebar maksimal portal 1264px) (`vAgenKapal`: pilih transport, invoice, bukti bayar, surat pernyataan) + dokumen: unduh surat tugas & SPD hasil TTE Srikandi (`s.fileST`) dan **unggah SPD bertanda tangan & cap basah agen** (`s.spdCap`, hanya jarak >= 4 mil); `#/a/akun` ganti sandi.
- **Langkah pelacakan** (`langkahAgen`): permohonan dikirim -> diterima & diproses satker (atau ditolak + alasan) -> surat tugas dibuat -> disahkan TTE Srikandi -> konfirmasi transportasi -> SPD dicap agen (jauh) -> tagihan terbit (atau dilewati bila dijemput & dekat) -> pembayaran & validasi -> selesai. Persen = langkah selesai / langkah yang berlaku.
- **Sisi satker**: menu **Permohonan Agen** (`#/permohonan`, hitungan permohonan baru) -> detail (lihat berkas) -> **Proses jadi surat tugas** membuka form surat yang sudah terisi dari permohonan (`#/surat/dari/<id>`, `vSuratForm(null,pm)`); saat disimpan permohonan menjadi `diproses`, `pm.stId` dan `s.permohonanId` terisi. Bisa juga **Tolak** dengan alasan. Detail surat menampilkan kartu **Dokumen dari agen** (surat permohonan, status SPD bercap).
- GT dari agen memakai titik ribuan dan koma desimal (`1.250` -> `1250`, `36,96`).
- Rute: `#/dash`, `#/surat`, `#/surat/baru`, `#/surat/<id>`, `#/surat/ubah/<id>`, `#/setoran`, `#/verifikasi` (Scan QR, juga publik), `#/akun`, `#/pengaturan`, `#/set/{pegawai,agen,pejabat,kop,app,akses,data}`, `#/t/<token>`.

## Alur surat tugas

1. Buat surat: jenis pemeriksaan (boleh kosong sampai simpan), jumlah petugas, **jarak sandar** (`dekat` < 4 mil / `jauh` >= 4 mil), agen, kapal, petugas, penandatangan.
2. Unduh DOCX -> disahkan di **Srikandi** -> catat nomor + **wajib** unggah surat bertanda tangan (`catatST`).
3. Agen membuka tautan atau portal agen, memilih dijemput (`jemput='ya'`) atau tidak. Surat bisa berawal dari **permohonan agen** (lihat Portal agen).
4. Tagihan: `dekat`+dijemput = tanpa tagihan (stage `jemput`, terbit surat pernyataan PDF). Selain itu `perlu` tagihan.
   - < 4 mil: per petugas pilih cara transport `pribadi` (wajib struk bensin) atau `online` (wajib bukti pesan); disediakan agen = Rp0.
   - >= 4 mil: uang harian `tarifLuar` (430.000) per orang + penginapan diisi; transport at cost Rp0.
5. Agen unggah bukti bayar -> satker validasi -> `valid` / `tolak`.

## Dokumen

- **Tag Srikandi** (jangan diganti teks biasa): `${nomor_naskah}`, `${ttd_pengirim}`, `${nama_pengirim}`, `${nip_pengirim}` (`TAG_NOMOR`, `TAG_TTD`, `TAG_NAMA`, `TAG_NIP`).
- DOCX surat tugas mengikuti contoh lampiran (ukuran dalam twip di `LAYOUT`). 1-2 petugas: nama di badan surat; 3+: lampiran halaman 2.
- Jarak `jauh`: DOCX ditambah bagian lain (`d.extra`): SPD satu halaman per petugas + halaman belakang SPD (tegak) + nominatif (mendatar). Gambar Garuda (domain publik) di `GAMBAR.garuda`.
- `docSTInti` = isi surat tugas saja (tidak boleh berubah karena jarak); `docST` = inti + bagian SPD bila jauh.
- Surat pernyataan (PDF, tanpa kop): agen di kiri, petugas berjejer di kanan, tanda tangan berupa QR berlogo; QR keaslian kecil di catatan kaki kiri bawah. QR hanya terbaca aplikasi (payload terenkripsi + MAC; kuncinya ada di berkas, jadi bukan pengaman sungguhan - prototipe).

## Kontrol kustom (penting saat menguji)

`select.inp`, `input[list]`, `input[type=date]`, dan `input[type=file].inp` otomatis diganti tampilan bertema oleh `MutationObserver` (`kkTerapkan`). Elemen asli **tetap ada tapi tersembunyi** (select diberi `.kk-asli`, tanggal jadi `type=hidden`), jadi `name`, nilai form, dan event `change` tetap berlaku.
- Mengisi dari kode/uji: set `.value` lalu kirim event `input`/`change` pada elemen asli (tampilan ikut sinkron).
- Mengukur posisi/lebar atau mengklik: pakai tombol tampilan `#kk_<id-asli>` (mis. `#kk_f_wilker`, `#kk_f_tanggal`), bukan `#f_wilker`.
- Jangan menambah `<select>`/`type=date` dengan gaya bawaan browser; cukup beri kelas `inp`, kontrol kustom terpasang sendiri.

## Mengubah data atau struktur

- Ubah bentuk data -> naikkan versi: `migrasi` (`if(v>=N)return false;` + langkah baru, langkah lama di blok `if(v<...)`), `seedState` `v:N`, dan semua uji yang memeriksa `DB.v===N`.
- Tambah petugas bawaan -> `PETUGAS_AWAL` (migrasi menambahkan yang belum ada berdasarkan `namaInti`, tanpa duplikat).
- Mobile: titik patah utama `max-width:860px`; cek juga 390 dan 320 px (segmen, tombol, judul tidak terpotong / meluap).

## Cara mengedit `index.html`

Berkas besar dan berbaris panjang (gambar base64), dan memakai CRLF.
- Untuk perubahan kecil pakai Edit; untuk banyak perubahan tulis skrip Node "patch" yang mengganti potongan persis sekali saja dan memeriksa sintaks:
  ```js
  const fs=require('fs');const F='C:/Users/USER/Downloads/rpl/index.html';let c=fs.readFileSync(F,'utf8');
  const crlf=c.includes('\r\n');const N=s=>crlf?s.replace(/\r?\n/g,'\r\n'):s;
  const rep=(a,b)=>{a=N(a);b=N(b);const n=c.split(a).length-1;if(n!==1)throw new Error(n+'x: '+a.slice(0,90));c=c.replace(a,()=>b);};
  // rep('teks lama persis','teks baru');
  const s0=c.lastIndexOf('<script>');new Function(c.slice(c.indexOf("'use strict';",s0),c.lastIndexOf('</script>')));
  fs.writeFileSync(F,c);
  ```
- Simpan cadangan sebelum patch besar (`cp index.html <scratchpad>/idx-vNN.bak`).
- Jangan `Read` seluruh berkas (terlalu besar); cari dengan Grep lalu baca rentang baris, atau `awk 'NR>=a&&NR<=b' | cut -c1-400`.
- Heredoc PowerShell sering gagal; tulis skrip dengan alat Write lalu jalankan dengan `node`.

## Pengujian

- Uji memakai Edge headless lewat CDP (`cdp.js`: `start()`, `b.ev()`, `b.goto()`, `b.size(w,h,mobile)`, `b.shot(nama)`, `b.errs`), dengan klik/papan ketik sungguhan lewat `Input.dispatchMouseEvent`/`Input.dispatchKeyEvent`. Setiap suite `tNN.js` mencetak `LULUS`/`GAGAL` dan diakhiri `SEMUA UJI LULUS`.
- Skrip uji berada di folder scratchpad sesi lama (`%TEMP%\claude\c--Users-USER-Downloads-rpl\<sesi>\scratchpad\`) dan bisa hilang. Bila tidak ada, buat ulang `cdp.js` kecil (Edge `--headless=new --remote-debugging-port`, WebSocket CDP) sebelum menguji.
- DOCX diperiksa dengan Word COM (`topdf.ps1`: buka DOCX, ekspor PDF, hitung halaman). PDF dilihat per halaman dengan `Windows.Data.Pdf` (skrip `pdf2png.ps1`; di PowerShell variabel tidak peka huruf besar, jangan pakai `$page` bersama `$Page`).
- Selalu lihat tangkapan layar terang + gelap, desktop + ponsel, sebelum menyatakan selesai.
- Uji lama yang diketahui usang: `t1` (mencari field `tarif` yang sudah dihapus), `t12` (mencari `#format-hint` yang dihapus atas permintaan). `t38` (pindai foto QR) sangat lambat; jalankan terpisah tanpa batas waktu ketat. Matikan sisa `msedge --headless` bila uji macet.
- Untuk masuk di uji: `doLogin('admin','admin123')` atau tombol `[data-l=admin]`; pegawai `[data-l="198002202006041003"]` (NIP perlu tanda kutip di selektor CSS) atau `doLogin(NIP,'petugas123')`. Pegawai yang tidak ada di surat contoh: `ivan`. Admin wajib memilih wilker di form surat (`[name=wilker]`).
- Uji utama model pengguna: `t53` (peran, hak, peran tambahan, migrasi v12->v13), `t6` (validasi pegawai, migrasi v4, tata letak), `t50` (petugas membuat surat sendiri), `t10` (tombol masuk cepat).

## Hal yang masih terbuka

- Logo resmi SKI belum ada di aplikasi; cetakan monev memakai logo pengganti sampai admin mengunggah logo SKI di halaman Monev SKI.
- Belum ada notifikasi WhatsApp/email otomatis ke agen saat status berubah (agen memantau di portal).

- 21 petugas belum punya NIP/pangkat/jabatan (tercetak kosong di surat tugas, SPD, nominatif).
- Nominatif di DOCX dibuat sebelum tagihan ada (penginapan "-"); bisa berbeda dengan tagihan akhir.
- Baris II halaman belakang SPD mengikuti contoh ("Berangkat dari Tarakan, Ke tujuan"), belum dibalik.
- Belum diuji di ponsel/printer/kamera nyata.

## Riwayat perubahan

Tambahkan satu baris per perubahan (terbaru di bawah): tanggal, commit, ringkasan, dan dampak ke data/uji.

- 2026-11-03 `7f32d14` Jarak hanya ditampilkan di jendela tagihan; cara transport per petugas dan klaim berbukti (`e763235`); unggah surat bertanda tangan wajib (`9cf7cdb`).
- 2026-11-04 `6f617b3` Pemulihan setelah `index.html` tertimpa tab editor lama (`e320886`).
- 2026-11-04 `c460dfe` Surat pernyataan: tanda tangan agen di kiri, petugas berjejer di kanan.
- 2026-11-05 `8c48a40` Surat jarak >= 4 mil: DOCX ditambah SPD per petugas, halaman belakang SPD, nominatif mendatar (tag Srikandi).
- 2026-11-05 `a602971` Akun petugas (username NIP, `petugas123`), petugas membuat surat sendiri dan memilih wilker (data v11); `bf9b564` label filter peran dipendekkan.
- 2026-11-05 `1db9975` Kontrol kustom bertema menggantikan select/datalist/tanggal/berkas bawaan browser (commit oleh pemilik).
- 2026-11-05 `05025c7` Daftar 30 petugas BKK Tarakan sebagai data bawaan (data v12).
- 2026-11-05 `2597126` SKILL.md ini dibuat.
- 2026-11-05 `4124940` Model pengguna baru (data v13): menu tunggal **Pegawai & Akun**, peran jamak per akun, peran sebagai data (bisa ditambah di Hak Akses), hak per peran (`permsPeran`), lingkup data per peran, akun wilker bersama dinonaktifkan, impor/ekspor CSV pegawai, pengingat ganti sandi bawaan, minimal satu admin aktif. Uji baru `t53`.
- 2026-11-05 `970c66c` Pegawai baru kembali otomatis punya akun; judul halaman di ponsel boleh dua baris (judul panjang seperti "Pegawai & Akun" tidak terpotong); migrasi data sangat lama (v<10) tanpa `perms` tidak lagi error. Uji lama (t5, t6, t7, t10, t13, t20, t25, t35, t36, t41, t42, t47, t50, t51, t52) disesuaikan ke model pegawai & peran.
- 2026-11-05 `39eefbb` Nama aplikasi resmi **SAFIRA** (data v14): judul tab, meta, halaman masuk (dengan kepanjangan), samping (kepanjangan saat disorot), catatan kaki surat pernyataan "aplikasi SAFIRA"; migrasi mengganti nama bawaan lama "RPL Kapal/RPL Tarakan" tanpa menimpa nama buatan admin. Folder skill diganti menjadi `.claude/skills/safira/`. Uji baru `t54`; uji memeriksa `DB.v===14`.
- 2026-11-05 `24bee2f` **Portal agen** (data v15): daftar mandiri + persetujuan admin, masuk dengan email, ajukan permohonan dengan unggah surat, progress bar pelacakan, unduh surat tugas & SPD TTE Srikandi, unggah SPD bertanda tangan & cap basah agen; sisi satker: menu Permohonan Agen, proses jadi surat tugas (form terisi otomatis) atau tolak, kartu Dokumen dari agen di detail surat. Uji baru `t55`; uji memeriksa `DB.v===15` dan 3 tombol masuk contoh.
- 2026-11-05 `51c01c6` **Peran Verifikator SKI & Monev RPL** (data v16): satu formulir monev per surat tugas dengan temuan otomatis dan data dukung, cetak PDF sesuai lampiran MONEV RPL SKI, unggah logo SKI, kartu Monev SKI di detail surat. Uji baru `t56`; uji memeriksa `DB.v===16` dan 5 peran bawaan.
- 2026-11-05 (commit ini) **Modul Keuangan** (data v17): peran "Validator keuangan" menjadi **Keuangan**; dasbor dana masuk, penyaluran dana ke rekening petugas per surat tugas (diblokir bila rekening kosong), laporan bulanan/tahunan CSV & PDF; rekening petugas di Akun & form pegawai (+ CSV), pengingat rekening, kartu dana saya. Portal agen: invoice kini selebar halaman. Uji baru `t57`; uji memeriksa `DB.v===17` dan kolom "Keuangan".
