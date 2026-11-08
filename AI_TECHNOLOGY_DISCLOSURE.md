# AI & Technology Disclosure

Produk: **CintaRupiah AI**
Tim: [Nama Tim], SD No. 2 Tuban
Versi yang dinilai: [commit hash / tag, mis. v1.0-submission]

Isian bertanda [ ] harus diperiksa dan dilengkapi oleh tim sebelum submit.

| Komponen | Isi Peserta |
|---|---|
| Nama Produk | CintaRupiah AI |
| Framework / Frontend | HTML, CSS, dan JavaScript tanpa framework (satu file `index.html`). Tailwind CSS lewat CDN |
| Backend / Runtime | Cloudflare Pages Functions, "advanced mode" (`_worker.js`, runtime Workers) |
| Database / Storage | Tidak ada database. Cloudflare KV (opsional) untuk cache jawaban chatbot dan pembatas permintaan; `localStorage` browser untuk buku kas |
| Hosting / Deployment | Cloudflare Pages, https://cintarupiah.pages.dev |
| AI Model / API / Method | Google Gemini API (scanner foto, ringkasan kas, latihan kasus, cadangan chatbot; nama model diatur lewat variabel `GEMINI_MODEL`, saat ini [isi nama model yang dipakai di Cloudflare]). Cloudflare Workers AI dengan model `@cf/meta/llama-3.3-70b-instruct-fp8-fast` untuk chatbot bila binding aktif. Browser Web Speech API (`speechSynthesis`) untuk suara |
| Peran AI dalam Produk | Menganalisis foto uang (perkiraan pecahan, kondisi, tanda terlihat); menjawab pertanyaan edukasi Rupiah dan QRIS; membuat ringkasan buku kas dari angka yang dihitung server; membuat skenario latihan. Tanpa AI, fungsi-fungsi ini tidak dapat berjalan |
| Dataset / Data Source | Tidak memakai dataset pelatihan sendiri. Konten edukasi (3D, 5 Jangan, CBP Rupiah, QRIS) mengacu pada materi publik Bank Indonesia. [isi sumber spesifik yang dipakai tim] |
| Open-source Libraries | Tailwind CSS (CDN), Font Awesome 6.4.0 (CDN), canvas-confetti 1.6.0 (CDN) |
| Template / Boilerplate | [isi: tidak ada / sebutkan bila ada] |
| AI Coding Assistant | Claude (Anthropic) dipakai sebagai asisten pemrograman untuk membantu menulis dan memperbaiki kode (termasuk `_worker.js`, `index.html`, dan dokumen ini). Tim menguji, memeriksa, dan bertanggung jawab atas kode yang dikumpulkan. [tambahkan alat lain bila ada, mis. ChatGPT, Copilot] |
| Pre-existing Components | [isi dengan jujur: bagian yang sudah ada sebelum lomba dan bagian yang dikembangkan untuk lomba] |
| Third-party Assets / Credits | Font Plus Jakarta Sans (Google Fonts); ikon Font Awesome. Tidak memakai logo atau gambar resmi Bank Indonesia. Aplikasi menyatakan bukan produk resmi Bank Indonesia |

## Data yang dikirim ke penyedia AI

- Scanner: satu foto uang yang dipilih pengguna.
- Chatbot: teks pertanyaan dan maksimal 6 pesan terakhir.
- Buku kas: tanggal serta nominal tunai dan QRIS yang dicatat pengguna (tanpa identitas).
- Latihan kasus: hanya nomor topik dari daftar tetap.

Aplikasi tidak menyimpan foto. Tidak ada akun pengguna.

## Mitigasi penyalahgunaan dan kesalahan AI

- Hasil scanner dinyatakan sebagai indikasi awal dan tidak pernah memastikan uang asli atau palsu.
- Chatbot diberi pegangan fakta dan diarahkan ke kanal resmi Bank Indonesia untuk hal yang tidak pasti.
- Keluaran AI divalidasi dan dibatasi di server; ditampilkan sebagai teks biasa.
- Ada pembatas permintaan per pengunjung dan pesan khusus saat kuota penuh.

## Pernyataan

Kami menyatakan bahwa informasi di atas benar dan lengkap. Kami bertanggung jawab atas penggunaan teknologi, source code, data, library, API, model AI, dan aset yang digunakan pada produk.
