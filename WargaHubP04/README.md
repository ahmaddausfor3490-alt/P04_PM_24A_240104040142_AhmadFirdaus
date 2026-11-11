# WargaHub P04 - UX, RecyclerView, Fragment, dan Navigasi

Proyek latihan Praktikum 04 Pemrograman Mobile Native Android berbasis Kotlin, XML Views, ViewBinding, dan Navigation Component.

## Identitas Mahasiswa
- **Nama**: Ahmad Firdaus
- **NIM**: 240104040142
- **Kelas**: 24A
- **Mata Kuliah**: Praktikum Pemrograman Mobile (PTI23043) - 1 SKS
- **Dosen Pengampu**: Muhayat, M.IT

---

## Lingkungan Pengembangan & Spesifikasi Alat
- **Sistem Operasi**: Windows 11 Home Single Language (64-bit)
- **IDE**: Android Studio
- **Bahasa Pemrograman**: Kotlin 2.3.21
- **Gradle Version**: Gradle 9.6.0
- **Android Gradle Plugin (AGP)**: 9.4.1
- **Kompilasi SDK (compileSdk)**: 37 (mendukung Android 15 & 16 APIs)
- **Minimum SDK (minSdk)**: 26 (Android 8.0 Oreo)
- **Target SDK (targetSdk)**: 36 / 37
- **Perangkat Uji / Emulator**: Android Emulator `Medium_Phone_API_37.0` (Android API 37, x86_64, resolusi 1080x2400)

---

## Ringkasan Fitur Aplikasi
1. **Satu Activity, Tiga Fragment**:
   - `MainActivity`: Wadah utama berisikan Toolbar (`setupActionBarWithNavController`) dan `FragmentContainerView` terhubung ke `nav_graph.xml`.
   - `HomeFragment`: Layar beranda sambutan, ringkasan fitur, dan tombol membuka daftar laporan.
   - `ListFragment`: Menampilkan daftar 15 laporan fiktif menggunakan `RecyclerView` vertikal dengan `ListAdapter` dan `DiffUtil`. Dilengkapi tombol simulasi daftar kosong dan pemulihan data contoh.
   - `DetailFragment`: Menampilkan rincian laporan lengkap berdasarkan argumen ID (`Long`). Menyediakan tombol kembali ke daftar serta penanganan aman (*fallback*) bila ID tidak ditemukan.
2. **Pengalaman Pengguna (UX) Responsif**:
   - Judul toolbar berubah otomatis sesuai tujuan navigasi.
   - Tombol kembali (Up navigasi & tombol fisik/gesture sistem) menggunakan `navigateUp()` sehingga tidak menduplikasi back stack atau memicu kebocoran memori.
   - Pemeriksaan `nav.currentDestination` sebelum navigasi untuk mencegah crash akibat *double tap*.
   - Manajemen siklus hidup `ViewBinding` aman (`_binding = null` pada `onDestroyView`).
   - Tampilan ramah aksesibilitas (*TalkBack*) melalui `accessibilityLiveRegion` dan `contentDescription` menyeluruh pada baris item.
   - Dukungan orientasi layar (Portrait & Landscape) dengan retensi state mode kosong via `onSaveInstanceState`.
3. **Batasan Data Contoh**:
   - Data laporan yang digunakan adalah 15 data fiktif lokal dalam objek Kotlin `LaporanContoh`.
   - Data bersifat read-only untuk latihan navigasi dan penyajian daftar; belum terintegrasi dengan database SQLite/Room maupun REST API backend.

---

## Cara Membuka dan Menjalankan Proyek
1. Buka **Android Studio**.
2. Pilih menu **File -> Open...**, lalu arahkan ke direktori `WargaHubP04`.
3. Tunggu hingga proses **Gradle Sync** selesai secara otomatis.
4. Pilih target perangkat (*Emulator* atau perangkat fisik dengan USB Debugging aktif).
5. Klik ikon **Run 'app'** (tombol hijau) atau tekan kombinasi tombol `Shift + F10`.
6. Alternatif build melalui terminal:
   ```powershell
   .\gradlew.bat assembleDebug
   ```
   File APK hasil build berada di:
   `app/build/outputs/apk/debug/app-debug.apk`

---

## Struktur Folder Pengumpulan
```text
P04_240104040142_AhmadFirdaus/
├── WargaHubP04/           # Source code proyek Android (bersih dari cache build)
├── README.md              # Dokumen panduan & identitas ini
├── docs/
│   ├── Laporan_P04.md     # Laporan praktikum lengkap (Markdown)
│   └── Laporan_P04.docx   # Laporan praktikum resmi (Microsoft Word)
└── screenshots/           # Bukti tangkapan layar pengujian aplikasi
    ├── 01_beranda.png
    ├── 02_daftar.png
    ├── 03_daftar_scroll.png
    ├── 04_rincian.png
    ├── 05_kosong.png
    ├── 06_landscape_atau_fontbesar.png
    └── 07_id_tidak_ditemukan.png
```
