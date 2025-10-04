# Arsitektur Detil - Sistem Putusan Pusat-Daerah

Dokumen ini menjelaskan diagram `diagram_arsitektur_detailed.svg` yang menggambarkan arsitektur terdistribusi antara Server Daerah (lokal) dan Server Pusat (nasional), serta komponen pendukung seperti message broker, indexing, dan strategi sinkronisasi.

## Komponen Utama

1. Server Daerah (per pengadilan)
   - Menangani CRUD putusan secara lokal.
   - Menyimpan file PDF dan detail lengkap (master data).
   - Menjalankan outbox pattern: setiap transaksi yang mengubah putusan menuliskan event ke tabel outbox dalam transaksi yang sama.
   - Melakukan publish event ke Message Broker (topik `putusan.events`).

2. Message Broker Cluster
   - Contoh teknologi: Apache Kafka, RabbitMQ, atau managed broker.
   - Menjamin durabilitas pesan (replication) dan mendukung consumer groups.
   - Topik utama: `putusan.events`, `sync.heartbeats`, `dlq` (dead-letter queue).

3. Server Pusat (Cluster)
   - Consumer event dari broker (Sync Worker) yang memperbarui `PUTUSAN_PUSAT` (metadata/indeks).
   - Menyediakan API publik untuk pencarian dan fetch metadata.
   - Mengelola indeks pencarian (Elasticsearch/OpenSearch) untuk query cepat.
   - Menyimpan `SINKRONISASI_LOG` untuk audit dan monitoring.

4. Pengguna (Clients)
   - Pengguna umum melakukan pencarian via API Server Pusat.
   - Jika membutuhkan file lengkap, Server Pusat memberikan redirect / signed URL ke Server Daerah atau menyajikan file jika direplikasi.

5. Optional Central File Store
   - Jika tersedia, file PDF bisa direplikasi ke central object storage untuk high-availability.

## Alur Sinkronisasi (Event-driven)

1. Admin Daerah melakukan perubahan (create/update/delete) pada Server Daerah.
2. Database Server Daerah melakukan commit dan menulis event ke tabel outbox (satu transaksi dengan perubahan data).
3. Outbox worker mem-publish event ke Message Broker.
4. Sync Worker di Server Pusat mengkonsumsi event, mem-validasi, dan memperbarui indeks metadata `PUTUSAN_PUSAT`.
5. Jika ada error, event dimasukkan ke DLQ, dan alert dikirim ke Admin Pusat.
6. Periodic reconciliation job dapat dilakukan untuk memastikan konsistensi antara pusat dan daerah.

## Model Konsistensi

- Server Daerah: consistency kuat (master lokal).
- Server Pusat: eventual consistency. Metadata di pusat akan memperbarui setelah event diproses.
- Untuk kebutuhan konsistensi global yang kuat, dapat diimplementasikan sinkronisasi sinkron (synchronous replication) namun dengan trade-off latensi.

## Fault Tolerance & Recoverability

- Message Broker: cluster dengan replication dan partitioning.
- Server Pusat: cluster dengan load balancer dan read replicas.
- Server Daerah: backup reguler, dan opsi failover lokal.
- Outbox pattern + durable broker untuk mencegah kehilangan event.
- Dead-letter queue untuk pesan yang gagal diproses, dengan alerting & manual intervention.

## Keamanan

- Semua komunikasi antar komponen via TLS/mTLS.
- Autentikasi & otorisasi: OAuth2 / OpenID Connect untuk admin, tokenized signed-URL untuk file access.
- API Gateway dengan rate limiting dan WAF.

## Monitoring & Operasional

- Metrics: sinkronization lag, broker consumer lag, error rates.
- Logs: central logging (ELK), tracing (OpenTelemetry) untuk alur event.
- Health checks & heartbeat events untuk mendeteksi server daerah yang offline.

## Checklist Implementasi

- [ ] Terapkan outbox pattern di Server Daerah.
- [ ] Deploy message broker yang durable.
- [ ] Implement idempotent consumers di Server Pusat.
- [ ] Setup DLQ + alerting.
- [ ] Rutin reconciliation job.
- [ ] TLS/mTLS dan autentikasi token.

---

File diagram yang dibuat: `diagram_arsitektur_detailed.svg`.

 Jika Anda mau, saya bisa:
 - Membuat anotasi PNG dari SVG (export) untuk dilampirkan; atau
 - Menambahkan versi singkat yang bisa ditempel di README; atau
 - Mengubah diagram sesuai preferensi warna, ukuran, atau menambah lebih banyak node contoh.

 Sebutkan pilihan Anda dan saya akan lanjutkan.

 ## Mapping DTO -> Tabel Database (Server Daerah)

 Ringkasan DTO yang direkomendasikan untuk Server Daerah beserta kolom tabel yang dipetakan:

 - PutusanDaerah (DTO) -> `PUTUSAN_DAERAH`
    - id_putusan -> id_putusan (UUID)
    - nomor_putusan -> nomor_putusan
    - jenis_putusan -> jenis_putusan
    - tingkat_proses -> tingkat_proses
    - klasifikasi -> klasifikasi
    - kata_kunci -> kata_kunci
    - tahun -> tahun
    - tanggal_register -> tanggal_register (ISO datetime)
    - amar -> amar
    - catatan_amar -> catatan_amar
    - file_pdf_url -> file_pdf_url
    - id_lembaga -> id_lembaga (FK ke LEMBAGA_PERADILAN)

 - PihakTerkait (DTO) -> `PIHAK_TERKAIT`
    - id_pihak -> id_pihak (UUID)
    - id_putusan -> id_putusan (FK)
    - nama_pihak -> nama_pihak
    - peran -> peran

 - Hakim (DTO) -> `HAKIM`
    - id_hakim -> id_hakim (UUID)
    - nama_hakim -> nama_hakim
    - nip -> nip (opsional)

 - PutusanHakim (DTO) -> `PUTUSAN_HAKIM` (pivot table)
    - id_putusan -> id_putusan (FK)
    - id_hakim -> id_hakim (FK)
    - peran -> peran (Ketua / Anggota)

 - Panitera (DTO) -> `PANITERA`
    - id_panitera -> id_panitera (UUID)
    - id_putusan -> id_putusan (FK)
    - nama_panitera -> nama_panitera

 - PutusanEvent / Outbox (DTO) -> `OUTBOX_EVENTS` (tabel outbox lokal)
    - event_id -> id (UUID)
    - event_type -> event_type
    - id_putusan -> id_putusan
    - payload -> json payload (hindari menyertakan file biner)
    - occurred_at -> timestamp

 Catatan validasi & penggunaan:
 - Semua ID utama direkomendasikan menggunakan UUID untuk memudahkan sinkronisasi dan traceability.
 - Validasi: gunakan schema validation (Zod / class-validator) pada boundary API sebelum menulis ke DB.
 - Outbox: pastikan menulis record perubahan dan record outbox dalam satu transaksi DB.
 - File: simpan file PDF di file storage lokal atau object storage regional; event hanya mengirimkan `file_pdf_url`.

 ---
