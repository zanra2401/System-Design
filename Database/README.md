Database 

# 🏛️ Arsitektur Basis Data Sistem Putusan Terdistribusi Nasional

Sistem ini menggunakan **arsitektur terdistribusi** antara **Server Daerah (lokal)** dan **Server Pusat (nasional)**.  
Setiap daerah menyimpan data putusan secara lengkap, sedangkan pusat hanya menyimpan data ringkas sebagai indeks nasional.

---

## 🌍 Server Daerah (Lokal di Pengadilan)

Server ini dimiliki oleh masing-masing **pengadilan daerah** (misalnya Pengadilan Negeri Bojonegoro).  
Data di sini adalah **master data** yang dikelola oleh **Admin Daerah**.

### 🧩 Tabel-Tabel di Server Daerah

#### 1. `PUTUSAN_DAERAH`
Berisi data lengkap putusan, mencakup metadata dan file dokumen (PDF).

**Kolom penting:**
- `id_putusan`
- `nomor_putusan`
- `jenis_putusan`
- `tingkat_proses`
- `klasifikasi`
- `kata_kunci`
- `tahun`
- `tanggal_register`
- `amar`, `catatan_amar`
- `file_pdf_url`
- `id_lembaga` → relasi ke `LEMBAGA_PERADILAN`

---

#### 2. `PIHAK_TERKAIT`
Menyimpan data para pihak dalam perkara.

**Kolom penting:**
- `id_pihak`
- `nama_pihak`
- `peran` → (Terdakwa, Pemohon, Penuntut Umum)
- `id_putusan` → relasi ke `PUTUSAN_DAERAH`

---

#### 3. `HAKIM` dan `PUTUSAN_HAKIM`
Relasi many-to-many antara putusan dan hakim.

**Kolom penting:**
- `HAKIM.id_hakim`, `nama_hakim`
- `PUTUSAN_HAKIM.id_putusan`, `PUTUSAN_HAKIM.id_hakim`, `peran` (Hakim Ketua / Hakim Anggota)

---

#### 4. `PANITERA`
Data panitera yang menangani putusan.

**Kolom penting:**
- `id_panitera`
- `nama_panitera`
- `id_putusan` → relasi ke `PUTUSAN_DAERAH`

---

🧠 **Kesimpulan:**
> Semua tabel di server daerah bersifat **master data**.  
> Semua input, edit, dan penghapusan dilakukan oleh **Admin Daerah**.  
> Server daerah menjadi sumber kebenaran (*source of truth*) bagi data putusan.

---

## 🏢 Server Pusat (Nasional)

Server ini berfungsi sebagai **portal nasional** yang mengindeks seluruh data putusan dari berbagai daerah.  
Server pusat **tidak menyimpan file PDF atau detail isi putusan**, hanya metadata ringkas untuk keperluan pencarian dan statistik nasional.

### 🧩 Tabel-Tabel di Server Pusat

#### 1. `PUTUSAN_PUSAT`
Menyimpan metadata dasar dari putusan hasil sinkronisasi dari daerah.

**Kolom penting:**
- `id_putusan_pusat`
- `id_putusan_daerah` → relasi ke data asli di daerah
- `id_lembaga` → pengadilan yang mengeluarkan putusan
- `nomor_putusan`, `tahun`, `jenis_putusan`, `kata_kunci`
- `url_detail` → tautan langsung ke server daerah
- `status_validasi` → (Terverifikasi / Belum Diverifikasi)

---

#### 2. `DAERAH`
Master daftar daerah di seluruh Indonesia.

**Kolom penting:**
- `id_daerah`
- `nama_daerah`
- `provinsi`
- `wilayah_hukum`
- `url_server`

---

#### 3. `LEMBAGA_PERADILAN`
Master daftar pengadilan dan jenis peradilannya.

**Kolom penting:**
- `id_lembaga`
- `nama_lembaga`
- `jenis_lembaga` → (Peradilan Umum, Agama, TUN, Militer)
- `tingkatan` → (PN, PT, MA)
- `id_daerah` → relasi ke `DAERAH`
- `url_api`

---

#### 4. `SINKRONISASI_LOG`
Mencatat proses sinkronisasi antara pusat dan daerah.

**Kolom penting:**
- `id_sync`
- `id_lembaga`
- `waktu_sync`
- `status` → (berhasil, gagal, pending)
- `jumlah_data`
- `pesan_error`

---

🧠 **Kesimpulan:**
> Server pusat bersifat **indexing & agregasi nasional**.  
> Tidak menyimpan dokumen penuh, hanya metadata ringkas.  
> Data diperbarui secara **otomatis dan real-time** dari server daerah.

---

## 🔗 Hubungan Antara Server Daerah dan Pusat

1. **Admin Daerah** mengunggah atau mengubah data pada `PUTUSAN_DAERAH`.  
2. **Server Daerah** mengirim event sinkronisasi (realtime) ke **Server Pusat**.  
3. **Server Pusat** memperbarui atau menambah data di `PUTUSAN_PUSAT`.  
4. **Pengguna Publik** mencari data melalui portal pusat dan diarahkan ke `url_detail` dari daerah.  

---

### 🧾 Kesimpulan Akhir

| Lapisan | Tipe Data | Akses | Contoh Tabel |
|----------|------------|--------|---------------|
| **Server Daerah** | Master Data Lengkap | Admin Daerah | PUTUSAN_DAERAH, PIHAK_TERKAIT, HAKIM, PANITERA |
| **Server Pusat** | Metadata Ringkas / Indeks Nasional | Admin Pusat & Publik | PUTUSAN_PUSAT, DAERAH, LEMBAGA_PERADILAN, SINKRONISASI_LOG |

---

💡 Dengan desain ini:
- Sistem tetap **terdistribusi** (tiap pengadilan mandiri).  
- **Sinkronisasi real-time** menjamin data pusat selalu terbaru.  
- Beban pusat ringan karena hanya menyimpan indeks, bukan seluruh file dokumen.
