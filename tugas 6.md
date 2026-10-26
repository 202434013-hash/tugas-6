# Laporan Praktikum Minggu 6 — Pagination, Filtering, dan Sorting

## 1. Tujuan

Praktikum Minggu 6 bertujuan mengembangkan endpoint collection buku pada aplikasi Laravel agar mendukung pagination, pencarian berdasarkan beberapa field, sorting dengan allowlist, dan validasi query parameter. Pengujian dilakukan menggunakan Postman untuk memastikan implementasi sesuai dengan API contract project.

## 2. Endpoint dan Query Contract

Endpoint yang digunakan:

`GET http://127.0.0.1:8000/api/books`

| Parameter | Tipe | Aturan | Default |
|---|---|---|---|
| search | String atau null | Maksimal 100 karakter | Tidak ada filter |
| sort | String | id, title, isbn, created_at | id |
| direction | String | asc atau desc | asc |
| page | Integer | Minimal 1 | 1 |
| per_page | Integer | Minimal 1 dan maksimal 50 | 5 |

Header request:

`Accept: application/json`

Endpoint menggunakan method GET untuk mengambil daftar buku dengan query parameter.

## 3. Field Search dan Sort Allowlist

### Search

Pencarian dilakukan pada dua field, yaitu:

- `title`: mencari berdasarkan judul buku.
- `isbn`: mencari berdasarkan nomor ISBN buku.

Pencarian menggunakan kondisi `LIKE` sehingga pengguna dapat mencari teks yang terdapat di dalam judul atau ISBN.

Contoh request:

`GET /api/books?search=Working`

`GET /api/books?search=9780131177055`

Jika data ditemukan, buku ditampilkan pada properti `data`. Jika tidak ada data yang cocok, API dapat mengembalikan `200 OK` dengan array `data` kosong.

### Sorting Allowlist

Kolom yang diperbolehkan untuk sorting adalah:

- `id`
- `title`
- `isbn`
- `created_at`

Parameter `direction` hanya menerima `asc` atau `desc`.

Contoh ascending:

`GET /api/books?sort=title&direction=asc`

Contoh descending:

`GET /api/books?sort=title&direction=desc`

Allowlist digunakan agar pengguna hanya dapat mengurutkan data berdasarkan kolom yang telah ditentukan.

## 4. Validation Rules

Validasi query parameter pada method `index()` di `BookController` adalah sebagai berikut:

- `search`: `sometimes`, `nullable`, `string`, `max:100`.
- `sort`: `sometimes`, `Rule::in(['id', 'title', 'isbn', 'created_at'])`.
- `direction`: `sometimes`, `Rule::in(['asc', 'desc'])`.
- `page`: `sometimes`, `integer`, `min:1`.
- `per_page`: `sometimes`, `integer`, `min:1`, `max:50`.

Jika parameter tidak memenuhi aturan validasi, Laravel mengembalikan status `422 Unprocessable Entity` dengan detail kesalahan dalam format JSON apabila request menggunakan header `Accept: application/json`.

## 5. Bukti Pagination dan Metadata

### Default Pagination

Request:

`GET /api/books`

Hasil pengujian Postman: `200 OK`.

Response body menampilkan daftar buku di dalam properti `data`.

### Custom Pagination

Request:

`GET /api/books?page=2&per_page=2`

Hasil yang diharapkan adalah `200 OK`, dengan `meta.current_page` bernilai 2 dan `meta.per_page` bernilai 2 jika data mencukupi.

### Struktur Response Pagination

Respons collection memiliki tiga bagian penting:

- `data`: berisi daftar buku pada halaman yang diminta.
- `links`: berisi tautan navigasi halaman, seperti first, last, prev, dan next.
- `meta`: berisi informasi pagination, seperti current_page, last_page, dan per_page.

Implementasi menggunakan `paginate($perPage)->withQueryString()` agar query parameter tetap disertakan pada pagination links.

Catatan: hasil custom pagination perlu diperiksa kembali melalui response aktual di Postman sebelum laporan dikumpulkan.

## 6. Bukti Search pada Dua Field

### Search berdasarkan Title

Request:

`GET /api/books?search=Working`

Hasil yang diharapkan adalah `200 OK`. Buku dengan judul yang mengandung kata Working akan muncul jika tersedia di database.

### Search berdasarkan ISBN

Request:

`GET /api/books?search=9780131177055`

Hasil yang diharapkan adalah `200 OK`. Buku akan ditemukan apabila ISBN tersebut tersedia di database.

Jika ISBN contoh tidak tersedia, gunakan ISBN dari buku yang benar-benar tersimpan di database.

Kedua pengujian ini memastikan pencarian bekerja pada field title dan isbn.

## 7. Bukti Sorting

### Sorting Ascending

Request:

