# RASTER

Susun foto jadi lembar cetak. Unggah beberapa foto, RASTER memilih tata letak
yang paling cocok dengan bentuknya — potret, lanskap, persegi — lalu menyusunnya
ke lembar A4, A5, Letter, persegi 200 mm, atau kanvas Instagram.

Semuanya berjalan di peramban. Tidak ada server, tidak ada unggahan: foto tidak
pernah meninggalkan komputermu.

**Demo:** https://ddsng.github.io/raster-print/

---

## Cara memasang di GitHub Pages

1. Buka https://github.com/new, beri nama repo **`raster-print`**, pilih
   **Public**, lalu **Create repository**.
2. Di halaman repo yang baru, klik **uploading an existing file**.
3. Seret semua isi folder ini ke sana — `index.html`, `README.md`,
   `.nojekyll`, `.gitignore`, `LICENSE`. Jangan seret foldernya, seret isinya,
   supaya `index.html` berada di akar repo.
   > Kalau `.nojekyll` dan `.gitignore` tidak ikut terlihat saat memilih berkas,
   > aktifkan tampilan berkas tersembunyi: `Cmd`+`Shift`+`.` di macOS,
   > atau centang *Hidden items* di tab *View* pada Windows.
4. Klik **Commit changes**.
5. Masuk ke **Settings → Pages**. Pada *Source* pilih **Deploy from a branch**,
   pilih branch **main** dan folder **/ (root)**, lalu **Save**.
6. Tunggu satu sampai dua menit. Situsmu hidup di
   `https://USERNAME.github.io/raster-print/` — ganti `USERNAME` dengan nama
   akun GitHub-mu.

Untuk memperbarui nanti: buka `index.html` di GitHub, klik ikon pensil, sunting,
commit. Pages menerbitkan ulang sendiri.

---

## Apa yang berubah dari versi artifact

Versi artifact berjalan di dalam sandbox yang menutup dua jalur sekaligus.
Versi ini tidak.

| | Di artifact | Di GitHub Pages |
|---|---|---|
| `window.print()` | diabaikan sandbox | jalan — tombol **Cetak** |
| Unduhan berkas | lewat host, dibatasi **16 MB** | `<a download>` biasa, tanpa batas |
| Jumlah halaman PDF | mentok ±10–15 lembar A4 sebelum kena batas ukuran | tidak dibatasi |

Tiga perubahan teknis yang membuatnya begitu:

- **Unduhan langsung.** Jalur `claude.use("downloads")` — yang meminta
  konfirmasi viewer dan menolak apa pun di atas 16 MB — diganti elemen
  `<a download>` biasa. Tidak ada plafon ukuran.
- **PDF dirakit dari potongan Blob.** Sebelumnya tiap halaman di-decode jadi
  `Uint8Array`, semuanya ditahan di memori, lalu disalin sekali lagi ke satu
  array raksasa di akhir — dua salinan penuh di heap JavaScript. Sekarang JPEG
  tiap halaman tetap berupa `Blob` dan langsung dioper ke konstruktor `Blob`
  terakhir, jadi datanya tidak pernah masuk heap dan peramban bebas
  menumpahkannya ke disk. Inilah yang membuat ratusan halaman muat.
- **Cetak native.** Tombol **Cetak** memanggil dialog cetak peramban. CSS
  `@page` sudah mengunci ukuran kertas ke format yang kamu pilih dengan margin
  nol, jadi *Save as PDF* di dialog itu menghasilkan PDF berukuran persis,
  satu lembar per halaman, tanpa plafon jumlah halaman sama sekali — dan foto
  digambar pada resolusi printer, bukan lewat canvas.

## Dua jalur PDF, kapan pakai yang mana

**Simpan PDF** membangun PDF sendiri: satu JPEG 300 dpi penuh per halaman,
langsung terunduh. Hasilnya konsisten di semua peramban dan ukuran halamannya
dijamin persis. Pakai ini untuk kiriman ke percetakan.

**Cetak** menyerahkannya ke peramban. Berkasnya lebih kecil, foto tidak melewati
canvas sehingga ketajamannya maksimal, dan tidak ada batas jumlah halaman sama
sekali. Pakai ini untuk mencetak sendiri atau untuk pekerjaan yang sangat
panjang. Pastikan *Margins* di dialog diset **None** dan *Scale* **100%**.

## Sampul album

Centang **Tambahkan halaman sampul** dan sampul menjadi lembar pertama, di depan
semua lembar foto. Judul dan subjudul hanya tercetak di sampul — lembar foto di
belakangnya tetap bersih tanpa teks sama sekali.

Dua gaya, bisa ditukar kapan saja dan langsung terlihat di pratinjau:

- **Berbingkai** — foto duduk di dalam margin mengisi bagian atas lembar, judul
  ditata di bawahnya pada kertas putih. Tenang, seperti album foto cetak.
- **Penuh** — foto memenuhi seluruh lembar tanpa margin, judul ditumpuk di
  atasnya dengan gradasi gelap di bawah supaya teks tetap terbaca.

Foto sampulnya bisa diunggah terpisah, atau ambil dari album lewat **Pakai foto
ke-1**. Kalau kamu mengosongkan fotonya, sampul jadi halaman judul tipografis —
judul di tengah lembar putih.

Ukuran huruf sampul dihitung sebagai pecahan dari sisi pendek halaman, bukan
angka tetap. Jadi sampul A5 punya proporsi yang sama dengan A4, dan sampul
Instagram Story ikut menyesuaikan tanpa perlu diatur ulang.

## Keterangan halaman di daftar urutan

Tiap baris di panel **Urutan & halaman** memakai lencana `L03` yang menyebut
lembar tempat foto itu akan dicetak, dan daftarnya dipotong pembatas per lembar
— `Lembar 03 · 4 foto`. Nomornya ikut bergeser sendiri saat kamu mengubah foto
per halaman, menyeret urutan, atau menyalakan sampul (semua foto mundur satu
lembar). Untuk kanvas Instagram lencananya berbunyi `S03` dan pembatasnya
menyebut *Slide*.

Saat **Kelompokkan berdasarkan orientasi** aktif, urutan daftar tidak lagi sama
dengan urutan cetak, jadi pembatasnya disembunyikan — lencana per barisnya tetap
benar dan tetap bisa dipakai mencari.

## Fitur

- Halaman sampul opsional dengan judul dan subjudul, dua gaya tata letak
- Lencana nomor lembar dan pembatas per halaman di daftar urutan
- Tata letak otomatis — mesin templat mencocokkan bentuk sel dengan rasio aspek
  tiap foto, jadi foto potret tidak dipaksa masuk kotak lanskap
- Format cetak (A4, A5, Letter, persegi 200 mm) dan kanvas Instagram
  (1:1, 4:5, 1.91:1, 9:16) dalam satu mesin tata letak
- Potret / lanskap, 1–12 foto per halaman atau penyeimbangan otomatis
- Isi penuh atau muat utuh, jarak antar foto dan margin halaman yang bisa diatur
- Urutkan ulang dengan seret, kelompokkan berdasarkan orientasi, acak
- Ekspor PNG 300 dpi per lembar
- Tanda potong di pratinjau, hilang saat dicetak
- Mode terang dan gelap mengikuti setelan sistem

## Lisensi

MIT — lihat [LICENSE](LICENSE).
