# MotionFrame — GIF & WebP Frame Tool

MotionFrame adalah aplikasi web **single-file HTML** untuk melihat, mengatur, mengganti, dan mengekspor animasi **GIF** serta **animated WebP** berbasis frame.

Seluruh antarmuka utama, CSS, dan JavaScript berada di dalam satu file:

```text
index.html
```

Aplikasi berjalan client-side di browser dan dirancang agar workflow animasi sederhana: **VIEW untuk melihat animasi** dan **MODIFY untuk mengubah susunan/isi frame**.

---

## Konsep

MotionFrame memperlakukan GIF dan animated WebP sebagai kumpulan frame dengan metadata animasi. Setelah file di-import, setiap frame disimpan sebagai data gambar beserta delay-nya sehingga pengguna dapat memanipulasinya tanpa harus mengedit gambar seperti pada aplikasi image editor penuh.

### VIEW

Mode **VIEW** berfokus pada playback:

- Memutar animasi.
- Play / pause dengan kontrol playback berbasis Font Awesome.
- Scrub melalui timeline.
- Highlight frame aktif mengikuti playback.
- Playback menggunakan timing berbasis `requestAnimationFrame()` dan `performance.now()`.
- Saat pause, playback berhenti pada frame yang sedang tampil.
- Scroll halaman tidak seharusnya mengubah posisi playback atau menyebabkan thumbnail ikut melakukan scroll vertikal.

### MODIFY

Mode **MODIFY** berfokus pada frame:

- Pilih frame dari thumbnail/timeline.
- Move Left.
- Move Right.
- Duplicate frame.
- Delete frame.
- Replace Selected Frame.
- Mengubah delay setiap frame secara individual.
- **Add to all** untuk menyalin delay frame aktif ke semua frame sekali klik.
- Setelah `Add to all`, setiap frame tetap dapat disesuaikan lagi secara individual.

MotionFrame bukan image editor penuh. Fitur Replace digunakan untuk mengganti isi sebuah frame dengan gambar lain.

---

## Import

Format utama yang didukung:

- `.gif`
- `.webp`

Untuk GIF, aplikasi menggunakan parser/decompressor GIF yang berada di dalam file aplikasi dan tidak bergantung pada global decoder library.

Untuk animated WebP, container RIFF/`ANMF` diparsing di aplikasi dan payload frame WebP didecode menggunakan kemampuan WebP native browser.

### Replace frame

Gambar replacement boleh memiliki resolusi berbeda, tetapi **aspect ratio wajib sama** dengan canvas/frame animasi.

Contoh yang valid:

```text
Target:     800 × 600 px  → 4:3
Replacement: 1200 × 900 px → 4:3

✓ Accepted
```

Contoh yang ditolak:

```text
Target:     800 × 600 px  → 4:3
Replacement: 1024 × 1024 px → 1:1

✕ Aspect ratio mismatch
```

Setelah validasi berhasil, replacement disesuaikan ke resolusi animasi tanpa mengubah aspect ratio.

---

## Animation Information

Panel informasi animasi menampilkan antara lain:

- Format sumber.
- Jumlah frame.
- Resolusi canvas dalam pixel.
- Aspect ratio.
- Total duration.
- Ukuran file import asli.
- Frame aktif.
- **Average Frame Delay**.
- Loop behavior sumber.

### Average Frame Delay

Karena setiap frame dapat memiliki delay berbeda, informasi delay pada level animasi menggunakan nilai rata-rata seluruh frame.

Contoh:

```text
9 frame × 40 ms
1 frame × 100 ms

(360 + 100) / 10 = 46 ms

Average Frame Delay = 46 ms
```

Playback dan export **tetap menggunakan delay individual setiap frame**. Nilai average hanya merupakan informasi ringkas pada panel animasi.

---

## Frame Delay

Pada Selected Frame tersedia kontrol delay dalam milidetik.

- Delay dapat diatur per frame.
- Nilai minimum yang digunakan aplikasi adalah 10 ms.
- Tombol **Add to all** menyalin delay frame aktif ke semua frame pada saat tombol ditekan.
- Setelah itu frame tetap independen dan dapat diubah satu per satu.

Perubahan delay ikut memengaruhi total duration serta estimasi ukuran export.