`GET /api/books?sort=title&direction=asc`

Hasil yang diharapkan adalah `200 OK`, dengan judul buku diurutkan dari A sampai Z.

### Sorting Descending

Request:

`GET /api/books?sort=title&direction=desc`

Hasil yang diharapkan adalah `200 OK`, dengan judul buku diurutkan dari Z sampai A.

Gunakan minimal tiga buku dengan judul berbeda agar perubahan urutan dapat diperiksa dengan jelas melalui response body Postman.

## 8. Bukti Error 422

Request yang diuji:

`GET http://127.0.0.1:8000/api/books?sort=password`

Header:

`Accept: application/json`

Hasil aktual yang terlihat pada Postman adalah:

`422 Unprocessable Entity`

Response body:

```json
{
  "message": "The selected sort is invalid.",
  "errors": {
    "sort": [
      "The selected sort is invalid."
    ]
  }
}
```

Hasil tersebut membuktikan bahwa parameter sort=password ditolak karena tidak termasuk dalam allowlist sorting.

Pengujian tambahan untuk per_page yang melebihi batas:

`GET /api/books?per_page=51`

Hasil yang diharapkan adalah `422 Unprocessable Entity`, karena nilai maksimum per_page adalah 50. Request ini harus dijalankan di Postman untuk memastikan hasil aktualnya.

## 9. Perbandingan Actual Response dengan API Contract

| Aspek | Ketentuan API Contract | Hasil Pemeriksaan |
|---|---|---|
| Endpoint collection | GET /api/books | Request default berhasil dengan 200 OK |
| Pagination | Menerima page dan per_page | Implementasi menggunakan paginate() |
| Batas per_page | Maksimal 50 | Dibatasi melalui validasi |
| Search | Minimal dua field relevan | Kode mencari pada title dan isbn |
| Sorting | Menggunakan allowlist | Kolom dibatasi pada id, title, isbn, created_at |
| Direction | asc atau desc | Dibatasi melalui validasi |
| Invalid query | Menghasilkan 422 | sort=password terbukti menghasilkan 422 |
| API Resource | Menggunakan Resource collection | Menggunakan BookResource::collection() |
| Pagination links | Mempertahankan query parameter | Menggunakan withQueryString() |

Berdasarkan kode dan pengujian yang sudah dilakukan, validasi invalid sort telah sesuai dengan API contract. Pengujian pagination, search, sorting, kombinasi parameter, dan invalid per_page harus diperiksa melalui hasil aktual Postman untuk memastikan seluruh ketentuan terpenuhi.

## 10. Kesimpulan

Endpoint GET /api/books dikembangkan untuk mendukung pagination, pencarian berdasarkan judul dan ISBN, serta sorting berdasarkan kolom yang diperbolehkan.

Validasi query parameter digunakan untuk membatasi input yang diterima. Pengujian Postman menunjukkan bahwa request default berhasil dengan status 200 OK dan request sort=password ditolak dengan status 422 Unprocessable Entity.

Implementasi menggunakan API Resource untuk membungkus data buku dan pagination Laravel untuk menyediakan metadata serta tautan navigasi.

Sebelum pengumpulan, seluruh request wajib dijalankan dan diperiksa kembali agar dokumentasi sesuai dengan hasil aktual. Collection Postman juga harus diekspor agar dapat diimpor dan dijalankan ulang.

## 11. Referensi

1. Materi 6 — Relasi, Pagination, Filtering, dan Sorting: https://app.notion.com/p/Materi-6-Relasi-Pagination-Filtering-dan-Sorting-28b8bda175f981629712ed30e3c13436?pvs=21
2. Praktikum 6 — Endpoint Collection yang Dapat Dicari: https://app.notion.com/p/Praktikum-6-Endpoint-Collection-yang-Dapat-Dicari-28b8bda175f981d08822e89623abebba?pvs=21
3. API contract project yang terdapat dalam repository praktikum.
4. Laravel Documentation — Validation: https://laravel.com/docs/validation
5. Laravel Documentation — Pagination: https://laravel.com/docs/pagination
6. Laravel Documentation — Eloquent Resources: https://laravel.com/docs/eloquent-resources
7. Postman Documentation — Collections: https://learning.postman.com/docs/collections/using-collections/

## 12. Deklarasi Penggunaan AI

Tool yang digunakan: ChatGPT dan Antigravity IDE.

Tujuan penggunaan AI adalah membantu memahami instruksi praktikum, meninjau implementasi search, sorting, pagination, validasi query parameter, serta menyusun kerangka laporan Markdown.

Ringkasan bantuan yang diminta meliputi penjelasan penempatan kode pada BookController, urutan proses validasi, filter, sorting, pagination, dan API Resource, serta pembuatan contoh request untuk pengujian Postman.