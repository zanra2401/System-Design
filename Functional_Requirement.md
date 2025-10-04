# Functional Requirements: Web Sistem Upload dan Melihat Putusan dengan Dua Admin (Admin Daerah dan Admin Pusat) dan Pengguna Umum

## 1. **Admin Authentication and Authorization (Autentikasi dan Otorisasi Pengguna)**

### **Requirement:**
- Pengguna harus dapat melakukan **login** dengan akun yang valid untuk mengakses sistem.
- **Role-based access control**: Pengguna dengan peran **admin pusat** dapat mengelola **monitoring sinkronisasi**, sementara **admin daerah** dapat **mengunggah**, **memperbarui**, **menghapus**, dan **melihat data putusan** yang ada di wilayahnya.
- **Pengguna umum** dapat mengakses **daftar putusan** dan **detail putusan** tanpa perlu login atau otentikasi.

### **Fitur:**
- **Admin Pusat** dan **Admin Daerah** login menggunakan **username** dan **password**.
- **Admin Pusat** dapat mengelola **sinkronisasi data** dan **memantau status event-driven**.
- **Admin Daerah** dapat mengelola **data putusan** di wilayahnya (CRUD data putusan).
- **Pengguna umum** dapat mengakses **daftar putusan** dan **melihat detail putusan** tanpa login.

---

## 2. **Upload Putusan (Pengunggahan Putusan) oleh Admin Daerah**

### **Requirement:**
- **Admin Daerah** dapat **mengunggah putusan** ke sistem untuk wilayahnya tanpa otentikasi untuk pengguna umum.
- **Admin Daerah** dapat mengunggah file **PDF** atau **format lain** yang berisi detail putusan (dengan metadata seperti nomor putusan, jenis perkara, hakim, tanggal, dll).
- Sistem harus menyimpan **metadata** putusan di **server pusat** dan file PDF di **server daerah**.
- Sistem harus memvalidasi format dan ukuran file yang diunggah.

### **Fitur:**
- Form upload untuk mengunggah file PDF putusan.
- Input untuk metadata (nomor putusan, jenis perkara, hakim, dsb).
- Sistem validasi untuk memastikan hanya file yang sesuai yang dapat diunggah.
- Menyimpan metadata putusan di **server pusat** dan file PDF di **server daerah**.
- Setelah **Admin Daerah** mengunggah putusan, sistem harus mengirimkan **event** ke **message broker** untuk sinkronisasi data real-time.

---

## 3. **Menampilkan Daftar Putusan (Tanpa Login untuk Pengguna Umum)**

### **Requirement:**
- **Pengguna umum** dapat **melihat daftar putusan** yang telah diunggah tanpa perlu login.
- Daftar putusan ditampilkan berdasarkan **metadata** seperti nomor putusan, jenis perkara, tahun, dll.
- Pengguna umum dapat **memfilter** dan **mencari putusan** berdasarkan kriteria tertentu (misalnya nomor putusan, jenis perkara, atau tahun).
- Daftar putusan harus memiliki **pagination** untuk menghindari overload data saat banyak putusan.

### **Fitur:**
- Tampilan daftar putusan dengan informasi dasar seperti nomor putusan, jenis perkara, hakim, dan tanggal.
- Fungsi **pencarian** dan **filtering** untuk memudahkan pengguna menemukan putusan tertentu.
- **Pagination** atau **infinite scroll** untuk menampilkan banyak putusan secara efisien.
- Setiap item di daftar dapat **di-klik** untuk melihat detail lebih lanjut dari putusan tersebut.
- **Pengguna umum** dapat melihat daftar putusan tanpa proses login.

---

## 4. **Melihat Detail Putusan (Tanpa Login untuk Pengguna Umum)**

### **Requirement:**
- **Pengguna umum** dapat melihat **detail lengkap putusan** setelah memilih satu putusan dari daftar, tanpa login.
- **File PDF** atau format lain yang diunggah sebelumnya dapat diakses dan dilihat di halaman detail.
- Semua **metadata** terkait putusan (nomor putusan, hakim, jenis perkara, dll) harus ditampilkan di halaman detail.

### **Fitur:**
- Tampilan **file PDF** atau **link** untuk mengunduh file putusan.
- Tampilan lengkap dari **metadata putusan** (nomor putusan, hakim, jenis perkara, dll).
- Pengguna dapat **mengunduh file** putusan jika diinginkan.
- **Pengguna umum** dapat mengakses **detail putusan** tanpa login.

