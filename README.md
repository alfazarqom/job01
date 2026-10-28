# E-commerce Guest Checkout dengan Duitku POP

> **Status:** Draft V1.1 — dokumen rancangan, bukan bukti bahwa fitur atau integrasi sudah diimplementasikan.  
> **Konsep:** Toko online dengan checkout tanpa akun pengguna (*guest checkout*).  
> **Pembayaran:** Duitku POP.  
> **Database inti yang direncanakan:** 7 tabel.

## Daftar Isi

- [Ringkasan Proyek](#ringkasan-proyek)
- [Ruang Lingkup MVP](#ruang-lingkup-mvp)
- [Daftar Halaman](#daftar-halaman)
- [Rancangan Database](#rancangan-database)
- [Integrasi Duitku POP](#integrasi-duitku-pop)
- [Alur Checkout dan Pembayaran](#alur-checkout-dan-pembayaran)
- [Keamanan dan Validasi](#keamanan-dan-validasi)
- [Aturan Status Pesanan](#aturan-status-pesanan)
- [Tahapan Pengembangan](#tahapan-pengembangan)
- [Catatan Implementasi](#catatan-implementasi)

## Ringkasan Proyek

Aplikasi ini dirancang sebagai e-commerce yang memungkinkan pembeli melihat produk, mengelola keranjang, dan menyelesaikan pembelian tanpa harus membuat akun atau login. Pembayaran dilakukan melalui halaman pembayaran Duitku POP, sedangkan status transaksi ditentukan oleh backend setelah memvalidasi notifikasi pembayaran.

Dokumen ini menjadi rancangan awal untuk menyelaraskan halaman aplikasi, struktur data, proses checkout, dan integrasi pembayaran. Detail teknis dapat disesuaikan setelah kebutuhan bisnis dan dokumentasi resmi penyedia layanan diverifikasi.

## Ruang Lingkup MVP

Fokus versi awal:

- Katalog produk, pencarian, kategori, dan detail produk.
- Keranjang belanja untuk pengguna guest.
- Checkout tanpa login.
- Informasi pembeli dan alamat pengiriman.
- Ringkasan pesanan dan perhitungan total di backend.
- Pembuatan transaksi Duitku POP melalui backend.
- Konfirmasi pembayaran berdasarkan verifikasi backend.
- Detail pesanan dan pelacakan pengiriman yang aman.

Wishlist, voucher dan promo, ulasan, pembatalan mandiri, serta pengajuan retur/refund dapat ditunda ke tahap berikutnya.

## Daftar Halaman

| No. | Halaman | Fungsi | Prioritas |
|---:|---|---|---|
| 1 | Splash Screen | Tampilan awal aplikasi | Pendukung |
| 2 | Onboarding | Pengenalan aplikasi | Pendukung |
| 3 | Home | Menampilkan produk dan promo | MVP |
| 4 | Search | Mencari produk | MVP |
| 5 | Category | Menampilkan kategori produk | MVP |
| 6 | Product Listing | Menampilkan daftar produk | MVP |
| 7 | Filter & Sort | Menyaring dan mengurutkan produk | Lanjutan |
| 8 | Product Detail | Menampilkan detail produk | MVP |
| 9 | Wishlist | Menyimpan produk favorit | Lanjutan |
| 10 | Cart | Menampilkan keranjang belanja | MVP |
| 11 | Edit Cart | Mengubah jumlah atau menghapus produk | MVP |
| 12 | Voucher & Promo | Memasukkan kode promo | Lanjutan |
| 13 | Guest Checkout | Memulai checkout tanpa login | MVP |
| 14 | Buyer Information | Mengisi nama, email, dan nomor telepon | MVP |
| 15 | Shipping Address | Mengisi alamat pengiriman | MVP |
| 16 | Shipping Method | Memilih layanan pengiriman | MVP |
| 17 | Payment Method | Memulai pembayaran melalui Duitku POP | MVP |
| 18 | Order Summary | Memeriksa rincian dan total pesanan | MVP |
| 19 | Payment Processing | Memulai proses pembayaran | MVP |
| 20 | Payment Pending | Menampilkan status menunggu pembayaran | MVP |
| 21 | Payment Failed | Menampilkan status pembayaran gagal | MVP |
| 22 | Order Success | Mengonfirmasi pesanan sesuai status backend | MVP |
| 23 | Order Detail | Menampilkan rincian transaksi | MVP |
| 24 | Track Order | Melacak status pengiriman | MVP |
| 25 | Cancel Order | Membatalkan pesanan sesuai ketentuan | Lanjutan |
| 26 | Return & Refund | Mengajukan retur barang atau pengembalian dana | Lanjutan |
| 27 | Help Center / FAQ | Menampilkan panduan bantuan | Pendukung |
| 28 | Contact Customer Service | Menghubungi layanan pelanggan | Pendukung |
| 29 | About Us | Informasi tentang aplikasi | Pendukung |
| 30 | Terms & Conditions | Syarat dan ketentuan penggunaan | Pendukung |
| 31 | Privacy Policy | Kebijakan privasi | Pendukung |

### Catatan halaman

- Checkout tidak memerlukan akun pengguna.
- Halaman hasil pembayaran tidak boleh menetapkan status transaksi hanya berdasarkan redirect atau callback JavaScript di browser.
- Detail dan pelacakan pesanan harus menggunakan token akses yang sulit ditebak atau mekanisme verifikasi lain yang aman; nomor pesanan saja bukan rahasia autentikasi.
- Prioritas MVP di atas merupakan usulan perencanaan dan bukan pernyataan bahwa halaman sudah tersedia.

## Rancangan Database

Struktur berikut merupakan rancangan awal. Nama tabel dan kolom harus diverifikasi terhadap implementasi serta kebutuhan bisnis sebelum dibuatkan migrasi.

| No. | Tabel | Kolom utama yang direncanakan | Fungsi |
|---:|---|---|---|
| 1 | `products` | `id`, `name`, `description`, `price`, `stock`, `category`, `product_image` | Menyimpan data produk |
| 2 | `carts` | `id`, `session_id`, `created_at` | Menyimpan keranjang guest |
| 3 | `cart_items` | `id`, `cart_id`, `product_id`, `quantity` | Menyimpan item keranjang |
| 4 | `orders` | `id`, `customer_name`, `email`, `phone`, `address`, `total_amount`, `order_status`, `created_at` | Menyimpan data pembeli dan pesanan |
| 5 | `order_items` | `id`, `order_id`, `product_id`, `product_name`, `price`, `quantity`, `subtotal` | Menyimpan snapshot item pesanan |
| 6 | `payments` | `id`, `order_id`, `merchant_order_id`, `duitku_reference`, `payment_method`, `payment_status`, `payment_url`, `paid_at` | Menyimpan data transaksi Duitku POP |
| 7 | `shipments` | `id`, `order_id`, `courier`, `tracking_number`, `shipping_status` | Menyimpan data pengiriman |

### Prinsip struktur data

- Kategori dan gambar produk pada rancangan awal disimpan di `products`; bentuk kolomnya perlu disesuaikan dengan kebutuhan katalog.
- Data pembeli dan alamat pengiriman disimpan pada pesanan. Batasi akses ke data pribadi tersebut.
- Keranjang guest diidentifikasi dengan `session_id` yang tidak mudah ditebak dan dikelola dengan aman.
- `order_items.product_name` dan `order_items.price` adalah snapshot saat pesanan dibuat agar riwayat transaksi tidak berubah ketika data produk diperbarui.
- `merchant_order_id` harus unik, dengan constraint unik di database.
- Nilai harga dan total harus menggunakan tipe numerik yang sesuai untuk mata uang; jangan mengandalkan floating-point untuk perhitungan uang.
- Total, harga, diskon, ongkos kirim, dan stok harus divalidasi atau dihitung di backend. Jangan mempercayai nilai total dari frontend.
- Pertimbangkan constraint foreign key, indeks, aturan retensi data, serta Row Level Security (RLS) dan grants yang sesuai.
- Tujuh tabel di atas adalah rancangan inti, bukan klaim bahwa skema tersebut sudah tersedia di database.

## Integrasi Duitku POP

Kolom transaksi yang direncanakan:

| Kolom | Tujuan |
|---|---|
| `merchant_order_id` | ID transaksi unik yang dibuat sistem toko |
| `duitku_reference` | Referensi transaksi dari Duitku, sesuai respons API |
| `payment_url` | URL halaman pembayaran Duitku POP |
| `payment_method` | Metode pembayaran yang dipilih atau dilaporkan |
| `payment_status` | Status pembayaran yang telah diproses backend |
| `paid_at` | Waktu pembayaran terkonfirmasi |

### Prinsip integrasi

1. Backend memvalidasi pesanan, harga, ongkos kirim, diskon, dan ketersediaan stok.
2. Backend membuat identitas transaksi unik dan mengirim permintaan ke Duitku POP sesuai dokumentasi resmi dan lingkungan yang digunakan.
3. Backend memvalidasi respons dan menyimpan referensi transaksi serta URL pembayaran.
4. Frontend mengarahkan pembeli ke URL pembayaran yang diterima dari backend.
5. Backend menerima callback Duitku melalui endpoint yang sesuai.
6. Backend memverifikasi signature dan mencocokkan identitas transaksi, nominal, merchant, dan status dengan catatan lokal.
7. Backend memperbarui status pembayaran dan pesanan secara aman serta idempoten.
8. Halaman konfirmasi membaca status dari backend; redirect browser hanya menjadi petunjuk navigasi, bukan bukti pembayaran.

**Dokumentasi resmi untuk verifikasi implementasi:**

- [Duitku POP — Dokumentasi Indonesia](https://docs.duitku.com/pop/id/)
- [Duitku API — Dokumentasi Indonesia](https://docs.duitku.com/api/id/)

Parameter request, format signature, respons, kode status, dan kontrak callback harus mengikuti dokumentasi Duitku yang berlaku. Jangan mengasumsikan nama field atau algoritme signature tanpa memverifikasinya.

## Alur Checkout dan Pembayaran

1. Pembeli memilih produk dan memasukkannya ke keranjang.
2. Pembeli memulai checkout sebagai guest.
3. Pembeli mengisi informasi kontak dan alamat pengiriman.
4. Backend memvalidasi ulang produk, harga, stok, diskon, dan ongkos kirim.
5. Sistem membuat pesanan dan snapshot item pesanan.
6. Backend membuat transaksi pembayaran ke Duitku POP.
7. Backend menyimpan identitas transaksi dan informasi pembayaran yang tervalidasi.
8. Pembeli diarahkan ke halaman pembayaran Duitku.
9. Duitku mengirim callback ke backend ketika status transaksi berubah.
10. Backend memverifikasi signature dan kecocokan data transaksi.
11. Backend memproses callback secara idempoten dan memperbarui status yang sah.
12. Pembeli melihat status pembayaran dan pesanan dari endpoint backend yang aman.
13. Pesanan yang telah dibayar diteruskan ke proses pemenuhan dan pengiriman sesuai kebijakan toko.

### Penanganan kegagalan

- Jika pembuatan invoice gagal, jangan menampilkan URL pembayaran yang tidak valid.
- Jika callback tidak diterima atau status belum jelas, sediakan proses pemeriksaan status melalui backend sesuai kontrak resmi.
- Callback duplikat tidak boleh menggandakan perubahan stok, pembayaran, atau pemenuhan pesanan.
- Kegagalan jaringan tidak boleh dianggap sebagai bukti pembayaran gagal maupun berhasil.
- Aturan kedaluwarsa, pembatalan, pengembalian stok, dan rekonsiliasi harus ditetapkan sebelum transaksi nyata digunakan.

## Keamanan dan Validasi

- **Kredensial:** simpan merchant key dan kredensial Duitku hanya di lingkungan backend/secret manager. Jangan masukkan ke kode frontend, README, atau log.
- **Signature callback:** verifikasi menggunakan metode dan format resmi Duitku; tolak callback dengan signature tidak valid.
- **Validasi transaksi:** cocokkan merchant, `merchant_order_id`, referensi transaksi jika tersedia, nominal, dan status.
- **Idempotensi:** callback yang sama dapat diproses berulang tanpa efek ganda. Gunakan constraint dan/atau transaksi database yang sesuai.
- **Sumber status:** status pembayaran hanya berubah setelah validasi backend berhasil. Jangan mempercayai query parameter redirect atau status yang dikirim browser.
- **Otorisasi pesanan:** lindungi detail pesanan dan pelacakan menggunakan token akses acak atau verifikasi aman. Hindari ID berurutan sebagai satu-satunya pengaman.
- **Stok:** lakukan pemeriksaan dan pengurangan stok secara atomik untuk mencegah overselling. Tetapkan kapan stok direservasi, dilepas, atau dikembalikan.
- **Perhitungan:** hitung ulang harga dan total di server; frontend hanya menampilkan hasil.
- **Validasi input:** validasi tipe, panjang, format, dan batas nilai untuk semua input pengguna.
- **Privasi:** batasi akses terhadap nama, email, nomor telepon, dan alamat; jangan mencatat data pribadi atau rahasia pembayaran secara berlebihan ke log.
- **Transportasi:** gunakan HTTPS untuk endpoint publik dan integrasi gateway.
- **Pengujian:** uji signature tidak valid, nominal tidak cocok, callback duplikat, transaksi tidak dikenal, status yang datang tidak berurutan, serta kegagalan jaringan.

### Aturan status pesanan

Pesanan baru diproses setelah pembayaran terkonfirmasi jika kebijakan toko mengharuskan pembayaran di muka. Pesanan belum dibayar tetap menunggu pembayaran dan dapat kedaluwarsa atau dibatalkan sesuai batas waktu yang ditentukan.

Nama status di bawah ini adalah usulan awal; enum final dan transisi yang diperbolehkan perlu ditetapkan sebelum implementasi.

| Domain | Status contoh |
|---|---|
| Pesanan | `pending_payment`, `processing`, `shipped`, `completed`, `cancelled` |
| Pembayaran | `pending`, `success`, `failed`, `expired`, `refunded`, `partially_refunded` |
| Pengiriman | `not_shipped`, `shipped`, `delivered`, `failed` |

Status pembayaran dan pesanan merupakan hal yang berbeda. Perubahan status harus mengikuti aturan transisi backend, bukan langsung menyalin input dari klien.

## Tahapan Pengembangan

1. **Konfirmasi kebutuhan:** kebijakan ongkos kirim, batas waktu pembayaran, reservasi stok, pembatalan, retur, dan refund.
2. **Audit skema:** cocokkan rancangan tabel dengan database aktual; tentukan constraints, indeks, RLS, grants, dan mekanisme akses guest.
3. **Rancang API backend:** kontrak pembuatan pesanan, invoice, callback, pemeriksaan status, dan pelacakan.
4. **Implementasi sandbox:** integrasikan Duitku POP dengan kredensial sandbox yang disimpan aman.
5. **Uji keamanan dan alur:** uji callback, signature, idempotensi, nominal, stok, status, dan kegagalan.
6. **Verifikasi operasional:** siapkan logging tanpa rahasia, rekonsiliasi, pemantauan, serta prosedur penanganan insiden.
7. **Rilis produksi:** hanya setelah pengujian dan pemeriksaan konfigurasi selesai serta persetujuan rilis diberikan.

## Catatan Implementasi

- README ini mendokumentasikan rancangan Draft V1.1. Keberadaan fitur, tabel, endpoint, dan integrasi harus dibuktikan melalui audit kode serta database.
- Gunakan lingkungan sandbox terlebih dahulu. Jangan melakukan transaksi nyata sebelum seluruh alur diverifikasi.
- Jangan menyimpan secret atau data pembeli sungguhan di repository.
- Perubahan skema database, kredensial, konfigurasi produksi, dan deployment harus ditinjau secara terpisah.
- Perbarui dokumen ini ketika kontrak API, struktur database, aturan bisnis, atau status implementasi telah dikonfirmasi.

---

**Versi dokumen:** Draft V1.1  
**Konsep:** E-commerce guest checkout tanpa akun pengguna  
**Integrasi pembayaran:** Duitku POP  
**Database inti yang direncanakan:** 7 tabel  
