# Functional Requirements: Web Sistem Upload dan Melihat Putusan

## 1. **User Authentication and Authorization (Autentikasi dan Otorisasi Pengguna)**

**Requirement:**
- Pengguna harus dapat melakukan **login** dengan akun yang valid untuk mengakses sistem.
- **Role-based access control**: Pengguna dengan peran **admin** dapat mengunggah dan mengelola putusan, sementara pengguna biasa hanya dapat melihat putusan.
- Otorisasi untuk memastikan hanya **admin** yang dapat **upload putusan** dan **pengguna biasa** hanya dapat melihat putusan yang sudah ada.

**Fitur:**
- Login menggunakan **username** dan **password**.
- Admin memiliki akses penuh untuk upload, update, dan menghapus putusan.
- Pengguna biasa hanya dapat melihat putusan yang ada.

---

## 2. **Upload Putusan (Pengunggahan Putusan)**

**Requirement:**
- Admin harus dapat **mengunggah putusan** ke sistem.
- Admin dapat mengunggah file **PDF** atau **format lain** yang berisi detail putusan (dengan metadata seperti nomor putusan, jenis perkara, hakim, tanggal, dll).
- Sistem harus menyimpan **metadata** putusan dan menyediakan **URL** untuk akses langsung ke file PDF.
- Sistem harus memvalidasi format dan ukuran file yang diunggah.

**Fitur:**
- Form upload untuk mengunggah file PDF putusan.
- Input untuk metadata (nomor putusan, jenis perkara, hakim, dsb).
- Sistem validasi untuk memastikan hanya file yang sesuai yang dapat diunggah.
- Menyimpan metadata putusan di **server pusat** dan file PDF di **server daerah**.

---

## 3. **Menampilkan Daftar Putusan**

**Requirement:**
- Pengguna dapat **melihat daftar putusan** yang telah diunggah.
- Daftar putusan ditampilkan berdasarkan **metadata** seperti nomor putusan, jenis perkara, tahun, dll.
- Pengguna dapat **memfilter** dan **mencari putusan** berdasarkan kriteria tertentu (misalnya nomor putusan, jenis perkara, atau tahun).
- Daftar putusan harus memiliki **pagination** untuk menghindari overload data saat banyak putusan.

**Fitur:**
- Tampilan daftar putusan dengan informasi dasar seperti nomor putusan, jenis perkara, hakim, dan tanggal.
- Fungsi **pencarian** dan **filtering** untuk memudahkan pengguna menemukan putusan tertentu.
- **Pagination** atau **infinite scroll** untuk menampilkan banyak putusan secara efisien.
- Setiap item di daftar dapat **di-klik** untuk melihat detail lebih lanjut dari putusan tersebut.

---

## 4. **Melihat Detail Putusan**

**Requirement:**
- Pengguna dapat melihat **detail lengkap putusan** setelah memilih satu putusan dari daftar.
- **File PDF** atau format lain yang diunggah sebelumnya dapat diakses dan dilihat di halaman detail.
- Semua **metadata** terkait putusan (nomor putusan, hakim, jenis perkara, dll) harus ditampilkan di halaman detail.

**Fitur:**
- Tampilan **file PDF** atau **link** untuk mengunduh file putusan.
- Tampilan lengkap dari **metadata putusan** (nomor putusan, hakim, jenis perkara, dll).
- Pengguna dapat **mengunduh file** putusan jika diinginkan.

---

## 5. **Sistem Pencarian dan Filter Putusan**

**Requirement:**
- Pengguna dapat melakukan **pencarian** terhadap **putusan** menggunakan **kata kunci** atau **parameter** tertentu.
- Filter berdasarkan **tahun**, **jenis perkara**, atau **hakim** untuk menyaring putusan yang relevan.

**Fitur:**
- **Search bar** untuk mencari berdasarkan nomor putusan, hakim, atau kata kunci.
- Filter berdasarkan **kategori** seperti **jenis perkara**, **tahun**, atau **daerah pengadilan**.
- Pengguna dapat **menyusun hasil pencarian** berdasarkan **tanggal**, **nomor**, atau **jenis perkara**.

---

## 6. **Pengelolaan Data Putusan oleh Admin**

**Requirement:**
- Admin dapat **mengedit**, **menghapus**, atau **memperbarui metadata** atau **file putusan** yang telah diunggah.
- Admin dapat melihat daftar **putusan yang telah diunggah** beserta metadata terkait.
- Admin harus bisa **melihat riwayat pengunggahan** untuk memonitor status upload dan perubahan data.

**Fitur:**
- Halaman admin untuk mengelola **data putusan** yang telah diunggah.
- Fitur untuk **update** atau **hapus** putusan dan metadata.
- **Riwayat perubahan** putusan, termasuk siapa yang mengubah dan kapan.

---

## 7. **Integrasi dengan Sistem Message Broker untuk Sinkronisasi Data**

**Requirement:**
- Jika menggunakan **sistem event-driven**, saat ada perubahan data atau penambahan putusan, sistem harus mengirimkan **event** ke **message broker** untuk sinkronisasi data real-time.
- Sinkronisasi ini akan memastikan bahwa
