# Log Penggunaan AI (Prompt Log) - Tugas Modul 06 Jadwalku

**Nama Proyek:** Konversi Desain Figma ke HTML5 Semantik Murni (JadwalKu)  
**Target Platform:** Lokal (VS Code / Browser)  

Dokumentasi ini mencatat riwayat instruksi (*prompt*) beserta proses evaluasi dan refaktorisasi kode yang dilakukan selama pengerjaan 8 file HTML utama untuk Tugas Modul 06.

---

## Ringkasan Riwayat Prompt & Iterasi Kode

### 1. Inisialisasi Proyek & Kerangka Halaman Beranda (`index.html`)
* **Prompt yang Diberikan:** 
  > *Meminta panduan langkah awal pengerjaan tugas modul 6, penyiapan folder aset, dan draf kode HTML semantik murni untuk Landing Page tanpa CSS eksternal/framework.*
* **Catatan Evaluasi & Refaktorisasi:**
  * Memastikan struktur landmark seperti `<header>`, `<nav>`, `<main>`, `<section>`, dan `<footer>` tersusun secara runut.
  * Menghindari penggunaan *div-soup* (pembungkus `<div>` berlebih yang tidak diperlukan) agar struktur dokumen lebih bersih.

### 2. Pembuatan Halaman Masuk & Simulasi Navigasi (`login.html`)
* **Prompt yang Diberikan:** 
  > *Membuat draf kode HTML untuk halaman login beserta form, label, input, pesan error validasi, serta instruksi agar tombol submit langsung mengarahkan ke halaman dashboard.*
* **Catatan Evaluasi & Refaktorisasi:**
  * Memastikan keterikatan formulir sempurna: atribut `for` pada `<label>` wajib berpasangan dengan atribut `id` pada `<input>` (misal: `for="email"` dan `id="email"`).
  * Mengarahkan atribut `action="dashboard.html"` pada tag `<form>` untuk mensimulasikan transisi antar-layar secara statis tanpa backend.

### 3. Pembuatan Dashboard Utama & Tabel Kalender (`dashboard.html`)
* **Prompt yang Diberikan:** 
  > *Membuat struktur markup untuk dashboard utama (sidebar navigasi, statistik, dan tabel kalender bulanan Oktober 2026), serta merapikan struktur tabel dan hierarki judul.*
* **Catatan Evaluasi & Refaktorisasi:**
  * Menyesuaikan elemen tabel kalender menggunakan tag semantik standar (`<table>`, `<caption>`, `<thead>`, `<th>` dengan `scope="col"`, dan `<tbody>`) tanpa menggunakan atribut gaya usang (seperti `border` atau `cellpadding`) demi kepatuhan terhadap W3C HTML5.
  * Menyelaraskan hierarki heading dari `<h1>` dokumen utama hingga `<h2>` dan `<h3>` pada setiap `<section>` dan `<article>`.

### 4. Pembuatan File Alur Layar Pendukung Lainnya
* **Prompt yang Diberikan:** 
  > *Membuat draf kode untuk sisa layar secara bertahap meliputi Dashboard Konflik (`dashboard-conflict.html`), Modal Detail Agenda (`detail-agenda.html`), Form Stepper (`form-stepper.html`), Konfirmasi Sukses (`success.html`), dan Layar Pemulihan Error (`error-recovery.html`).*
* **Catatan Evaluasi & Refaktorisasi:**
  * **Detail Agenda:** Menambahkan tag `<h1>` utama dokumen serta heading internal di dalam `<section>` untuk mengatasi peringatan *section lacks heading* dari W3C Validator.
  * **Form Stepper:** Memperbaiki elemen `<select>` dengan memberikan opsi pertama berupa nilai kosong (`value="" disabled selected`) agar valid saat atribut `required` diterapkan.
  * **Success Page:** Melengkapi dokumen dengan elemen `<h1>` utama agar memenuhi standar hierarki konten semantik.

---

## Catatan Pengujian & Validasi
* Seluruh berkas telah diuji secara lokal menggunakan **W3C Nu Validator** (`validator.w3.org/nu`) untuk memastikan tidak ada kesalahan sintaks, tag yang tidak ditutup, ataupun pelanggaran hierarki heading.
* Tangkapan layar hasil pengujian disimpan sebagai `validation.png` di dalam direktori utama proyek bersama folder aset (`assets/images/`).