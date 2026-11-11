# LAPORAN PRAKTIKUM PEMROGRAMAN MOBILE
## MODUL 04: UX, RECYCLERVIEW, FRAGMENT, DAN NAVIGASI

---

### IDENTITAS PRAKTIKUM
- **Mata Kuliah**: Praktikum Pemrograman Mobile
- **Kode Mata Kuliah**: PTI23043 (1 SKS)
- **Pertemuan / Modul**: Pertemuan 4 / Praktikum 04
- **Dosen Pengampu**: Muhayat, M.IT
- **Nama Mahasiswa**: Ahmad Firdaus
- **NIM**: 240104040142
- **Kelas**: 24A
- **Program Studi**: Teknologi Informasi
- **Platform**: Native Android (Kotlin, XML Views, ViewBinding, Navigation Component)
- **Proyek Latihan**: WargaHubP04 (`id.antasari.p04.wargahub`)
- **Tanggal Pengerjaan**: 11 November 2026

---

## 1. PENDAHULUAN DAN TUJUAN

### 1.1 Latar Belakang
Pada pengembangan aplikasi mobile modern, menyajikan data berjumlah banyak secara dinamis dan efisien merupakan kebutuhan fundamental. Penggunaan komponen konvensional seperti penambahan banyak `TextView` di dalam `ScrollView` terbukti membebani memori dan menyebabkan *frame drop* (jank) karena seluruh komponen antarmuka harus diinisiasi sekaligus di memori. Sebagai solusinya, Android SDK menyediakan komponen `RecyclerView` yang menerapkan pola daur ulang *view* (*view recycling*) menggunakan `ViewHolder` dan `Adapter`.

Selain efisiensi penyajian daftar, aspek navigasi multi-layar memerlukan arsitektur yang terstruktur agar alur perpindahan antarlayar berjalan mulus, tidak memboroskan siklus hidup *Activity*, serta terbebas dari kebocoran memori (*memory leak*). Pendekatan *Single Activity Multiple Fragments* yang dipadukan dengan Jetpack Navigation Component menjadi standar industri untuk mencapai tujuan tersebut.

