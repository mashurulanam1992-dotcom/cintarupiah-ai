# CintaRupiah AI

Platform edukasi **Cinta, Bangga, Paham Rupiah** berbasis web dengan AI, untuk pelajar, pelaku UMKM, dan wisatawan di Bali.
Karya Tim SD No. 2 Tuban untuk AI Web Innovation Challenge, Bali DigiFest 2026 (tema: *Build with AI, Solve for Bali*).

> Aplikasi ini karya peserta lomba dan **bukan produk resmi Bank Indonesia**.

## Description

CintaRupiah AI membantu masyarakat mengenali dan merawat uang Rupiah serta memakai QRIS dengan percaya diri. Aplikasi menggabungkan asisten percakapan, analisis foto uang, buku kas UMKM, latihan kasus, dan kuis bersertifikat dalam satu halaman web yang ringan dan dapat dibuka di HP.

## Problem & Target User

- **Masalah:** banyak orang belum hafal cara memeriksa keaslian uang (3D: Dilihat, Diraba, Diterawang), cara merawatnya (5 Jangan), cara menukar uang lusuh, dan manfaat QRIS. Wisatawan juga sering kesulitan mengenali pecahan Rupiah.
- **Pengguna:** pelajar, pedagang/UMKM di Bali, wisatawan, dan penyandang tunanetra (lewat pembacaan pecahan dengan suara).

## Key Features

| Fitur | Fungsi | Memakai AI |
|---|---|---|
| Vision AI Authenticator | Analisis awal foto uang: perkiraan pecahan, kondisi fisik, tanda yang terlihat. Hasil selalu berupa indikasi awal, bukan kepastian asli/palsu | Ya (Gemini Vision) |
| Pembaca nominal bersuara | Membacakan perkiraan pecahan dan kondisi uang dalam bahasa Indonesia atau Inggris | Pengenalan pecahan oleh AI, suara dari browser |
| Bli Rupiah AI | Chatbot edukasi Rupiah dan QRIS, dengan pegangan fakta dan riwayat percakapan | Ya (LLM) |
| Buku Kas Harian UMKM | Catat penjualan tunai dan QRIS per hari; AI membuat ringkasan dari angka yang sudah dihitung server | Ya (LLM) |
| Latihan Kasus | AI membuat skenario sehari-hari dengan 3 pilihan; penilaian dilakukan di browser | Ya (LLM) |
| Kalkulator QRIS | Simulasi belanja tunai vs QRIS | Tidak |
| Kuis & Sertifikat | 10 soal acak dari bank soal, sertifikat digital yang dapat dicetak | Tidak |

## Tech Stack

- **Frontend:** HTML, CSS (Tailwind CSS via CDN), JavaScript tanpa framework. Satu file: `index.html`.
- **Backend:** satu Cloudflare Pages Function dalam "advanced mode": `_worker.js`.
- **Database / penyimpanan:** tidak ada database. Cloudflare KV (opsional) untuk cache jawaban chatbot dan pembatas permintaan; data buku kas disimpan di `localStorage` browser pengguna.
- **Hosting:** Cloudflare Pages.

## AI Implementation

| Bagian | Model / metode | Input | Output | Alasan AI diperlukan |
|---|---|---|---|---|
| Scanner foto (`/api/scan`) | Google Gemini (model diatur lewat `GEMINI_MODEL`), mode vision dengan keluaran JSON terstruktur | Satu foto uang (JPEG/PNG/WebP, diperkecil di browser) | `hasil`, `nominal`, `kondisi`, `temuan`, `saran` | Membaca isi gambar: pecahan, kondisi fisik, tanda visual |
| Chatbot (`/api/gemini`) | Workers AI (`llama-3.3-70b-instruct-fp8-fast`) bila tersedia, Gemini sebagai cadangan | Pertanyaan + maks. 6 pesan terakhir | Teks jawaban (streaming) | Menjawab pertanyaan bebas dengan bahasa natural |
| Ringkasan kas (`/api/ringkas`) | Gemini | Angka penjualan harian (tanggal, tunai, QRIS) | Ringkasan teks maks. 4 kalimat | Merangkai ringkasan dan saran dari data; angka dihitung server |
| Latihan kasus (`/api/latihan`) | Gemini, keluaran JSON terstruktur | Satu topik dari daftar tetap | Skenario, 3 pilihan, jawaban benar, penjelasan | Membuat skenario yang bervariasi |