---

## Playback

Playback dirancang untuk tetap halus terutama pada animasi dengan jumlah frame banyak.

Prinsipnya:

```text
requestAnimationFrame()
        ↓
cek elapsed time
        ↓
gunakan delay frame aktif
        ↓
pindah ke frame berikutnya
        ↓
draw canvas
```

Highlight thumbnail mengikuti frame aktif selama playback. Ketika playback dihentikan, highlight tetap berada pada frame yang sedang tampil.

---

## Export

MotionFrame memiliki dua kelompok export: **individual frame** dan **animated output**.

### Current Frame

Frame yang dipilih dapat diekspor sebagai:

- PNG.
- JPG.

Panel Selected Frame menampilkan resolusi frame dalam pixel dan menyediakan tombol export current frame.

### All Frames

Export seluruh frame menggunakan ZIP:

- **All PNG → ZIP**: satu file ZIP berisi semua frame PNG.
- **All JPG → ZIP**: satu file ZIP berisi semua frame JPG.

Nama file frame diberi nomor berurutan, misalnya:

```text
animation_frame_001.png
animation_frame_002.png
animation_frame_003.png
```

JPG menggunakan background putih untuk menangani area transparan.

### Estimated Size

Setiap tombol export menampilkan estimated size pada **baris di bawah tombol**.

Contoh:

```text
Export Current PNG
Est. ~184 KB
```

Estimasi bersifat perkiraan dan dapat berubah ketika frame, delay, format, atau kualitas WebP berubah.

---

## Animated GIF / WebP

Format animated output dipilih menggunakan **radio button**.

```text
◉ GIF
○ WebP
```

### GIF

GIF menggunakan plain/original export behavior tanpa kontrol visual quality tambahan.

Looping dapat dipilih:

- `1×`
- `2×`
- `3×`
- `Autoloop ∞`

### WebP

Ketika **WebP** dipilih, kontrol kualitas akan muncul:

```text
WebP Visual Quality
10% ─────────────── 100%
```

- `100%` = kualitas visual encoding tertinggi/original quality setting.
- Nilai lebih rendah menurunkan kualitas visual encoding.
- Resolusi pixel tetap sama.
- Contoh `50%`: canvas tetap `800 × 600 px`, tetapi kualitas encoding visual diturunkan.

Looping WebP memiliki pilihan yang sama:

- `1×`
- `2×`
- `3×`
- `Autoloop ∞`

### Export progress dan Cancel

Animated export menampilkan popup progress dengan status **Processed** sehingga pengguna mengetahui bahwa proses rendering sedang berlangsung.

Popup menyediakan tombol:

```text
Cancel Export
```

Cancel diperiksa selama proses encoding frame. Jika dibatalkan, proses export dihentikan dan file hasil tidak dibuat.

---

## Autosave Session

Workspace menggunakan **IndexedDB** untuk autosave lokal browser.

Session menyimpan informasi seperti:

- Mode VIEW/MODIFY.
- Nama file sumber.
- Ukuran file sumber.
- Format.
- Resolusi canvas.
- Loop source.
- Pilihan format animated export.
- Loop count export.
- WebP quality.
- Frame aktif.
- Current playback time.
- Seluruh frame hasil edit.
- Delay masing-masing frame.

Frame hasil edit disimpan sebagai data gambar lokal sehingga workspace dapat dipulihkan meskipun file sumber tidak sedang dipilih ulang.

Ketika halaman di-refresh, aplikasi mencoba memulihkan workspace terakhir secara otomatis.

---

## Reset Workspace

Tombol **Reset** tidak langsung menghapus workspace.

Aplikasi terlebih dahulu menampilkan modal konfirmasi di tengah halaman.

Setelah konfirmasi:

- Playback dihentikan.
- Semua frame dihapus.
- Canvas dikosongkan.
- Thumbnail/timeline dikosongkan.
- Metadata dikembalikan ke keadaan kosong.
- File input dikosongkan.
- IndexedDB workspace dibersihkan.
- Autosave lama dicegah agar tidak menulis kembali session setelah reset.

Hasil akhirnya adalah workspace kosong seperti saat aplikasi pertama dibuka.

---

## File Size Formatting