---

## 5. **Sistem Pencarian dan Filter Putusan**

### **Requirement:**
- Pengguna umum dapat melakukan **pencarian** terhadap **putusan** menggunakan **kata kunci** atau **parameter** tertentu.
- Filter berdasarkan **tahun**, **jenis perkara**, atau **hakim** untuk menyaring putusan yang relevan.

### **Fitur:**
- **Search bar** untuk mencari berdasarkan nomor putusan, hakim, atau kata kunci.
- Filter berdasarkan **kategori** seperti **jenis perkara**, **tahun**, atau **daerah pengadilan**.
- Pengguna dapat **menyusun hasil pencarian** berdasarkan **tanggal**, **nomor**, atau **jenis perkara**.

---

## 6. **Pengelolaan Data Putusan oleh Admin Daerah**

### **Requirement:**
- **Admin Daerah** dapat **mengedit**, **menghapus**, atau **memperbarui metadata** atau **file putusan** yang telah diunggah.
- **Admin Daerah** dapat melihat daftar **putusan yang telah diunggah** beserta metadata terkait.
- **Admin Daerah** harus bisa **melihat riwayat pengunggahan** untuk memonitor status upload dan perubahan data.

### **Fitur:**
- Halaman admin untuk mengelola **data putusan** yang telah diunggah.
- Fitur untuk **update** atau **hapus** putusan dan metadata.
- **Riwayat perubahan** putusan, termasuk siapa yang mengubah dan kapan.

---

## 7. **Monitoring Sinkronisasi Data oleh Admin Pusat**

### **Requirement:**
- **Admin Pusat** harus dapat memantau status **sinkronisasi data** antara **server pusat** dan **server daerah** menggunakan sistem **event-driven**.
- Sistem harus mengirimkan **event** ke **message broker** setiap kali ada perubahan data (misalnya saat **Admin Daerah** mengunggah putusan).
- **Admin Pusat** harus dapat melihat laporan status event untuk memastikan bahwa sinkronisasi data **berhasil** atau **gagal**.

### **Fitur:**
- **Admin Pusat** dapat melihat **riwayat sinkronisasi** dan statusnya (berhasil/gagal).
- Laporan status sinkronisasi yang memberikan informasi rinci tentang apakah event-driven system berhasil menyinkronkan **metadata** dan **data lengkap**.
- Sistem memberikan **notifikasi** kepada **Admin Pusat** jika terjadi **kegagalan** dalam sinkronisasi data.

---

## 8. **Integrasi dengan Sistem Message Broker untuk Sinkronisasi Data**

### **Requirement:**
- Jika menggunakan **sistem event-driven**, saat ada perubahan data atau penambahan putusan, sistem harus mengirimkan **event** ke **message broker** untuk sinkronisasi data real-time.
- Sinkronisasi ini akan memastikan bahwa **server pusat** dan **server daerah** selalu memiliki data yang konsisten dan terbaru.
- **Admin Pusat** dapat memonitor status sinkronisasi dan mengambil tindakan jika terjadi masalah.

### **Fitur:**
- **Event-driven architecture** untuk memicu sinkronisasi data antar **server pusat** dan **server daerah**.
- Pengiriman **event** yang berisi informasi perubahan data atau penambahan putusan.
- **Laporan status** yang dapat dipantau oleh **Admin Pusat** untuk memastikan sinkronisasi berjalan dengan baik.

---

### **Perubahan yang Diterapkan:**
- **Pengguna Umum** dapat **melihat daftar putusan** dan **detail putusan** tanpa perlu login.
- **Admin Daerah** bertanggung jawab atas **CRUD data putusan** di wilayahnya.
- **Admin Pusat** fokus pada **monitoring sinkronisasi** dan **memantau event-driven architecture** untuk memastikan bahwa proses sinkronisasi antara **server pusat** dan **server daerah** berjalan lancar dan berhasil.

Dengan **dua peran admin** (Admin Pusat dan Admin Daerah) serta **akses publik** untuk pengguna umum, sistem ini menjadi lebih fleksibel dan mudah diakses.

---

Jika ada hal lain yang perlu disesuaikan atau dijelaskan lebih lanjut, beri tahu saya!