Pengaman yang dipakai:
- Scanner **tidak pernah** menyatakan uang pasti asli atau palsu, karena AI hanya melihat foto dan tidak dapat meraba atau menerawang. Pengguna selalu diarahkan ke pemeriksaan 3D langsung.
- Foto yang sangat gelap atau sangat terang disaring di browser sebelum dikirim.
- Keluaran AI divalidasi dan dipotong panjangnya di server sebelum dikirim ke browser; semua teks ditampilkan lewat `textContent`.
- Chatbot diberi pegangan fakta dan diminta tidak menebak hal di luar itu.
- Soal latihan dibuat AI dan diberi catatan agar aturan resmi dicek ke kanal Bank Indonesia.
- Pembatas permintaan per pengunjung per jam untuk menjaga kuota.

## Architecture / Data Flow

```
Browser (index.html)
   |  POST /api/scan | /api/gemini | /api/ringkas | /api/latihan
   v
Cloudflare Pages Function (_worker.js)
   |-- validasi input + pembatas permintaan (KV CACHE atau memori)
   |-- chatbot : cache jawaban (KV) -> Workers AI -> cadangan Gemini
   |-- scan / ringkas / latihan : Gemini API (kunci disimpan sebagai secret)
   v
Respons JSON atau teks streaming -> ditampilkan di browser

Data buku kas: localStorage di browser pengguna.
```

## Installation / Run Instructions

Prasyarat: Node.js dan akun Google AI Studio (untuk API key Gemini).

```bash
# 1. salin contoh pengaturan lalu isi nilainya (jangan di-commit)
cp .env.example .dev.vars

# 2. jalankan lokal
npx wrangler pages dev .
```

Buka alamat yang ditampilkan wrangler (biasanya http://localhost:8788).

**Deploy ke Cloudflare Pages:** hubungkan repository, tanpa build command, direktori output = akar repo. Lalu di *Settings → Variables and Secrets* isi `GEMINI_API_KEY` sebagai **secret**. Opsional: binding **AI** (Workers AI) dan binding KV bernama **CACHE**.

Variabel yang dikenali ada di `.env.example`. Hanya `GEMINI_API_KEY` yang wajib.

## Deployment URL

https://cintarupiah.pages.dev

## Demo Account

Tidak diperlukan, aplikasi dapat dipakai tanpa login.

Catatan untuk penguji: scanner dibatasi beberapa permintaan per jam per perangkat untuk menjaga kuota. Tombol contoh hasil (mode demo) diberi label jelas dan tidak memakai AI.

## Known Limitations

- Scanner hanya menganalisis **foto**, jadi hanya indikasi awal. Keaslian uang tetap harus diperiksa langsung dengan 3D.
- Pengenalan pecahan oleh AI bisa keliru pada foto buram, miring, atau terpotong; pembacaan suara adalah perkiraan.
- Kuota Gemini bersifat terbatas; saat penuh, aplikasi menampilkan pesan dan chatbot menyediakan jawaban cadangan.
- Chatbot dapat keliru; informasi seperti jadwal, syarat, dan alamat penukaran harus dicek di kanal resmi Bank Indonesia.
- Buku kas tersimpan di browser yang dipakai; data hilang bila data situs dihapus atau berganti perangkat.
- Tidak ada akun pengguna dan tidak ada database server.

## Privacy

- Foto yang dianalisis dikirim ke Google Gemini API untuk diproses; aplikasi tidak menyimpannya.
- Teks pertanyaan chatbot dikirim ke penyedia AI. Jawaban untuk pertanyaan tanpa riwayat dapat disimpan di cache selama 24 jam, tanpa identitas pengguna.
- Pembatas permintaan memakai alamat IP sebagai bagian dari kunci sementara (1 jam).
- Jangan mengunggah foto yang memuat data pribadi.

## Third-party Resources

Lihat juga `AI_TECHNOLOGY_DISCLOSURE.md`.

- Tailwind CSS (CDN), Font Awesome 6.4.0, Google Fonts (Plus Jakarta Sans), canvas-confetti 1.6.0
- Google Gemini API, Cloudflare Pages / Workers AI / KV
- Konten edukasi mengacu pada materi publik kampanye Cinta Bangga Paham Rupiah (Bank Indonesia). Nama dan istilah tersebut milik pemiliknya masing-masing.

## Team Members

- [Nama Ketua] , [peran]
- [Nama Anggota 2] , [peran]
- [Nama Anggota 3] , [peran]

Asal instansi: SD No. 2 Tuban