Ukuran file import menggunakan ukuran aktual dari `File.size`.

Format tampilan:

- Bytes untuk ukuran sangat kecil.
- KB ditampilkan sebagai angka normal hingga 3 digit, misalnya `382 KB`.
- MB menggunakan 2 angka desimal, misalnya `1.47 MB`.
- GB menggunakan 2 angka desimal jika diperlukan.

Contoh:

```text
382 KB
1.47 MB
12.83 MB
```

---

## Teknologi

MotionFrame menggunakan teknologi browser standar dan beberapa library open-source yang dimuat dari CDN.

### Browser APIs

- HTML5 Canvas.
- `CanvasRenderingContext2D`.
- `Canvas.toBlob()`.
- `createImageBitmap()`.
- `File` / `Blob` APIs.
- IndexedDB.
- `requestAnimationFrame()`.
- `performance.now()`.
- Dynamic `import()` untuk encoder GIF.

### External libraries

- **Font Awesome 6.7.2** — ikon UI/playback.
- **JSZip 3.10.1** — ZIP export seluruh frame.
- **gifenc 1.0.3** — animated GIF encoding.

Animated WebP output menggunakan native browser WebP encoding (`canvas.toBlob(..., "image/webp", quality)`) kemudian payload WebP frame dirangkai menjadi animated WebP container.

---

## Single-File Architecture

Tidak diperlukan backend untuk workflow utama.

```text
index.html
├── HTML structure
├── Internal CSS
└── Internal JavaScript
```

Data gambar diproses di sisi client/browser. Aplikasi tidak membutuhkan server API untuk membaca atau mengedit frame.

> Catatan: karena beberapa library UI/ZIP/GIF encoder dimuat dari CDN, koneksi internet diperlukan ketika dependency tersebut belum tersedia di cache browser. Decoder GIF dan pipeline import animated WebP tidak bergantung pada remote decoder service.

---

## Recommended Repository Structure

Untuk repository GitHub sederhana:

```text
motionframe/
├── index.html
├── README.md
└── .gitignore
```

Tidak diperlukan `package.json` untuk versi single-file saat ini karena dependency digunakan melalui CDN.

---

## Running Locally

Cara paling sederhana:

1. Download atau clone repository.
2. Buka `index.html` di browser modern.
3. Import file GIF atau animated WebP.

Untuk pengalaman browser yang lebih konsisten, terutama terkait module loading dan security policy browser, disarankan menjalankan static local server, misalnya melalui VS Code Live Server atau static HTTP server sederhana.

---

## GitHub Pages

Karena aplikasinya berupa static single-file web app, project ini cocok untuk GitHub Pages.

Struktur minimal:

```text
Repository root
└── index.html
```

Setelah GitHub Pages diaktifkan, halaman dapat dibuka langsung melalui URL Pages repository tersebut.

---

## Privacy

Workflow frame dan autosave dilakukan di browser pengguna. Aplikasi tidak memiliki backend upload untuk mengirim gambar ke server.

Namun, dependency eksternal seperti Font Awesome, JSZip, dan gifenc dimuat dari CDN saat diperlukan. Artinya browser dapat melakukan request ke domain CDN tersebut.

---

## Current Scope

MotionFrame saat ini berfokus pada:

**View → Decode → Frame management → Timing → Replace → Export**

Bukan pada:

- Photo retouching.
- Advanced drawing.
- Layer editor.
- Color grading penuh.
- Non-linear timeline editing.
- Audio editing.

Pendekatan tersebut menjaga aplikasi tetap ringan dan fokus pada pekerjaan frame animation.

---

## Future Expansion Ideas

Beberapa pengembangan yang tetap selaras dengan konsep:

- Undo / redo untuk perubahan frame.
- Batch replace frame.
- Drag-and-drop reorder langsung pada thumbnail.
- Better export progress detail.
- Import static PNG/JPG sebagai frame baru.
- Import sequence gambar sebagai animasi baru.
- Project/session export dan import.
- Offline bundling seluruh dependency untuk mode tanpa internet.

---

## Status

**Prototype / Active Development**

Fokus pengembangan saat ini adalah stabilitas import, playback, frame manipulation, autosave, dan export animated GIF/WebP.