### 1.2 Capaian Pembelajaran (Sub-CPMK)
Praktikum ini merujuk pada **Sub-CPMK 2**: "Merancang dan menerapkan UI/UX yang responsif serta mudah digunakan". Melalui modul ini, mahasiswa diharapkan mampu:
1. Menyiapkan proyek Native Android berbasis Views dengan dependensi RecyclerView dan Navigation Component.
2. Mengimplementasikan penyajian daftar dinamis menggunakan `RecyclerView`, `ListAdapter`, dan `DiffUtil`.
3. Mengelola siklus hidup (*lifecycle*) Fragment dan ViewBinding secara aman guna mencegah *memory leak*.
4. Mengalirkan data antar-tujuan navigasi secara efisien menggunakan argumen bertipe primitif (`Long`).
5. Menyediakan umpan balik UX yang komprehensif, mencakup *empty state*, pesan *fallback*, navigasi *Up*/*Back*, serta dukungan aksesibilitas (*TalkBack*).

---

## 2. PERANCANGAN UI/UX DAN SKETSA ANTARMUKA

### 2.1 Alur Tugas Pengguna (User Flow)
Alur interaksi pengguna dirancang sederhana dan terarah:
```
[ Beranda (HomeFragment) ]
            │
            ▼ (Klik: "Lihat Daftar Laporan")
[ Daftar Laporan (ListFragment) ] ◄───────────────┐
      │                   ▲                      │
      │ (Klik Item)       │ (Klik Up / Back /    │ (Klik Up / Back)
      ▼                   │  Tombol Kembali)     │
[ Rincian Laporan (DetailFragment) ]             │
      │                                          │
      └──────────────────────────────────────────┘
```

Aplikasi WargaHub P04 terdiri dari satu `MainActivity` sebagai wadah (*host*) dan tiga `Fragment`:
1. **Layar Beranda (`HomeFragment`)**: Menyambut pengguna, memberikan keterangan bahwa data yang disajikan adalah data simulasi latihan, dan menyediakan tombol primer untuk masuk ke daftar laporan.
2. **Layar Daftar Laporan (`ListFragment`)**: Menampilkan 15 item laporan fiktif, indikator jumlah laporan yang aktif diumumkan ke pembaca layar, tombol simulasi daftar kosong ("Uji Daftar Kosong"), serta wadah pesan kosong (*empty container*) bila tidak ada data.
3. **Layar Rincian Laporan (`DetailFragment`)**: Menampilkan judul lengkap, kategori, status contoh, dan deskripsi masalah berdasarkan ID laporan yang dipilih. Menyediakan tombol "Kembali ke Daftar" serta proteksi bila ID tidak valid.

### 2.2 Sketsa Antarmuka (Wireframe)

#### A. Layar Beranda (`HomeFragment`)
```text
┌──────────────────────────────────────────┐
│ Beranda                                  │ <- Toolbar (Title dinamis)
├──────────────────────────────────────────┤
│                                          │
│ WargaHub                                 │ <- Judul Utama (Title Style, Navy)
│                                          │
│ Pelajari cara membaca laporan lingkungan │ <- Subjudul
│ melalui daftar dan rincian.              │
│                                          │
│ Data contoh untuk latihan. Bukan laporan │ <- Keterangan Simulasi
│ nyata dan belum memakai database.        │
│                                          │
│ ┌──────────────────────────────────────┐ │
│ │         Lihat Daftar Laporan         │ │ <- Tombol Aksi Primer
│ └──────────────────────────────────────┘ │
│                                          │
└──────────────────────────────────────────┘
```

#### B. Layar Daftar Laporan (`ListFragment`)
```text
┌──────────────────────────────────────────┐
│ ←  Daftar Laporan                        │ <- Toolbar dengan ikon Up
├──────────────────────────────────────────┤
│ Pilih satu laporan untuk melihat         │
│ rinciannya.                              │ <- Petunjuk Navigasi
│                                          │
│ 15 laporan contoh                        │ <- Textview Jumlah Data
│                                          │
│ ┌──────────────────────────────────────┐ │
│ │          Uji Daftar Kosong           │ │ <- Tombol Toggle Empty Mode
│ └──────────────────────────────────────┘ │
│                                          │
│ ┌──────────────────────────────────────┐ │
│ │ Lampu jalan padam di zona latihan 1  │ │ <- Item Baris 1 (Card/List item)
│ │ Lampu jalan padam                    │ │
│ │ Status contoh: Simulasi              │ │
│ ├──────────────────────────────────────┤ │
│ │ Sampah menumpuk di zona latihan 2    │ │ <- Item Baris 2
│ │ Sampah menumpuk                      │ │
│ │ Status contoh: Simulasi              │ │
│ ├──────────────────────────────────────┤ │
│ │ Drainase tersumbat di zona latihan 3 │ │ <- Item Baris 3
│ │ ...                                  │ │
└──────────────────────────────────────────┘
```

#### C. Layar Rincian Laporan (`DetailFragment`)
```text
┌──────────────────────────────────────────┐
│ ←  Rincian Laporan                       │ <- Toolbar dengan ikon Up
├──────────────────────────────────────────┤
│ Periksa kesesuaian rincian dengan        │
│ laporan yang dipilih.                    │ <- Teks Bantuan
│                                          │
│ Lampu jalan padam di zona latihan 13     │ <- Judul Laporan Terpilih
│                                          │
│ Kategori: Lampu jalan padam              │ <- Kategori
│ Status contoh: Simulasi                  │ <- Status
│                                          │
│ Contoh masalah di zona latihan 13. Data  │
│ fiktif ini dipakai untuk mempelajari     │ <- Deskripsi Masalah
│ daftar dan navigasi rincian laporan.     │
│                                          │
│ Data contoh untuk latihan. Bukan laporan │ <- Catatan Simulasi
│ nyata dan belum memakai database.        │
│                                          │
│ ┌──────────────────────────────────────┐ │
│ │          Kembali ke Daftar           │ │ <- Tombol Navigasi Up
│ └──────────────────────────────────────┘ │
└──────────────────────────────────────────┘
```

---

## 3. IMPLEMENTASI TEKNIS DAN ANALISIS ARSITEKTUR

### 3.1 RecyclerView, ListAdapter, dan DiffUtil
Pada implementasi daftar, digunakan `RecyclerView` yang dikombinasikan dengan `ListAdapter<Laporan, LaporanHolder>` serta kalkulasi perbedaan data menggunakan `DiffUtil.ItemCallback<Laporan>`.
- **`areItemsTheSame(a, b)`**: Memeriksa kesamaan identitas item berdasarkan atribut kunci yang stabil, yaitu `a.id == b.id`. Hal ini memastikan bahwa perubahan posisi baris tidak mengacaukan identitas data.
- **`areContentsTheSame(a, b)`**: Memeriksa apakah properti isi objek mengalami perubahan (`a == b`), sehingga `RecyclerView` hanya merender ulang (*re-bind*) baris yang benar-benar mengalami pembaruan konten.
- **`RecyclerView.ViewHolder`**: Berperan sebagai wadah yang memegang referensi ke hierarki view baris (`ItemLaporanBinding`). Dengan pola ini, pemanggilan mahal seperti `findViewById` tidak dilakukan berulang kali saat daftar digulir.
- **`LinearLayoutManager`**: Menentukan tata letak vertikal daftar serta mengendalikan pemanfaatan kembali (*recycling*) view yang terdorong keluar dari batas layar.

### 3.2 Siklus Hidup Fragment dan Manajemen ViewBinding
Fragment memiliki siklus hidup tampilan (*View Lifecycle*) yang terpisah dari siklus hidup Fragment itu sendiri. Ketika sebuah Fragment berpindah ke *back stack*, hierarki *View*-nya dihancurkan (`onDestroyView`), namun instansiasi Fragment tetap bertahan di memori.
- Untuk mengantisipasi kebocoran memori (*memory leak*), variabel `_binding` dibuat *nullable* (`FragmentXBinding? = null`).
- Inisialisasi binding dilakukan pada `onViewCreated()` menggunakan metode `FragmentXBinding.bind(view)`.
- Pada `onDestroyView()`, binding wajib dilepas dengan menyetel `_binding = null`.
- Pada `ListFragment`, referensi adapter pada `rvLaporan.adapter` juga disetel `null` pada `onDestroyView()` guna memutus rantai referensi memori antara RecyclerView, adapter, dan lambda callback.

### 3.3 Jetpack Navigation Component dan Navigasi Aman
Perpindahan antar-layar dikelola oleh `NavController` melalui deklarasi deklaratif di `nav_graph.xml`:
- **Pengiriman Argumen**: Alih-alih mengirimkan keseluruhan objek data melalui Bundle (yang memboroskan memori dan berisiko *TransactionTooLargeException*), aplikasi hanya melewatkan argumen ID berupa tipe data primitif `Long` (`bundleOf(DetailFragment.ARG_ID to laporan.id)`). `DetailFragment` kemudian mencari data rincian secara independen menggunakan `LaporanContoh.cari(id)`.
- **Pemeriksaan Destinasi Aktif**: Sebelum menjalankan `nav.navigate()`, diterapkan pengecekan `if (nav.currentDestination?.id == R.id.homeFragment)` dan `R.id.listFragment`. Hal ini mencegah terjadinya *crash* (*IllegalArgumentException: navigation destination unknown*) akibat pengguna melakukan ketukan ganda yang sangat cepat (*double tapping*).
- **Integrasi Toolbar**: `setupActionBarWithNavController(navController, appBarConfiguration)` pada `MainActivity` memastikan judul toolbar dan ikon panah navigasi *Up* sinkron secara otomatis dengan tujuan yang sedang aktif.

### 3.4 Penanganan Kondisi Khusus (Edge Cases)
1. **Mode Daftar Kosong (*Empty State*)**: Diimplementasikan menggunakan `FrameLayout` yang menampung `rvLaporan` dan `emptyContainer`. State mode kosong disimpan di `savedInstanceState` melalui `onSaveInstanceState` dengan kunci `"contoh_kosong"`. Ketika perangkat dirotasi atau dipulihkan dari *process death*, kondisi daftar kosong tetap dipertahankan.
2. **Penanganan ID Tidak Ditemukan (*Fallback UI*)**: Apabila `DetailFragment` menerima ID yang tidak ada pada sumber data (misalnya uji nilai `-99L`), aplikasi tidak mengalami *crash* karena tidak menggunakan operator *force unwrap* (`!!`). Sebagai gantinya, antarmuka beralih ke mode aman dengan menampilkan teks peringatan `"Laporan tidak ditemukan..."` dan menyembunyikan view lainnya (`View.GONE`).

---

## 4. MATRIKS HASIL PENGUJIAN AKTUAL (T01 - T16)

Seluruh pengujian dilakukan secara langsung pada emulator Android dengan spesifikasi:
- **Perangkat**: Android Emulator `Medium_Phone_API_37.0`
- **Sistem Operasi**: Android API 37 (Build Android 15/16 Preview, x86_64)
- **Orientasi**: Portrait dan Landscape
- **Tanggal Uji**: 11 November 2026
- **Commit/Versi**: v1.0-debug (`WargaHubP04`)

| ID Skenario | Kondisi Pengujian | Langkah & Aksi Uji | Hasil yang Diharapkan | Hasil Aktual yang Diamati | Status | Bukti Screenshot |
|:---:|:---|:---|:---|:---|:---:|:---|
| **T01** | Beranda | Buka aplikasi dari app launcher. | Satu layar Beranda tampil dengan judul "WargaHub", keterangan simulasi, dan tombol navigasi. | Layar Beranda tampil sempurna dengan toolbar "Beranda", teks sambutan, catatan simulasi, dan tombol "Lihat Daftar Laporan". | **LULUS** | `01_beranda.png` |
| **T02** | Daftar | Tekan tombol "Lihat Daftar Laporan". | Layar berganti ke Daftar Laporan; teks jumlah menunjukkan 15 laporan; baris item memuat judul, kategori, dan status. | Toolbar berubah menjadi "Daftar Laporan" dengan panah Up; tampil "15 laporan contoh" dan item 1–5 terlihat rapi. | **LULUS** | `02_daftar.png` |
| **T03** | Gulir (Scroll) | Gulir daftar ke zona latihan 10–15. | Daftar berputar lancar (*smooth*), item zona 10–15 muncul dan dapat diketuk. | Daftar bergulir mulus tanpa jank; item 12, 13, 14, 15 terlihat dan siap dipilih. | **LULUS** | `03_daftar_scroll.png` |
| **T04** | Rincian | Ketuk item "Lampu jalan padam di zona latihan 13". | Halaman rincian terbuka menampilkan data yang sesuai dengan ID item yang dipilih. | Toolbar berganti "Rincian Laporan", judul zona 13, kategori, status simulasi, dan deskripsi sesuai. | **LULUS** | `04_rincian.png` |
| **T05** | Tombol Kembali | Pada rincian, tekan tombol "Kembali ke Daftar". | Kembali ke layar daftar sebelumnya tanpa menumpuk instance daftar baru di back stack. | Berhasil kembali ke layar daftar; posisi daftar tetap terjaga; tidak ada duplikasi layar. | **LULUS** | Terverifikasi |
| **T06** | Up Toolbar | Tekan ikon panah Up pada toolbar di rincian lalu di daftar. | Rincian → Daftar → Beranda. | Alur navigasi Up bertingkat berfungsi dengan tepat: Rincian kembali ke Daftar, dan Daftar kembali ke Beranda. | **LULUS** | Terverifikasi |
| **T07** | Back Sistem | Jalankan alur Beranda → Daftar → Rincian, lalu tekan tombol Back sistem berkali-kali. | Urutan kembali masuk akal (Rincian → Daftar → Beranda → Keluar) tanpa crash. | Sistem back stack pop secara berurutan; tidak terjadi crash atau *white screen*. | **LULUS** | Terverifikasi |
| **T08** | Mode Kosong | Pada layar daftar, tekan tombol "Uji Daftar Kosong". | Jumlah menjadi 0; RecyclerView disembunyikan; pesan kosong dan tombol pemulihan muncul. | Teks berubah "0 laporan contoh", container kosong tampil dengan pesan petunjuk, tombol menjadi "Tampilkan Data Contoh". | **LULUS** | `05_kosong.png` |
| **T09** | Pulihkan Data | Pada kondisi kosong, tekan tombol "Tampilkan Data Contoh". | Jumlah kembali 15 laporan; daftar item muncul kembali secara utuh. | RecyclerView aktif kembali menampilkan 15 item; tombol kembali berlabel "Uji Daftar Kosong". | **LULUS** | Terverifikasi |
| **T10** | Rotasi Rincian | Putar perangkat ke landscape saat layar rincian terbuka. | Layar menyesuaikan orientasi; data rincian dan ID tidak hilang. | Konten rincian tetap utuh dan dapat digulir; toolbar mempertahankan judul "Rincian Laporan". | **LULUS** | Terverifikasi |
| **T11** | Rotasi Kosong | Aktifkan mode kosong lalu lakukan rotasi perangkat. | Mode kosong tetap bertahan; tombol tetap berlabel "Tampilkan Data Contoh". | Berkat `onSaveInstanceState`, state `contohKosong` tersimpan dan layar tetap dalam kondisi kosong pasca rotasi. | **LULUS** | Terverifikasi |
| **T12** | Landscape & Font | Uji antarmuka pada orientasi landscape. | Seluruh teks terbaca jelas; layout tidak terpotong (*clipped*); area konten dapat digulir. | Tampilan adaptif pada orientasi horizontal; padding dan insets system bar terkalkulasi secara tepat. | **LULUS** | `06_landscape_atau_fontbesar.png` |
| **T13** | Aksesibilitas (TalkBack) | Periksa atribut aksesibilitas dan pembacaan layar pada baris daftar. | Setiap baris dibaca sebagai satu kesatuan interaktif yang bermakna. | Atribut `contentDescription` pada baris memuat teks terpadu ("%1$s. Kategori %2$s. Status contoh %3$s. Buka rincian."). | **LULUS** | Terverifikasi |
| **T14** | Ketukan Cepat | Ketuk tombol navigasi dan item secara cepat berulang kali (*spam tap*). | Tidak memicu navigasi ganda atau crash aplikasi. | Pengecekan `nav.currentDestination?.id` secara efektif menolak event navigasi kedua saat transisi sedang berlangsung. | **LULUS** | Terverifikasi |
| **T15** | Mulai Ulang | Hentikan proses aplikasi (`force-stop`), lalu luncurkan ulang dari launcher. | Aplikasi terbuka di Beranda; 15 data contoh tetap tersedia saat daftar dibuka. | Aplikasi memulai siklus hidup baru; data fiktif lokal terbentuk kembali secara konsisten dari kode Kotlin. | **LULUS** | Terverifikasi |
| **T16** | ID Tidak Ditemukan | Ubah argumen pengiriman menjadi `-99L` pada `ListFragment`, build, dan pilih baris. | Rincian menampilkan pesan bahwa laporan tidak ditemukan; tombol kembali tetap berfungsi. | Tampil teks "Laporan tidak ditemukan...", view kategori/status disembunyikan, dan tombol "Kembali ke Daftar" tetap bekerja. | **LULUS** | `07_id_tidak_ditemukan.png` |

> *Catatan Pengujian T16*: Pengujian dilakukan secara terkontrol sementara dengan mengubah `bundleOf(DetailFragment.ARG_ID to -99L)` untuk memvalidasi *fallback UI*. Setelah bukti tangkapan layar `07_id_tidak_ditemukan.png` diperoleh, kode dikembalikan ke `bundleOf(DetailFragment.ARG_ID to laporan.id)` dan di-build ulang untuk pengumpulan akhir.

---

## 5. PENGUJIAN USABILITY (UJI TUGAS TEMAN)

Untuk mengevaluasi aspek kebergunaan (*usability*) antarmuka WargaHub P04, dilakukan pengujian tugas (*task-based usability test*) bersama seorang rekan penguji (*peer reviewer*).

### 5.1 Skenario dan Ketentuan Pengujian
- **Tugas yang Diberikan**: *"Temukan rincian laporan untuk zona latihan 10, lalu kembali ke daftar laporan."*
- **Kondisi**: Penguji tidak diberi petunjuk langkah maupun bantuan ketukan (*unassisted*).
- **Titik Awal Pengukuran**: Saat aplikasi terbuka di layar Beranda.
- **Titik Akhir Pengukuran**: Saat halaman rincian zona latihan 10 berhasil terbuka secara penuh.

### 5.2 Hasil Pengujian Usability
- **Status Tugas**: Berhasil (Lulus tanpa bantuan).
- **Waktu Penyelesaian**: 6,8 detik.
- **Jumlah Permintaan Bantuan**: 0 kali.
- **Skor Kemudahan (Skala 1–5)**: **5 / 5** (Sangat Mudah).
- **Komentar Penguji**:
  > *"Hierarki layarnya sangat jelas. Tombol 'Lihat Daftar Laporan' langsung terlihat di layar awal. Pada daftar, judul zona dicetak tebal sehingga memudahkan pemindaian cepat saat menggulir ke zona 10. Tombol 'Kembali ke Daftar' di bagian bawah rincian juga sangat intuitif."*

### 5.3 Temuan UX dan Tindak Lanjut
- **Temuan**: Pada layar daftar dengan 15 item, pengguna baru terkadang tidak menyadari adanya opsi uji daftar kosong karena tombol diletakkan di bawah jumlah data. Namun, untuk navigasi utama (melihat laporan dan membaca rincian), alur sangat intuitif dan minim hambatan kognitif.
- **Tindak Lanjut**: Mempertahankan pemisahan visual yang tegas antara kontrol simulasi latihan dengan daftar konten utama melalui pemberian margin dan gaya tombol berkontras tinggi (`@color/p04_navy` di atas `@color/p04_page`).

---

## 6. REFLEKSI PRAKTIKUM

Berikut adalah analisis dan jawaban mendalam atas pertanyaan refleksi yang terdapat pada bagian 9.4 Modul Praktikum 04:

#### 1. Mengapa RecyclerView lebih sesuai daripada menambahkan banyak TextView secara manual untuk daftar yang panjang?
`RecyclerView` bekerja dengan membatasi jumlah objek *View* yang dibuat di memori hanya sebatas yang muat di layar ditambah sedikit *buffer*. Saat pengguna menggulir daftar, *View* yang terdorong ke luar layar tidak dihancurkan, melainkan didaur ulang (*recycled*) untuk diisi ulang datanya (*binding*) dengan item baru yang masuk ke layar. Sebaliknya, menambahkan banyak `TextView` secara manual ke dalam `ScrollView` mengharuskan sistem mengalokasikan memori untuk seluruh elemen sekaligus saat layout dimuat. Untuk kumpulan data yang besar, pendekatan manual dapat memicu *OutOfMemoryError* (OOM), lag animasi yang parah, dan pemborosan konsumsi baterai.

#### 2. Apa perbedaan tanggung jawab Adapter, ViewHolder, dan LayoutManager?
- **`Adapter`**: Bertindak sebagai jembatan antara sumber data (*data source*) dan antarmuka *RecyclerView*. Bertanggung jawab menentukan jumlah item, membuat *ViewHolder* baru (`onCreateViewHolder`), dan mengikat data ke dalam *view* baris (`onBindViewHolder`).
- **`ViewHolder`**: Memegang referensi langsung ke elemen-elemen tampilan di setiap baris item (dalam hal ini menggunakan `ItemLaporanBinding`). Menghindari pencarian view berulang kali (`findViewById`) saat daur ulang berlangsung.
- **`LayoutManager`**: Mengontrol tata letak fisik dan orientasi penyusunan item pada layar (vertikal, horizontal, grid, atau *staggered*), serta mengukur dan mengatur kapan sebuah *View* harus didaur ulang saat proses *scrolling*.

#### 3. Mengapa binding Fragment dilepas saat onDestroyView, bukan menunggu Activity ditutup?
Siklus hidup tampilan (*View Lifecycle*) sebuah Fragment tidak selalu sejalan dengan siklus hidup instansiasi Fragment itu sendiri. Saat Fragment masuk ke dalam *back stack* (misalnya berpindah dari `ListFragment` ke `DetailFragment`), *View* milik `ListFragment` dihancurkan melalui `onDestroyView()`, namun objek `ListFragment` tetap hidup di memori. Jika properti binding tidak disetel menjadi `null`, referensi ke pohon *View* yang telah hancur akan tetap tertahan oleh objek Fragment. Kondisi ini menyebabkan kebocoran memori (*View memory leak*) yang signifikan.

#### 4. Mengapa argumen berisi ID lebih tepat daripada posisi baris?
Posisi baris (*index position*) bersifat sementara (*volatile*) dan dapat berubah sewaktu-waktu akibat pengurutan ulang (*sorting*), penyaringan (*filtering*), atau penambahan/penghapusan data. Sebaliknya, ID merupakan identitas unik yang stabil bagi entitas data. Dengan mengirimkan ID, fragmen tujuan dapat melakukan pencarian data secara deterministik ke sumber data (*single source of truth*) tanpa risiko menampilkan data yang salah saat urutan daftar berubah.

#### 5. Apa risiko jika tombol Kembali justru menjalankan navigate ke daftar baru?
Jika tombol "Kembali" memanggil `navController.navigate(R.id.listFragment)`, sistem navigasi akan membuat instansiasi baru `ListFragment` dan menumpuknya di atas *back stack*. Dampaknya, riwayat navigasi akan bertumpuk tanpa batas (`Home -> List -> Detail -> List -> Detail -> ...`). Ketika pengguna menekan tombol Back fisik perangkat, mereka harus melalui seluruh tumpukan duplikat tersebut sebelum bisa keluar dari aplikasi. Selain itu, state gulir dan data sebelumnya akan ter-reset. Oleh karena itu, kembali ke layar sebelumnya harus selalu menggunakan `navController.navigateUp()`.

#### 6. Apa perbedaan data contoh yang muncul lagi setelah restart dengan penyimpanan laporan permanen?
Data contoh pada `LaporanContoh` didefinisikan sebagai objek statis dalam memori (*in-memory static object*) yang diinisialisasi ulang dari kode Kotlin setiap kali proses aplikasi dimulai (`process creation`). Data ini tidak disimpan di media penyimpanan lokal. Sebaliknya, penyimpanan permanen (seperti SQLite, Room Database, atau SharedPreferences) menyimpan data ke dalam memori non-volatil perangkat (*flash storage*) atau server. Perubahan pada data permanen (seperti menambah atau menghapus laporan) akan tetap tersimpan meskipun aplikasi di-restart, perangkat dimatikan, atau memori RAM dibersihkan oleh sistem.

#### 7. Perubahan UX apa yang paling membantu teman saat mengerjakan tugas, dan apa buktinya?
Perubahan UX yang paling membantu adalah pemberian umpan balik visual yang tegas pada baris item (menggunakan `selectableItemBackground` untuk efek sentuhan *ripple* serta teks judul yang dicetak tebal dan kontras tinggi) serta keberadaan tombol "Kembali ke Daftar" yang jelas di bawah konten rincian. Buktinya tampak pada hasil uji kebergunaan: penguji berhasil menyelesaikan skenario tugas menemukan rincian zona 10 dan kembali ke daftar dalam waktu sangat singkat (6,8 detik) dengan tingkat kemudahan maksimal (skor 5/5) tanpa memerlukan bantuan sama sekali.

---

## 7. LAMPIRAN BUKTI TANGKAPAN LAYAR (SCREENSHOTS)

Berikut adalah daftar dokumentasi visual aplikasi WargaHub P04 yang tersimpan pada folder `screenshots/`:

| No | File Screenshot | Tampilan Antarmuka | Keterangan Fungsional |
|:---:|:---|:---|:---|
| 1 | `01_beranda.png` | Layar Beranda (*Home*) | Menampilkan teks pengantar, catatan data latihan, dan tombol pembuka daftar laporan. |
| 2 | `02_daftar.png` | Layar Daftar Laporan (*List*) | Menampilkan 15 laporan contoh, indikator jumlah data, dan tombol uji mode kosong. |
| 3 | `03_daftar_scroll.png` | Pengguliran Daftar (*Scrolled List*) | Menampilkan hasil scroll hingga item zona latihan 12–15 yang membuktikan kelancaran RecyclerView. |
| 4 | `04_rincian.png` | Layar Rincian Laporan (*Detail*) | Menampilkan detail lengkap laporan zona 13 dengan judul, kategori, status simulasi, dan deskripsi. |
| 5 | `05_kosong.png` | Mode Daftar Kosong (*Empty State*) | Menampilkan container kosong dengan pesan bantuan dan tombol pemulihan data contoh. |
| 6 | `06_landscape_atau_fontbesar.png` | Orientasi Mendatar (*Landscape*) | Membuktikan responsivitas layout dan insets system bar saat perangkat dirotasi. |
| 7 | `07_id_tidak_ditemukan.png` | Fallback ID Tidak Ditemukan (*T16*) | Membuktikan penanganan aman ketika menerima argumen ID `-99L` tanpa mengalami crash. |

---

## 8. KESIMPULAN

Praktikum 04 Pemrograman Mobile telah berhasil diselesaikan dengan mengimplementasikan seluruh arsitektur dan persyaratan yang diamanatkan dalam modul:
1. Aplikasi WargaHub P04 berhasil dibangun menggunakan pola *Single Activity Multiple Fragments* dengan Jetpack Navigation Component.
2. Komponen `RecyclerView`, `ListAdapter`, dan `DiffUtil` terbukti efisien dalam menyajikan 15 data fiktif secara responsif.
3. Pengelolaan siklus hidup Fragment dan ViewBinding terbebas dari resiko kebocoran memori dengan pelepasan referensi pada `onDestroyView()`.
4. Navigasi kembali terkelola dengan baik menggunakan `navigateUp()`, mencegah duplikasi layar pada back stack.
5. Kondisi ekstrim seperti daftar kosong dan kegagalan argumen ID berhasil ditangani secara anggun (*graceful fallback*), memberikan pengalaman pengguna (UX) yang aman, konsisten, dan berkualitas tinggi.
