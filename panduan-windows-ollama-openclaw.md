# Panduan Lengkap: Ollama + OpenClaw + Qwen di Windows

> Panduan ini dibuat berdasarkan pengalaman instalasi nyata di Windows dengan spesifikasi:
> **Intel Core i7-12700F | RAM 16 GB | NVIDIA GTX 1650 (4 GB VRAM) | Storage 1.14 TB**
> Termasuk solusi error-error yang ditemui selama proses instalasi.

---

## Daftar Isi

1. [Analisis Spesifikasi & Rekomendasi Model](#1-analisis-spesifikasi--rekomendasi-model)
2. [Install Driver NVIDIA](#2-install-driver-nvidia)
3. [Install Ollama](#3-install-ollama)
4. [Download Model Qwen](#4-download-model-qwen)
5. [Install Node.js](#5-install-nodejs) ← *wajib sebelum OpenClaw*
6. [Install Git](#6-install-git) ← *wajib sebelum OpenClaw*
7. [Install & Jalankan OpenClaw](#7-install--jalankan-openclaw)
8. [Navigasi OpenClaw Dashboard](#8-navigasi-openclaw-dashboard)
9. [Upload Dokumen & Mulai Chat](#9-upload-dokumen--mulai-chat)
10. [Menjalankan Ulang Setelah Restart PC](#10-menjalankan-ulang-setelah-restart-pc)
11. [Troubleshooting & Error yang Pernah Ditemui](#11-troubleshooting--error-yang-pernah-ditemui)
12. [Ringkasan Perintah Penting](#12-ringkasan-perintah-penting)

---

## 1. Analisis Spesifikasi & Rekomendasi Model

| Komponen | Spesifikasi | Penilaian |
|----------|-------------|-----------|
| CPU | Intel Core i7-12700F (12 core, 12th Gen) | ✅ Sangat baik |
| RAM | 16 GB | ✅ Cukup untuk model 7B |
| GPU | NVIDIA GTX 1650 (4 GB VRAM) | ⚠️ Terbatas — mode hybrid GPU+CPU |
| Storage | 1.14 TB (~960 GB kosong) | ✅ Sangat lega |
| OS | Windows 10/11 | ✅ Didukung penuh |

### Rekomendasi Model Qwen untuk spesifikasi ini

| Model | Ukuran | RAM Dibutuhkan | Cocok? | Kecepatan |
|-------|--------|----------------|--------|-----------|
| `qwen2.5:3b` | ~2 GB | 8 GB RAM | ✅ Full GPU, sangat cepat | ⚡⚡⚡ |
| `qwen2.5:7b` | ~4.4 GB | 8 GB RAM | ✅ **Rekomendasi** — hybrid GPU+CPU | ⚡⚡ |
| `qwen2.5:14b` | ~9 GB | 16 GB RAM | ⚠️ Bisa, tapi lambat (full CPU) | ⚡ |

**→ Pilihan terbaik: `qwen2.5:7b`**

GTX 1650 punya VRAM 4 GB — tidak cukup menampung model 7B sepenuhnya (~4.4 GB). Ollama otomatis pakai mode **hybrid**: sebagian layer di GPU, sisanya di RAM 16 GB. Hasilnya lebih cepat dari full-CPU dan kualitas jawaban jauh lebih baik dari model 3B. Qwen juga unggul untuk teks bahasa Indonesia dibanding Llama.

---

## 2. Install Driver NVIDIA

Pastikan driver GPU terbaru agar Ollama bisa memanfaatkan GTX 1650.

1. Buka: **https://www.nvidia.com/drivers**
2. Isi form pencarian:
   - Product Type: `GeForce`
   - Product Series: `GeForce 16 Series`
   - Product: `GeForce GTX 1650`
   - Operating System: `Windows 11` (atau 10 sesuai OS Anda)
   - Download Type: `Game Ready Driver`
3. Klik **Search** → **Download** → jalankan file `.exe`
4. **Restart komputer** setelah selesai

> **Cek versi driver saat ini:** Device Manager → Display Adapters → klik kanan GTX 1650 → Properties → tab Driver

---

## 3. Install Ollama

### Download & Install

1. Buka: **https://ollama.com/download/windows**
2. Klik **Download for Windows** → file `OllamaSetup.exe` (~10-15 MB) akan terdownload
3. Jalankan file tersebut
4. Jika muncul dialog _"Windows protected your PC"_ → klik **More info** → **Run anyway**
5. Klik **Install**, tunggu 1-2 menit hingga selesai
6. Setelah selesai, Ollama otomatis berjalan di background

> **Catatan:** Setelah install, Ollama membuka **jendela GUI desktop** dengan tampilan chat bawaan — ini normal. Ollama versi terbaru sudah include interface chat sendiri. Anda bisa langsung chat dari sini, atau lanjutkan install OpenClaw untuk fitur yang lebih lengkap.

### Verifikasi

Buka PowerShell (`Windows + R` → ketik `powershell` → Enter):

```powershell
ollama --version
```

Harus muncul nomor versi, contoh: `ollama version 0.5.x`

---

## 4. Download Model Qwen

Di PowerShell, ketik:

```powershell
ollama pull qwen2.5:7b
```

Progress download akan terlihat seperti ini:
```
pulling manifest
pulling 8b4481071afe... 100% ▕████████████████▏ 4.4 GB
verifying sha256 digest
writing manifest
success
```

> **Estimasi waktu download:**
> - 10 Mbps → ~60-70 menit
> - 20 Mbps → ~25-35 menit
> - 50 Mbps → ~10-15 menit
> - 100 Mbps → ~5-8 menit
>
> File model tersimpan di: `C:\Users\NamaUser\.ollama\models`

> **Catatan:** Jika Anda langsung chat via GUI Ollama tanpa pull dulu, model akan otomatis terdownload saat pesan pertama dikirim. Progress bar download akan muncul di jendela chat — ini normal, tunggu saja hingga selesai.

### Test model berjalan

```powershell
ollama run qwen2.5:7b
```

Tunggu hingga muncul `>>>`, lalu ketik:
```
Halo, perkenalkan dirimu dalam bahasa Indonesia
```

Model akan merespons dalam bahasa Indonesia. Ketik `/bye` atau tekan `Ctrl+D` untuk keluar.

---

## 5. Install Node.js

> ⚠️ **Wajib dilakukan sebelum install OpenClaw.**
>
> Jika dilewati, akan muncul error saat menjalankan `ollama launch openclaw`:
> ```
> Error: openclaw is not installed and npm was not found
> Install Node.js first: https://nodejs.org/
> ```

### Download & Install

1. Buka: **https://nodejs.org/**
2. Klik tombol **LTS** (versi stabil, disarankan) → download file `.msi`
3. Jalankan installer, klik **Next** terus hingga **Finish** (semua setting default sudah benar)
4. Selama proses install, mungkin muncul jendela hitam dengan banyak baris teks warning dari **Visual Studio Installer** — **abaikan saja**, itu tidak ada hubungannya dengan Node.js (lihat penjelasan di bagian Troubleshooting)

### Verifikasi

**Tutup PowerShell lama**, buka PowerShell **baru**, lalu:

```powershell
node --version
```

Harus muncul nomor versi, contoh: `v24.x.x`

---

## 6. Install Git

> ⚠️ **Wajib dilakukan sebelum install OpenClaw.**
>
> Jika dilewati, akan muncul error:
> ```
> npm error syscall spawn git
> npm error path git
> npm error errno -4058
> Error: failed to install openclaw: exit status 0xfffff026
> ```

### Download & Install

1. Buka: **https://git-scm.com/download/win**
2. Download otomatis dimulai → file `Git-x.x.x-64-bit.exe`
3. Jalankan installer, klik **Next** terus hingga **Install** (semua setting default sudah benar)
4. Klik **Finish**

### Verifikasi

**Tutup PowerShell lama**, buka PowerShell **baru**, lalu:

```powershell
git --version
```

Harus muncul: `git version 2.x.x`

---

## 7. Install & Jalankan OpenClaw

> ⚠️ **Harus menggunakan PowerShell sebagai Administrator.**
>
> Jika menggunakan PowerShell biasa, akan muncul error:
> ```
> Gateway service install failed: Error: schtasks create failed: ERROR: Access is denied.
> Error: openclaw onboarding failed: exit status 1
> ```

### Buka PowerShell sebagai Administrator

1. Klik tombol **Start**
2. Ketik `powershell`
3. Klik kanan **Windows PowerShell** → pilih **"Run as administrator"**
4. Klik **Yes** pada dialog konfirmasi UAC
5. Pastikan judul jendela menampilkan **"Administrator: Windows PowerShell"**

### Jalankan OpenClaw

```powershell
ollama launch openclaw
```

Ollama akan otomatis download dan install OpenClaw. Setelah selesai, muncul menu pemilihan model interaktif:

```
Select models for OpenClaw: Type to filter...

  Recommended
    glm-4.7-flash       Reasoning and code generation locally, ~25GB (not downloaded)
    qwen3:8b            Efficient all-purpose assistant, ~11GB (not downloaded)
    minimax-m2.5:cloud  (not downloaded)
    glm-5:cloud         (not downloaded)
    kimi-k2.5:cloud     (not downloaded)

  More
  ► qwen2.5:7b          ← PILIH INI
```

### Pilih model

Gunakan tombol **↓ (panah bawah)** untuk navigasi ke `qwen2.5:7b` di bagian **More**, lalu tekan **Enter**.

> **Jangan pilih model lain** karena:
> - `glm-4.7-flash` → ~25 GB, terlalu besar untuk RAM 16 GB
> - `qwen3:8b` → ~11 GB, butuh RAM lebih besar
> - Model bertanda `:cloud` → berbasis cloud API, bukan lokal

### Tanda OpenClaw berhasil berjalan

```
OpenClaw 2026.x.x — Your personal assistant...

The OpenClaw gateway is running in the background.
Stop it with: openclaw gateway stop

Open the Web UI:
  http://localhost:18789/#token=ollama

agent main | session main | ollama/qwen2.5:7b | tokens ?/33k
connected | idle
```

### Buka di browser

Buka browser (Chrome/Edge/Firefox), ketik di address bar:

```
http://localhost:18789/#token=ollama
```

Tampilan **OpenClaw Gateway Dashboard** akan muncul dengan status **Health: OK** di pojok kanan atas.

> ⚠️ **Jangan tutup jendela PowerShell** selama menggunakan OpenClaw. Menutup PowerShell akan menghentikan service OpenClaw.

---

## 8. Navigasi OpenClaw Dashboard

Setelah berhasil membuka dashboard, berikut penjelasan menu di sidebar kiri:

### Bagian Chat
| Menu | Fungsi |
|------|--------|
| **Chat** | Chat langsung dengan AI — gunakan ini untuk tanya-jawab dan upload dokumen |

### Bagian Control
| Menu | Fungsi |
|------|--------|
| **Overview** | Ringkasan status sistem dan model |
| **Channels** | Hubungkan ke Telegram, WhatsApp, Slack, dll |
| **Instances** | Kelola instance model yang sedang berjalan |
| **Sessions** | Riwayat semua sesi chat |
| **Usage** | Statistik penggunaan token |
| **Cron Jobs** | Jadwalkan tugas otomatis |

### Bagian Agent
| Menu | Fungsi |
|------|--------|
| **Agents** | Buat AI agent dengan instruksi/persona khusus |
| **Skills** | Tambah kemampuan ke agent (baca dokumen, browsing, dll) |
| **Nodes** | Kelola koneksi antar agent/node |

### Bagian Settings
| Menu | Fungsi |
|------|--------|
| **Config** | Pengaturan utama (model, API, koneksi) |
| **Debug** | Log dan informasi debug |

---

## 9. Upload Dokumen & Mulai Chat

### Cara mulai chat

1. Klik menu **Chat** di sidebar kiri
2. Klik kolom **Message** di bagian bawah halaman
3. Ketik pertanyaan dan tekan **Enter** (atau klik tombol **Send**)

### Upload dokumen ke chat

Klik tombol **"+"** di sebelah kiri kolom Message untuk melampirkan file. Format yang didukung:
- **PDF** — laporan, kontrak, dokumen legal
- **DOCX** — dokumen Word
- **TXT** — teks biasa
- **Gambar** — screenshot, foto dokumen

### Contoh prompt untuk dokumen

```
Ringkaskan isi dokumen ini dalam 5 poin utama
```
```
Sebutkan semua tanggal dan deadline yang disebutkan dalam dokumen
```
```
Siapa saja pihak yang terlibat dalam kontrak ini?
```
```
Apa risiko yang perlu diperhatikan dari dokumen ini?
```
```
Buat tabel ringkasan dari semua item yang ada di laporan ini
```

### Tips membuat prompt yang efektif

| ❌ Kurang efektif | ✅ Lebih efektif |
|---|---|
| `ringkas dong` | `Ringkaskan laporan ini dalam 3 paragraf, fokus pada kesimpulan utama` |
| `apa isinya?` | `Sebutkan 5 poin penting dari dokumen ini dalam bahasa Indonesia` |
| `cari info gaji` | `Cari semua informasi terkait kompensasi dan tunjangan yang disebutkan dalam dokumen` |

### Perkiraan kecepatan respons di spesifikasi ini

| Jenis respons | Estimasi waktu |
|---------------|----------------|
| Jawaban pendek (1-2 kalimat) | 3-8 detik |
| Jawaban menengah (1 paragraf) | 10-20 detik |
| Rangkuman dokumen panjang | 30-60 detik |

---

## 10. Menjalankan Ulang Setelah Restart PC

Ollama otomatis berjalan di background saat Windows startup (ikon llama di System Tray pojok kanan bawah). Namun **OpenClaw perlu dijalankan manual** setiap kali ingin digunakan.

### Langkah setiap kali ingin menggunakan OpenClaw

1. Klik **Start** → ketik `powershell` → klik kanan → **Run as administrator**
2. Ketik:
   ```powershell
   ollama launch openclaw
   ```
3. Pilih `qwen2.5:7b` jika diminta memilih model
4. Buka browser → akses `http://localhost:18789/#token=ollama`

### Jika Ollama tidak berjalan otomatis

Jalankan manual di PowerShell (tidak perlu Administrator untuk ini):
```powershell
ollama serve
```
Biarkan jendela ini tetap terbuka di background.

---

## 11. Troubleshooting & Error yang Pernah Ditemui

### ❌ Error: `openclaw is not installed and npm was not found`

**Pesan lengkap:**
```
Error: openclaw is not installed and npm was not found
Install Node.js first: https://nodejs.org/
Then rerun: ollama launch and select OpenClaw
```

**Penyebab:** Node.js belum terinstall di sistem.

**Solusi:**
1. Install Node.js dari https://nodejs.org/ (pilih versi LTS)
2. Tutup PowerShell lama, buka PowerShell **baru**
3. Verifikasi: `node --version` → harus muncul `v2x.x.x`
4. Jalankan ulang: `ollama launch openclaw`

---

### ❌ Error: `npm error syscall spawn git`

**Pesan lengkap:**
```
npm error code ENOENT
npm error syscall spawn git
npm error path git
npm error errno -4058
npm error enoent An unknown git error occurred
Error: failed to install openclaw: exit status 0xfffff026
```

**Penyebab:** Git belum terinstall di sistem.

**Solusi:**
1. Install Git dari https://git-scm.com/download/win
2. Tutup PowerShell lama, buka PowerShell **baru**
3. Verifikasi: `git --version` → harus muncul `git version 2.x.x`
4. Jalankan ulang: `ollama launch openclaw`

---

### ❌ Error: `Gateway service install failed: Access is denied`

**Pesan lengkap:**
```
Gateway service install failed: Error: schtasks create failed: ERROR: Access is denied.
Run PowerShell as Administrator or rerun without installing the daemon.
Tip: rerun from an elevated PowerShell (Start → type PowerShell → right-click → Run as administrator)
Error: openclaw onboarding failed: exit status 1
```

**Penyebab:** PowerShell tidak dijalankan sebagai Administrator.

**Solusi:**
1. Tutup PowerShell yang ada
2. Klik Start → ketik `powershell`
3. Klik kanan **Windows PowerShell** → **Run as administrator** → klik **Yes**
4. Pastikan judul jendela bertuliskan **"Administrator: Windows PowerShell"**
5. Jalankan ulang: `ollama launch openclaw`

---

### ⚠️ Warning/Error saat install Node.js (Visual Studio Installer)

**Pesan yang muncul:**
```
Warning: [55d0:0011] Didn't find any channel feed.
WebClient error 'RequestCanceled' with 'https://aka.ms/vs/channels'
Failed to get the HttpWebResponse...
[55d0:000d] Authenticode verification returned 0x00000000
```

**Penyebab:** Visual Studio Installer mencoba mengecek update ke server Microsoft tapi koneksi gagal/timeout. **Tidak ada hubungannya dengan Node.js.**

**Solusi:** Abaikan sepenuhnya. Node.js tetap berhasil terinstall. Lanjutkan dengan membuka PowerShell baru dan verifikasi `node --version`.

---

### ❌ OpenClaw tidak bisa dibuka di browser

**Penyebab:** PowerShell dengan proses OpenClaw sudah ditutup, atau salah URL.

**Solusi:**
- Pastikan PowerShell Administrator dengan `ollama launch openclaw` **masih terbuka dan berjalan**
- Gunakan URL yang benar: `http://localhost:18789/#token=ollama`
- Jangan tutup jendela PowerShell selama menggunakan OpenClaw

---

### ❌ `ollama` tidak dikenali sebagai perintah

**Pesan:**
```
ollama : The term 'ollama' is not recognized as the name of a cmdlet...
```

**Solusi:** Tutup semua jendela PowerShell dan buka yang baru. Jika masih error, restart komputer.

---

### ⚠️ Model sangat lambat / GPU tidak terdeteksi

**Solusi:**
1. Pastikan driver NVIDIA sudah terbaru (lihat bagian 2)
2. Buka Task Manager (`Ctrl+Shift+Esc`) → tab **Performance** → pilih **GPU** → pastikan ada aktivitas saat model berjalan
3. Jalankan perintah ini untuk cek penggunaan VRAM:
   ```powershell
   ollama ps
   ```
4. Jika GPU tidak terdeteksi, coba set variabel environment sebelum `ollama serve`:
   ```powershell
   $env:CUDA_VISIBLE_DEVICES=0
   ollama serve
   ```

---

### ⚠️ Komputer berat saat model berjalan

Ini normal — model 7B memang membutuhkan resource yang signifikan. Tips:
- Tutup aplikasi berat lain (game, video editor, browser dengan banyak tab)
- Atau gunakan model lebih kecil: `ollama pull qwen2.5:3b`

---

## 12. Ringkasan Perintah Penting

| Perintah | Fungsi |
|----------|--------|
| `ollama --version` | Cek versi Ollama |
| `ollama pull qwen2.5:7b` | Download model Qwen 7B |
| `ollama run qwen2.5:7b` | Chat langsung via terminal |
| `ollama launch openclaw` | Install/jalankan OpenClaw *(gunakan PowerShell Admin)* |
| `ollama list` | Lihat semua model yang terinstall |
| `ollama ps` | Cek model aktif + penggunaan VRAM |
| `ollama serve` | Jalankan Ollama service manual |
| `ollama rm qwen2.5:7b` | Hapus model (bebaskan storage) |
| `node --version` | Verifikasi Node.js terinstall |
| `git --version` | Verifikasi Git terinstall |

---

## Struktur Sistem Akhir

```
Windows PC
│
├── Ollama            (background service, port 11434)
│   └── qwen2.5:7b   (~4.4 GB di C:\Users\PC\.ollama\models)
│
├── OpenClaw          (dijalankan via PowerShell Admin)
│   └── Web UI        http://localhost:18789/#token=ollama
│       ├── Chat      (tanya-jawab + upload dokumen)
│       ├── Agents    (AI agent dengan instruksi khusus)
│       ├── Skills    (kemampuan tambahan)
│       └── Channels  (integrasi Telegram, WhatsApp, dll)
│
├── Dependencies
│   ├── Node.js v24+  (wajib untuk OpenClaw)
│   └── Git 2.x       (wajib untuk OpenClaw)
│
└── Data lokal        C:\Users\PC\.openclaw\
```

**Semua berjalan 100% offline. Tidak ada data yang dikirim keluar dari komputer Anda.**

---

## Checklist Instalasi

- [ ] Driver NVIDIA terbaru sudah terinstall
- [ ] Ollama terinstall → `ollama --version` ✓
- [ ] Model `qwen2.5:7b` terdownload → `ollama list` ✓
- [ ] Node.js terinstall → `node --version` ✓
- [ ] Git terinstall → `git --version` ✓
- [ ] OpenClaw berhasil dijalankan via PowerShell Administrator
- [ ] Dashboard terbuka di `http://localhost:18789/#token=ollama`
- [ ] Status **Health: OK** muncul di pojok kanan atas dashboard
- [ ] Berhasil chat dengan model Qwen dalam bahasa Indonesia ✓

---

*Dibuat berdasarkan pengalaman instalasi nyata di Windows — termasuk semua error yang ditemui beserta solusinya.*
