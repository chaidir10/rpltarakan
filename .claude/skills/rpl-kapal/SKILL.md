---
name: rpl-kapal
description: Panduan kerja aplikasi RPL Kapal BKK Kelas I Tarakan (index.html satu berkas) - struktur kode, alur surat tugas/tagihan, data & migrasi, DOCX/PDF/QR, kontrol kustom, cara mengubah, menguji, dan push. Pakai setiap kali mengubah, memperbaiki, atau menjelaskan aplikasi ini.
---

# RPL Kapal - BKK Kelas I Tarakan

Prototipe aplikasi Rekening Pemerintah Lainnya (RPL) untuk biaya transportasi pemeriksaan kapal.
Satu-satunya berkas aplikasi: `index.html` (~575 KB, HTML + CSS + JS tanpa pustaka luar, tanpa build).
Repo publik: https://github.com/chaidir10/rpltarakan (cabang `main`). Bahasa antarmuka, komentar, dan pesan commit: **Bahasa Indonesia**.

## Aturan kerja dengan pemilik repo

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
| Wilker (5) | `const WILKER=`, `akunWilker` |
| Petugas bawaan (30 nama) & akun petugas | `PETUGAS_AWAL`, `akunPetugas`, `lengkapiAkunPetugas`, `akunDariPetugas`, `PW_PETUGAS` |
| Migrasi data | `function migrasi(d)` (versi data sekarang **12**) |
| Data awal | `function seedState()` (harus `v:12`) |
| Sesi, peran, hak akses | `loadSession`, `isAdmin`, `isPetugas`, `can`, `PERM_PETUGAS`, `stSaya`, `bolehLihat`, `milikSaya`, `terbatas` |
| Status surat | `const STAGE=`, `function stage(s)`, `tanpaTagihan`, `perluSatker` |
| Jarak & transport | `JARAK`, `PILIH_JARAK`, `jarakST`, `pilihJarak`, `MODA`, `barisTagihan`, `aturBarisTagihan` |
| Kontrol kustom (select/tanggal/saran/berkas) | `Kontrol kustom`, `kkSelect`, `kkTanggal`, `kkSaran`, `kkBerkas`, `kkTerapkan` |
| Modal, toast, field | `openModal`, `closeModal`, `confirmBox`, `toast`, `const field=` |
| Model surat & DOCX | `LAYOUT`, `docSTInti`, `docST`, `docSPD`, `spdDepan`, `spdBelakang`, `nominatifHalaman`, `buildDocx`, `xXml`, `pXml`, `zipStore` |
| Pratinjau HTML surat | `docHtml`, `pHtml`, `xHtml`, `fitSheets` |
| PDF (penulis sendiri, font base-14) | `pdfNominatif`, `pdfPernyataan`, `lebarTeks`, `bungkus` |
| QR tanda tangan & keaslian | `KUNCI_QR`, `kodeQrTtd`, `ttdData`, `kanvasQr`, `vVerif`, `cariQrSurat`, `cariQrPdf` |
| Router & kerangka | `function route()`, `renderShell`, `NAV_SATKER`, `NAV_SET`, `renderLogin`, `doLogin` |
| Aksi klik & form | objek `actions` (atribut `data-act`), `forms` (`data-form`), handler `data-change`/`data-input` |

Pola UI: delegasi event. Tombol memakai `data-act="namaAksi"`, form `data-form="nama"` + `novalidate` (validasi manual dengan `toast(...,'bad')`), perubahan input `data-change`/`data-input`. Render ulang lewat `route()`.

## Peran & rute

- **Admin satker** (`admin`/`admin123`): semua data, semua pengaturan.
- **User wilker** (`malundung`, `tanjungselor`, `berau`, `nunukan`, `sebatik` / `user123`): surat wilkernya; hak mengikuti `DB.settings.perms` (diatur admin di Hak Akses).
- **Petugas** (username = NIP, atau nama depan bila tanpa NIP / `petugas123`): membuat surat sendiri dan memilih wilker di form; hanya melihat surat yang dibuat atau diikutinya; hak dibatasi `PERM_PETUGAS` (surat & tagihan saja).
- **Agen**: tanpa login, lewat tautan WhatsApp `#/t/<token>`.
- Rute: `#/dash`, `#/surat`, `#/surat/baru`, `#/surat/<id>`, `#/surat/ubah/<id>`, `#/setoran`, `#/verifikasi` (Scan QR, juga publik), `#/akun`, `#/pengaturan`, `#/set/{users,petugas,agen,pejabat,kop,app,akses,data}`, `#/t/<token>`.

## Alur surat tugas

1. Buat surat: jenis pemeriksaan (boleh kosong sampai simpan), jumlah petugas, **jarak sandar** (`dekat` < 4 mil / `jauh` >= 4 mil), agen, kapal, petugas, penandatangan.
2. Unduh DOCX -> disahkan di **Srikandi** -> catat nomor + **wajib** unggah surat bertanda tangan (`catatST`).
3. Agen membuka tautan, memilih dijemput (`jemput='ya'`) atau tidak.
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

## Hal yang masih terbuka

- 21 petugas belum punya NIP/pangkat/jabatan (tercetak kosong di surat tugas, SPD, nominatif).
- Nominatif di DOCX dibuat sebelum tagihan ada (penginapan "-"); bisa berbeda dengan tagihan akhir.
- Baris II halaman belakang SPD mengikuti contoh ("Berangkat dari Tarakan, Ke tujuan"), belum dibalik.
- Nama aplikasi "RPL Kapal" vs "RPL Tarakan" belum diputuskan; belum diuji di ponsel/printer/kamera nyata.
