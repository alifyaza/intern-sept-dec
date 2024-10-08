# SQL Query API Explanation (2)

SQL Query API didapatkan dari spreadsheet [Mapping Data Portaverse MDM]()

Dokumen ini memberikan penjelasan untuk Query API nomor 6 sampai dengan nomor 10.

## Table of Contents
- [SQL Query API Explanation (2)](#sql-query-api-explanation-2)
  - [Table of Contents](#table-of-contents)
  - [API 6: Get Item Alamat Rumah Tinggal](#api-6-get-item-alamat-rumah-tinggal)
    - [1. Query Pertama](#1-query-pertama)
      - [Struktur \& Penjelasan](#struktur--penjelasan)
    - [2. Query Kedua](#2-query-kedua)
      - [Struktur \& Penjelasan](#struktur--penjelasan-1)
    - [Cara Penggunaan](#cara-penggunaan)
  - [API 7: Get Data Riwayat Jabatan](#api-7-get-data-riwayat-jabatan)
    - [1. Query Pertama](#1-query-pertama-1)
      - [Struktur \& Penjelasan](#struktur--penjelasan-2)
    - [2. Query Kedua](#2-query-kedua-1)
      - [Struktur \& Penjelasan](#struktur--penjelasan-3)
    - [Cara Penggunaan](#cara-penggunaan-1)
  - [API 8: Get Data Riwayat Penugasan External](#api-8-get-data-riwayat-penugasan-external)
    - [1. Query Pertama](#1-query-pertama-2)
      - [Struktur \& Penjelasan](#struktur--penjelasan-4)
    - [2. Query Kedua](#2-query-kedua-2)
      - [Struktur \& Penjelasan](#struktur--penjelasan-5)
    - [Cara Penggunaan](#cara-penggunaan-2)
  - [API 9: Get Data Penugasan Secondary Assignment Pegawai](#api-9-get-data-penugasan-secondary-assignment-pegawai)
    - [1. Query Pertama](#1-query-pertama-3)
      - [Struktur \& Penjelasan](#struktur--penjelasan-6)
        - [1. CTE (WITH tpegawai AS)](#1-cte-with-tpegawai-as)
        - [2. SELECT dari CTE](#2-select-dari-cte)
    - [2. Query Kedua](#2-query-kedua-3)
      - [Struktur \& Penjelasan](#struktur--penjelasan-7)
    - [Cara Penggunaan](#cara-penggunaan-3)
  - [API 10: Get Data Penugasan yang berkaitan dengan Dewan Komisari atau Dewan Pengawas](#api-10-get-data-penugasan-yang-berkaitan-dengan-dewan-komisari-atau-dewan-pengawas)
    - [1. Query Pertama](#1-query-pertama-4)
      - [Struktur \& Penjelasan](#struktur--penjelasan-8)
        - [1. CTE (WITH Clause):](#1-cte-with-clause)
        - [2. Query Utama:](#2-query-utama)
    - [2. Query Kedua](#2-query-kedua-4)
      - [Struktur \& Penjelasan](#struktur--penjelasan-9)
    - [Cara Penggunaan](#cara-penggunaan-4)

## API 6: Get Item Alamat Rumah Tinggal

### 1. Query Pertama

<details>
  <summary>Show SQL Query 1</summary>

```sql
WITH tpegawai AS ( SELECT msp.*, ROW_NUMBER () OVER(PARTITION BY msp.pernr, msp.pnalt_new ORDER BY msp.pernr ASC) AS row_num 
        FROM mdm_single_pegawai msp JOIN (SELECT DISTINCT PERNR FROM SAFM_PEGAWAI_ADDRESSES) spa on msp.PERNR = spa.PERNR  WHERE 1=1 ${appendQueryNipp} ${appendQueryPernr} ${appendQuery} )
        SELECT  msp.PERNR,
        count(*) OVER() AS total_count ,
        msp.PNALT_NEW AS nipp_pegawai,
        msp.cname AS nama_pegawai
        FROM tpegawai msp
        JOIN (SELECT DISTINCT PERNR FROM SAFM_PEGAWAI_ADDRESSES) spa on msp.PERNR = spa.PERNR
        WHERE msp.pnalt_new IS NOT NULL and msp.row_num = 1${appendQueryNipp} ${appendQueryPernr} ${appendQuery}
        ORDER BY msp.LAST_UPDATED_DATE DESC
        OFFSET :offset ROWS FETCH NEXT :per_page ROWS ONLY
```
</details>

**Query pertama** digunakan untuk mendapatkan daftar pegawai yang memiliki informasi alamat rumah tinggal yang tercatat dalam sistem.

#### Struktur & Penjelasan

- Query ini membuat tabel sementara (`tpegawai`) yang berisi data pegawai dari tabel `mdm_single_pegawai` yang dihubungkan dengan tabel `SAFM_PEGAWAI_ADDRESSES` berdasarkan `PERNR` (nomor pegawai).
- Penggunaan `ROW_NUMBER()` berfungsi untuk memberikan nomor urut pada baris yang dipartisi berdasarkan `PERNR` dan `PNALT_NEW`, diurutkan berdasarkan `PERNR` secara ascending.
- Hasil akhir dari query pertama adalah daftar pegawai yang memiliki informasi alamat, ditampilkan bersama dengan `total_count`, `nipp_pegawai`, dan `nama_pegawai`.

### 2. Query Kedua

<details>
  <summary>Show SQL Query 2</summary>

```sql
SELECT PERNR,
            ADDRESS_TYPE_TEXT AS jenis_alamat,
            STREET_AND_HOUSE_NO || ' ' || SECOND_ADDRESS_LINE AS nama_jalan_dan_nomor_rumah,
            DISTRICT AS kecamatan,
            CITY AS kabupaten_kota,
            REGION_TEXT AS provinsi,
            POSTAL_CODE AS kode_pos,
            COUNTRY_TEXT AS negara,
            START_DATE AS tanggal_mulai_tinggal,
            END_DATE AS tanggal_berhenti_tinggal
        FROM SAFM_PEGAWAI_ADDRESSES
        WHERE PERNR IN (${pernrList})
```
</details>

**Query kedua** digunakan untuk mendapatkan detail alamat rumah tinggal pegawai yang diperoleh dari query pertama.

#### Struktur & Penjelasan

- Query ini mengambil informasi detail alamat dari tabel `SAFM_PEGAWAI_ADDRESSES` berdasarkan PERNR yang didapatkan dari query pertama.
- Data yang ditampilkan meliputi jenis alamat, nama jalan dan nomor rumah, kecamatan, kabupaten/kota, provinsi, kode pos, negara, serta tanggal mulai dan berhenti tinggal.

### Cara Penggunaan

- **Query Pertama**:

Hapus `${appendQueryNipp} ${appendQueryPernr} ${appendQuery}`dan Jalankan query pertama untuk mendapatkan daftar pegawai dengan informasi alamat, termasuk `PERNR`, `nipp_pegawai`, dan `nama_pegawai`.

Maka hasil outputnya akan seperti contoh berikut:

| PERNR       | total_count | nipp_pegawai | nama_pegawai |
|-------------|-------------|--------------|--------------|
| 000000284848| 50          | 123456789    | John Doe     |

- **Query Kedua**:

Gunakan `PERNR` yang diperoleh dari query pertama untuk menjalankan query kedua dan mendapatkan detail alamat rumah tinggal.

```sql
...
WHERE PERNR IN (${pernrList})
...
```

Misal user ingin mencari data dengan pernr = '000000284848', maka code diubah menjadi:

```sql
...
WHERE PERNR IN ('000000284848')
...
```

Namun, jika user ingin melakukan inisialisasi setelah eksekusi query. Maka user bisa mengubah value pada bind parameter tab yang tampil. Pada tab tersebut, user diharuskan untuk memasukkan nilai `pernrList` pada tab tersebut dengan tanda ('), contohnya adalah `000000284848`

Maka hasil outputnya akan seperti contoh berikut:

| PERNR        | jenis_alamat   | nama_jalan_dan_nomor_rumah   | kecamatan | kabupaten_kota | provinsi    | kode_pos | negara    | tanggal_mulai_tinggal | tanggal_berhenti_tinggal |
|--------------|----------------|------------------------------|-----------|----------------|-------------|----------|-----------|------------------------|--------------------------|
| 000000284848 | Rumah Tinggal  | Jl. Merdeka No. 1             | Menteng   | Jakarta Pusat  | DKI Jakarta | 10310    | Indonesia | 2020-01-01             | NULL                     |

## API 7: Get Data Riwayat Jabatan

### 1. Query Pertama

<details>
  <summary>Show SQL Query 1</summary>

```sql
WITH tpegawai AS ( SELECT msp.*, ROW_NUMBER () OVER(PARTITION BY msp.pernr, msp.pnalt_new ORDER BY msp.pernr ASC) AS row_num
        FROM mdm_single_pegawai msp JOIN (SELECT DISTINCT PERNR FROM SAFM_PERUBAHAN_ORGANISASI) spo on msp.PERNR = spo.PERNR  WHERE 1=1 ${appendQueryNipp} ${appendQueryPernr} ${appendQuery} )
        SELECT  msp.PERNR,
            count(*) OVER() AS total_count ,
            msp.PNALT_NEW AS nipp_pegawai,
            msp.cname AS nama_pegawai
        FROM tpegawai msp
        JOIN (SELECT DISTINCT PERNR FROM SAFM_PERUBAHAN_ORGANISASI) spo on msp.PERNR = spo.PERNR
        WHERE msp.pnalt_new IS NOT NULL and msp.row_num = 1 ${appendQueryNipp} ${appendQueryPernr} ${appendQuery}
        ORDER BY msp.LAST_UPDATED_DATE DESC
        OFFSET :offset ROWS FETCH NEXT :per_page ROWS ONLY
```
</details>

**Query pertama** digunakan untuk mendapatkan daftar pegawai yang memiliki riwayat perubahan organisasi atau jabatan.

#### Struktur & Penjelasan

- Query ini membuat tabel sementara (`tpegawai`) yang berisi data pegawai dari tabel `mdm_single_pegawai`, yang kemudian dihubungkan dengan tabel `SAFM_PERUBAHAN_ORGANISASI` berdasarkan `PERNR`.
- `ROW_NUMBER()` digunakan untuk memberikan nomor urut pada baris yang dipartisi berdasarkan `PERNR` dan `PNALT_NEW`, diurutkan berdasarkan `PERNR` secara ascending.
- Hasil akhir dari query ini adalah daftar pegawai yang memiliki riwayat perubahan organisasi atau jabatan, termasuk `total_count`, `nipp_pegawai`, dan `nama_pegawai`.
  
### 2. Query Kedua

<details>
  <summary>Show SQL Query 2</summary>

```sql
Query pertama : 

WITH tpegawai AS ( SELECT msp.*, ROW_NUMBER () OVER(PARTITION BY msp.pernr, msp.pnalt_new ORDER BY msp.pernr ASC) AS row_num
        FROM mdm_single_pegawai msp JOIN (SELECT DISTINCT PERNR FROM SAFM_PERUBAHAN_ORGANISASI) spo on msp.PERNR = spo.PERNR  WHERE 1=1 ${appendQueryNipp} ${appendQueryPernr} ${appendQuery} )
        SELECT  msp.PERNR,
            count(*) OVER() AS total_count ,
            msp.PNALT_NEW AS nipp_pegawai,
            msp.cname AS nama_pegawai
        FROM tpegawai msp
        JOIN (SELECT DISTINCT PERNR FROM SAFM_PERUBAHAN_ORGANISASI) spo on msp.PERNR = spo.PERNR
        WHERE msp.pnalt_new IS NOT NULL and msp.row_num = 1 ${appendQueryNipp} ${appendQueryPernr} ${appendQuery}
        ORDER BY msp.LAST_UPDATED_DATE DESC
        OFFSET :offset ROWS FETCH NEXT :per_page ROWS ONLY
```
</details>

**Query kedua** digunakan untuk mendapatkan detail alamat rumah tinggal pegawai yang diperoleh dari query pertama.

#### Struktur & Penjelasan

- Query ini mengambil data dari tabel `SAFM_PERUBAHAN_ORGANISASI` dan menggabungkannya dengan tabel `SAFM_STRUKTUR_ORGANISASI` untuk mendapatkan informasi mengenai jabatan, kategori jabatan, dan organisasi.
- Selain itu, query juga menggabungkan data dengan tabel `SAFM_PEGAWAI_ACTION` untuk mendapatkan informasi terkait surat keterangan dan tanggal-tanggal penting seperti tanggal mulai dan selesai menjabat.
- Data juga digabungkan dengan tabel `SAFR_COMPANY` untuk mendapatkan informasi perusahaan tempat pegawai bekerja.
- Hasil akhirnya adalah detail dari setiap jabatan yang pernah diemban pegawai, termasuk informasi mengenai jabatan, kategori jabatan, organisasi, perusahaan, nomor surat keterangan, dan jenis mutasi.

### Cara Penggunaan

- **Query Pertama**:

Hapus `${appendQueryNipp} ${appendQueryPernr} ${appendQuery}`dan  Jalankan query pertama untuk mendapatkan daftar pegawai yang memiliki riwayat perubahan jabatan.

Maka hasil outputnya akan seperti contoh berikut:

| PERNR       | total_count | nipp_pegawai | nama_pegawai |
|-------------|-------------|--------------|--------------|
| 000000284848| 50          | 123456789    | John Doe     |

- **Query Kedua**:

Gunakan PERNR dari hasil query pertama untuk menjalankan query kedua dan mendapatkan detail riwayat jabatan.

```sql
...
WHERE PERNR IN (${pernrList})
...
```

Misal user ingin mencari data dengan pernr = '000000284848', maka code diubah menjadi:

```sql
...
WHERE PERNR IN ('000000284848')
...
```

Namun, jika user ingin melakukan inisialisasi setelah eksekusi query. Maka user bisa mengubah value pada bind parameter tab yang tampil. Pada tab tersebut, user diharuskan untuk memasukkan nilai `pernrList` pada tab tersebut dengan tanda ('), contohnya adalah `000000284848`

Maka hasil outputnya akan seperti contoh berikut:

| PERNR        | id_master_posisi | nama_jabatan      | kategori_jabatan  | fungsi_jabatan | kelas_jabatan | organisasi    | perusahaan  | nomor_surat_keterangan | tanggal_surat_keterangan | tanggal_mulai_menjabat | tanggal_selesai_menjabat | jenis_mutasi    | deskripsi |
|--------------|------------------|-------------------|-------------------|----------------|---------------|---------------|-------------|------------------------|--------------------------|------------------------|--------------------------|-----------------|-----------|
| 000000284848 | 1001             | Manajer Proyek    | Manajemen         |                | KJ-3          | IT Department | PT ABC      | SK/123/HRD             | 2024-01-15               | 2024-01-15             | 2024-12-31               | Promosi         |           |
| 000000284849 | 1002             | Analis Bisnis     | Operasional       |                | KJ-4          | Finance       | PT XYZ      | SK/124/HRD             | 2023-05-20               | 2023-05-20             | 2023-11-30               | Rotasi          |           |
| 000000284850 | 1003             | Supervisor        | Operasional       |                | KJ-2          | HR Department | PT DEF      | SK/125/HRD             | 2022-02-10               | 2022-02-10             | 2023-01-15               | Penurunan Jabatan |           |

## API 8: Get Data Riwayat Penugasan External

### 1. Query Pertama

<details>
  <summary>Show SQL Query 1</summary>

```sql
WITH tpegawai AS ( SELECT msp.*, ROW_NUMBER () OVER(PARTITION BY msp.pernr, msp.pnalt_new ORDER BY msp.pernr ASC) AS row_num
        FROM mdm_single_pegawai msp JOIN (SELECT DISTINCT PERNR FROM SAFM_PEGAWAI_SECOND_ASSIGNMENT) spsa on msp.PERNR = spsa.PERNR  WHERE 1=1 ${appendQueryNipp} ${appendQueryPernr} ${appendQuery} )
        SELECT  msp.PERNR,
          count(*) OVER() AS total_count ,
          msp.PNALT_NEW AS nipp_pegawai,
          msp.cname AS nama_pegawai
        FROM tpegawai msp
        JOIN (SELECT DISTINCT PERNR FROM SAFM_PEGAWAI_SECOND_ASSIGNMENT) spsa on msp.PERNR = spsa.PERNR
        WHERE msp.pnalt_new IS NOT NULL and msp.row_num = 1 ${appendQueryNipp} ${appendQueryPernr} ${appendQuery}
        ORDER BY msp.LAST_UPDATED_DATE DESC
        OFFSET :offset ROWS FETCH NEXT :per_page ROWS ONLY
```
</details>

**Query pertama** digunakan untuk Mengambil daftar pegawai yang memiliki penugasan eksternal dari tabel SAFM_PEGAWAI_SECOND_ASSIGNMENT.

#### Struktur & Penjelasan

- `CTE (WITH tpegawai AS)`: Membuat tabel sementara yang berisi data pegawai dari tabel `mdm_single_pegawai`, dengan nomor urut (`ROW_NUMBER()`) yang dipartisi berdasarkan PERNR dan PNALT_NEW, diurutkan secara ascending berdasarkan PERNR.
- Data pegawai yang diambil kemudian digabungkan dengan tabel `SAFM_PEGAWAI_SECOND_ASSIGNMENT` untuk memastikan hanya pegawai yang memiliki penugasan eksternal yang diambil.
- Hasil dari query ini adalah daftar pegawai, termasuk total_count, nipp_pegawai, dan nama_pegawai, yang terurut berdasarkan `LAST_UPDATED_DATE` dan dibatasi dengan offset serta jumlah baris per halaman.
  
### 2. Query Kedua

<details>
  <summary>Show SQL Query 2</summary>

```sql
SELECT PERNR,
            MGTXT AS nama_penugasan,
            NAMA_JABATAN AS jabatan_penugasan,
            DOC_NUMBER AS nomor_surat_penugasan,
            '' AS surat_penugasan,
            NVL(EFFECTIVE_DATE, BEGDA) AS tanggal_mulai_penugasan,
            ENDDA AS tanggal_selesai_penugasan,
            '' AS deskripsi
        FROM SAFM_PEGAWAI_SECOND_ASSIGNMENT
        WHERE MASSG = 'Z008' AND PERNR IN (${pernrList})
```
</details>

**Query kedua** digunakan untuk mendapatkan detail penugasan eksternal untuk pegawai yang ditemukan pada query pertama.

#### Struktur & Penjelasan

Mengambil data dari tabel SAFM_PEGAWAI_SECOND_ASSIGNMENT untuk setiap pegawai (PERNR) yang terdaftar pada query pertama.
Data yang diambil mencakup informasi tentang penugasan eksternal, termasuk nama penugasan, jabatan penugasan, nomor surat penugasan, tanggal mulai dan selesai penugasan, serta deskripsi (meskipun kolom deskripsi tidak diisi).
Filter MASSG = 'Z008' memastikan hanya penugasan dengan kode tertentu yang diambil.

### Cara Penggunaan

- **Query Pertama**:

Hapus `${appendQueryNipp} ${appendQueryPernr} ${appendQuery}`dan  Jalankan query pertama untuk mendapatkan daftar pegawai yang memiliki riwayat perubahan jabatan.

Maka hasil outputnya akan seperti contoh berikut:

| PERNR       | total_count | nipp_pegawai | nama_pegawai |
|-------------|-------------|--------------|--------------|
| 000000284848| 50          | 123456789    | John Doe     |

- **Query Kedua**:

Gunakan PERNR dari hasil query pertama untuk menjalankan query kedua dan mendapatkan detail riwayat jabatan.

```sql
...
WHERE PERNR IN (${pernrList})
...
```

Misal user ingin mencari data dengan pernr = '000000284848', maka code diubah menjadi:

```sql
...
WHERE PERNR IN ('000000284848')
...
```

Namun, jika user ingin melakukan inisialisasi setelah eksekusi query. Maka user bisa mengubah value pada bind parameter tab yang tampil. Pada tab tersebut, user diharuskan untuk memasukkan nilai `pernrList` pada tab tersebut dengan tanda ('), contohnya adalah `000000284848`

Maka hasil outputnya akan seperti contoh berikut:

| PERNR        | id_master_posisi | nama_jabatan      | kategori_jabatan  | fungsi_jabatan | kelas_jabatan | organisasi    | perusahaan  | nomor_surat_keterangan | tanggal_surat_keterangan | tanggal_mulai_menjabat | tanggal_selesai_menjabat | jenis_mutasi    | deskripsi |
|--------------|------------------|-------------------|-------------------|----------------|---------------|---------------|-------------|------------------------|--------------------------|------------------------|--------------------------|-----------------|-----------|
| 000000284848 | 1001             | Manajer Proyek    | Manajemen         |                | KJ-3          | IT Department | PT ABC      | SK/123/HRD             | 2024-01-15               | 2024-01-15             | 2024-12-31               | Promosi         |           |
| 000000284849 | 1002             | Analis Bisnis     | Operasional       |                | KJ-4          | Finance       | PT XYZ      | SK/124/HRD             | 2023-05-20               | 2023-05-20             | 2023-11-30               | Rotasi          |           |
| 000000284850 | 1003             | Supervisor        | Operasional       |                | KJ-2          | HR Department | PT DEF      | SK/125/HRD             | 2022-02-10               | 2022-02-10             | 2023-01-15               | Penurunan Jabatan |           |

## API 9: Get Data Penugasan Secondary Assignment Pegawai

### 1. Query Pertama

<details>
  <summary>Show SQL Query 1</summary>

```sql
WITH tpegawai AS ( SELECT msp.*, ROW_NUMBER () OVER(PARTITION BY msp.pernr, msp.pnalt_new ORDER BY msp.pernr ASC) AS row_num 
        FROM mdm_single_pegawai msp JOIN (SELECT DISTINCT PERNR FROM SAFM_PEGAWAI_ADDRESSES) spa on msp.PERNR = spa.PERNR  WHERE 1=1 ${appendQueryNipp} ${appendQueryPernr} ${appendQuery} )
        SELECT  msp.PERNR,
        count(*) OVER() AS total_count ,
        msp.PNALT_NEW AS nipp_pegawai,
        msp.cname AS nama_pegawai
        FROM tpegawai msp
        JOIN (SELECT DISTINCT PERNR FROM SAFM_PEGAWAI_ADDRESSES) spa on msp.PERNR = spa.PERNR
        WHERE msp.pnalt_new IS NOT NULL and msp.row_num = 1${appendQueryNipp} ${appendQueryPernr} ${appendQuery}
        ORDER BY msp.LAST_UPDATED_DATE DESC
        OFFSET :offset ROWS FETCH NEXT :per_page ROWS ONLY
```
</details>

**Query pertama** digunakan untuk mengambil daftar pegawai yang memiliki penugasan kedua dan mengurutkan data tersebut berdasarkan `pernr` (nomor pegawai) serta `pnalt_new` (nomor identifikasi pegawai lainnya), lalu mengelompokkan data tersebut untuk memastikan hanya satu entri per pegawai yang dipilih (berdasarkan `row_num` = 1).

#### Struktur & Penjelasan

##### 1. CTE (WITH tpegawai AS)

- CTE ini bernama `tpegawai`, dan di dalamnya, kita mengambil data dari tabel `mdm_single_pegawai` (msp).
- Menggunakan `ROW_NUMBER()` untuk memberikan nomor urut pada setiap baris yang dikelompokkan berdasarkan `pernr` (nomor pegawai) dan `pnalt_new` (nomor identifikasi lainnya). Ini memastikan bahwa setiap pegawai hanya memiliki satu baris data yang diambil, yaitu yang memiliki nomor urut (`row_num`) terkecil.
- `JOIN` dilakukan dengan tabel `SAFM_PEGAWAI_SECOND_ASSIGNMENT` (`spsa`) untuk memilih hanya pegawai yang memiliki penugasan kedua. Hanya pegawai yang ada dalam tabel ini yang akan diambil dari mdm_single_pegawai.

##### 2. SELECT dari CTE

- Data yang diambil meliputi nomor pegawai (`PERNR`), nomor identifikasi lainnya (`PNALT_NEW`), nama pegawai (`cname`), dan total jumlah pegawai yang memenuhi syarat filter (`total_count`).
- Menggunakan filter `WHERE msp.pnalt_new IS NOT NULL` and `msp.row_num = 1` untuk memastikan hanya satu baris data per pegawai yang diambil, yaitu data dengan nomor urut pertama.
- Data diurutkan berdasarkan `LAST_UPDATED_DATE` secara menurun (`DESC`), sehingga data yang paling baru akan muncul terlebih dahulu.
Pagination (`OFFSET :offset ROWS FETCH NEXT :per_page ROWS ONLY`) digunakan untuk membatasi jumlah data yang ditampilkan per halaman.

### 2. Query Kedua

<details>
  <summary>Show SQL Query 2</summary>

```sql
SELECT PERNR,
            spsa.SOBID AS id_master_posisi,
            spsa.NAMA_JABATAN AS nama_jabatan,
            COALESCE(spsa.PKTXT, spsa.PGTXT) AS kategori_jabatan,
            '' AS fungsi_jabatan,
            '' AS kelas_jabatan,
            spsa.NAMA_ORG AS organisasi,
            (SELECT sc.comp_name FROM SAFR_COMPANY sc WHERE spsa.COMPANY_CODE = sc.comp_code) AS perusahaan,
            spsa.doc_number AS nomor_surat_penugasan,
            spsa.EFFECTIVE_DATE AS tanggal_surat_penugasan,
            '' AS surat_penugasan,
            NVL(spsa.EFFECTIVE_DATE, spsa.BEGDA) AS tanggal_mulai_menjabat,
            spsa.ENDDA AS tanggal_selesai_menjabat,
            spsa.MGTXT AS jenis_secondary_assignment,
            '' AS deskripsi
        FROM SAFM_PEGAWAI_SECOND_ASSIGNMENT spsa
        WHERE MASSG IN ('Z006', 'Z007', 'Z009', 'Z010') AND PERNR IN (${pernrList})
```
</details>

**Query kedua** ini mengambil detail lebih lanjut mengenai penugasan kedua untuk pegawai yang telah dipilih di query pertama.

#### Struktur & Penjelasan

- Data yang diambil termasuk nomor pegawai (`PERNR`), ID master posisi (`SOBID`), nama jabatan (`NAMA_JABATAN`), dan informasi lain terkait posisi dan organisasi pegawai.
- COALESCE digunakan untuk memilih antara dua kolom (PKTXT dan PGTXT) mana yang memiliki nilai, dan digunakan sebagai kategori_jabatan.
- Nama perusahaan (perusahaan) diambil dari tabel `SAFR_COMPANY` yang cocok dengan `COMPANY_CODE` di spsa.
- NVL digunakan untuk memilih antara `EFFECTIVE_DATE` dan `BEGDA` untuk mendapatkan tanggal mulai menjabat (tanggal_mulai_menjabat).
- Data disaring (`WHERE MASSG IN ('Z006', 'Z007', 'Z009', 'Z010')`) untuk mengambil hanya penugasan kedua dengan tipe tertentu (`MASSG`), dan hanya untuk pegawai yang nomor pegawainya ada di dalam pernrList.

### Cara Penggunaan

- **Query Pertama**:

1. **Filter Data (Optional)**: Gunakan variabel appendQueryNipp, appendQueryPernr, dan appendQuery untuk menyaring data pegawai sesuai kebutuhan (misalnya, berdasarkan departemen atau jabatan). Namun, user bisa menghapus bagian ini jika tidak diperlukan.
2. **Pagination**: Gunakan `:offset` dan `:per_page` untuk mengatur jumlah data yang ditampilkan per halaman. Misalnya, jika Anda ingin menampilkan 10 data per halaman, Anda dapat mengatur `:per_page = 10` dan `:offset` sesuai dengan halaman yang ingin ditampilkan.
3. **Eksekusi Query**: Jalankan query untuk mendapatkan daftar pegawai dengan penugasan kedua. Hasilnya dapat ditampilkan dalam tabel atau laporan.

Maka hasil outputnya akan seperti contoh berikut:

| PERNR       | total_count | nipp_pegawai | nama_pegawai |
|-------------|-------------|--------------|--------------|
| 000000284848| 50          | 123456789    | John Doe     |

- **Query Kedua**:

1. **Pilih Pegawai**: Gunakan PERNR dari hasil query pertama untuk memilih pegawai yang ingin ditelusuri lebih lanjut.
2. **Tentukan MASSG**: Filter data dengan nilai MASSG yang sesuai ('Z006', 'Z007', 'Z009', 'Z010') untuk mendapatkan tipe penugasan yang relevan.
3. **Isi pernrList**: Masukkan nomor pegawai tersebut ke dalam variabel `${pernrList}`.
   Gunakan `PERNR` yang diperoleh dari query pertama untuk menjalankan query kedua dan mendapatkan detail list pegawai yang mendapatkan penugasan secondary assignment.

    ```sql
    ...
    WHERE PERNR IN (${pernrList})
    ...
    ```

    Misal user ingin mencari data dengan `pernr = '000000284848'`, maka code diubah menjadi:

    ```sql
    ...
    WHERE PERNR IN ('000000284848')
    ...
    ```

    Namun, jika user ingin melakukan inisialisasi setelah eksekusi query. Maka user bisa mengubah value pada bind parameter tab yang tampil. Pada tab tersebut, user diharuskan untuk memasukkan nilai `pernrList` pada tab tersebut dengan tanda ('), contohnya adalah `'000000284848'`
    
4. **Eksekusi Query**: Jalankan query untuk mendapatkan detail penugasan kedua, seperti jabatan, tanggal efektif, dan perusahaan terkait. Hasilnya dapat ditampilkan dalam laporan detail.

Maka hasil outputnya akan seperti contoh berikut:

| PERNR  | ID Master Posisi | Nama Jabatan          | Kategori Jabatan | Fungsi Jabatan | Kelas Jabatan | Organisasi        | Perusahaan        | Nomor Surat Penugasan | Tanggal Surat Penugasan | Surat Penugasan | Tanggal Mulai Menjabat | Tanggal Selesai Menjabat | Jenis Secondary Assignment | Deskripsi |
|--------|------------------|----------------------|------------------|----------------|---------------|-------------------|-------------------|-----------------------|-------------------------|-----------------|------------------------|--------------------------|----------------------------|-----------|
| 123456 | SOB123            | Manager IT           | Teknologi        |                |               | Divisi Teknologi  | PT XYZ             | 001/IT/2023            | 2023-01-10              |                 | 2023-01-15             | 2024-01-15               | Penugasan Sementara         |           |

## API 10: Get Data Penugasan yang berkaitan dengan Dewan Komisari atau Dewan Pengawas

### 1. Query Pertama

<details>
  <summary>Show SQL Query 1</summary>

```sql
WITH tpegawai AS ( SELECT msp.*, ROW_NUMBER () OVER(PARTITION BY msp.pernr, msp.pnalt_new ORDER BY msp.pernr ASC) AS row_num
        FROM mdm_single_pegawai msp JOIN (SELECT DISTINCT PERNR FROM SAFM_PEGAWAI_FAMILY ) spf ON msp.PERNR = spf.PERNR  WHERE 1=1 ${appendQueryNipp} ${appendQueryPernr} ${appendQuery} )
        SELECT  msp.PERNR,
          count(*) OVER() AS total_count ,
          msp.PNALT_NEW AS nipp_pegawai,
          msp.cname AS nama_pegawai
        FROM tpegawai msp
        JOIN (SELECT DISTINCT PERNR FROM SAFM_PEGAWAI_SECOND_ASSIGNMENT) spsa on msp.PERNR = spsa.PERNR
        WHERE msp.pnalt_new IS NOT NULL and msp.row_num = 1 ${appendQueryNipp} ${appendQueryPernr} ${appendQuery}
        ORDER BY msp.LAST_UPDATED_DATE DESC
        OFFSET :offset ROWS FETCH NEXT :per_page ROWS ONLY
```
</details>

`Query pertama` bertujuan untuk mendapatkan daftar pegawai yang memiliki penugasan terkait dengan Dewan Komisaris atau Dewan Pengawas. Ini dilakukan dengan membangun daftar pegawai dari tabel `mdm_single_pegawai` yang juga terdapat di dalam tabel `SAFM_PEGAWAI_FAMILY`.

#### Struktur & Penjelasan

##### 1. CTE (WITH Clause):

- `CTE` (Common Table Expression) digunakan untuk menyusun data pegawai dari tabel `mdm_single_pegawai` (`msp`) yang juga memiliki data di tabel `SAFM_PEGAWAI_FAMILY` (`spf`). Query ini berguna untuk mempersiapkan data sementara sebelum digunakan dalam query utama.
- `JOIN` dilakukan antara `mdm_single_pegawai` (`msp`) dan `SAFM_PEGAWAI_FAMILY` (`spf`) pada kolom `PERNR` (nomor pegawai) untuk memastikan hanya pegawai yang memiliki data di kedua tabel tersebut yang akan diambil.
- Fungsi `ROW_NUMBER()` digunakan untuk menambahkan nomor urut pada setiap pegawai dalam grup yang sama (`PARTITION BY msp.pernr, msp.pnalt_new`). Ini penting untuk mengidentifikasi satu baris unik per pegawai, yang kemudian difilter dalam bagian berikutnya.

##### 2. Query Utama:

- Query utama mengambil kolom-kolom yang dibutuhkan seperti `PERNR`, `PNALT_NEW` (NIPP pegawai), dan `cname` (nama pegawai).
- Kolom `count(*) OVER() AS total_count` digunakan untuk menghitung jumlah total baris yang sesuai dengan filter, tanpa mempertimbangkan pagination.
- Hanya baris dengan `row_num = 1` yang akan diambil, artinya hanya satu baris per pegawai (yang muncul pertama kali berdasarkan urutan yang ditentukan).
- Query melakukan `JOIN` dengan tabel `SAFM_PEGAWAI_SECOND_ASSIGNMENT` (`spsa`) untuk memastikan hanya pegawai yang memiliki penugasan kedua yang akan diambil.
- Data kemudian diurutkan berdasarkan `LAST_UPDATED_DATE` secara menurun (`DESC`) dan hasilnya diambil dengan teknik pagination menggunakan `OFFSET` dan `FETCH NEXT` untuk membatasi jumlah baris yang diambil.
  
### 2. Query Kedua

<details>
  <summary>Show SQL Query 2</summary>

```sql
SELECT PERNR,
            MGTXT AS nama_penugasan,
            NAMA_JABATAN AS jabatan_penugasan,
            DOC_NUMBER AS nomor_surat_penugasan,
            EFFECTIVE_DATE AS tanggal_surat_penugasan,
            '' AS surat_penugasan,
            NVL(EFFECTIVE_DATE, BEGDA) AS tanggal_mulai_penugasan,
            ENDDA AS tanggal_selesai_penugasan,
            '' AS deskripsi
        FROM SAFM_PEGAWAI_SECOND_ASSIGNMENT
        WHERE MASSG IN ('Z011', 'Z012', 'Z013', 'Z014') AND PERNR IN (${pernrList})
```
</details>

**Query kedua** bertujuan untuk mengambil detail penugasan kedua dari pegawai yang terpilih, dengan fokus pada penugasan yang terkait dengan Dewan Komisaris atau Dewan Pengawas.

#### Struktur & Penjelasan

- Query ini mengambil data dari tabel SAFM_PEGAWAI_SECOND_ASSIGNMENT, yang menyimpan informasi tentang penugasan kedua pegawai.
- Kolom yang Diambil:
  - `PERNR`: Nomor pegawai.
  - `MGTXT` (`nama_penugasan`): Nama penugasan yang berhubungan dengan Dewan Komisaris atau Dewan Pengawas.
  - `NAMA_JABATAN` (`jabatan_penugasan`): Nama jabatan yang dipegang dalam penugasan tersebut.
  - `DOC_NUMBER` (`nomor_surat_penugasan`): Nomor surat penugasan yang terkait dengan penugasan ini.
  - `EFFECTIVE_DATE` (`tanggal_surat_penugasan`): Tanggal surat penugasan dibuat.
  - `BEGDA` dan `EFFECTIVE_DATE` (`tanggal_mulai_penugasan`): Tanggal mulai pegawai mulai menjabat posisi tersebut.
  - `ENDDA` (`tanggal_selesai_penugasan`): Tanggal akhir dari penugasan.
  - Deskripsi: Detail tambahan, jika ada.
- Query ini difilter dengan kondisi `MASSG` IN ('Z011', 'Z012', 'Z013', 'Z014') untuk memastikan hanya penugasan yang terkait dengan kode ini yang akan diambil. Kode-kode ini biasanya merepresentasikan berbagai jenis penugasan yang berhubungan dengan Dewan Komisaris atau Dewan Pengawas.
- Query ini juga difilter berdasarkan `PERNR`, yang didapatkan dari hasil query pertama. Hanya pegawai yang nomor pegawainya ada di dalam daftar ini (`pernrList`) yang akan ditampilkan.

### Cara Penggunaan

- **Query Pertama**:

  1. **Filter Data (Optional)**: Gunakan variabel appendQueryNipp, appendQueryPernr, dan appendQuery untuk menyaring data pegawai sesuai kebutuhan (misalnya, berdasarkan departemen atau jabatan). Namun, user bisa menghapus bagian ini jika tidak diperlukan.
  2. **Pagination**: Gunakan `:offset` dan `:per_page` untuk mengatur jumlah data yang ditampilkan per halaman. Misalnya, jika Anda ingin menampilkan 10 data per halaman, Anda dapat mengatur `:per_page = 10` dan `:offset` sesuai dengan halaman yang ingin ditampilkan.
  3. **Eksekusi Query Pertama**: Jalankan query pertama untuk mendapatkan daftar pegawai yang memiliki penugasan.

Maka hasil outputnya akan seperti contoh berikut:

| PERNR       | total_count | nipp_pegawai | nama_pegawai |
|-------------|-------------|--------------|--------------|
| 000000284848| 50          | 123456789    | John Doe     |

- **Query Kedua**:

  1. **Pilih Pegawai**: Gunakan PERNR dari hasil query pertama untuk memilih pegawai yang ingin ditelusuri lebih lanjut.
  2. **Tentukan MASSG**: Filter data dengan nilai MASSG yang sesuai ('Z006', 'Z007', 'Z009', 'Z010') untuk mendapatkan tipe penugasan terkait dengan Dewan Komisaris atau Dewan Pengawas 
  3. **Isi pernrList**: Masukkan nomor pegawai tersebut ke dalam variabel `${pernrList}`.
  
  Gunakan `PERNR` yang diperoleh dari query pertama untuk menjalankan query kedua dan mendapatkan detail list pegawai yang mendapatkan penugasan yang berkaitan dengan Dewan Komisari atau Dewan Pengawas.

    ```sql
    ...
    WHERE PERNR IN (${pernrList})
    ...
    ```

    Misal user ingin mencari data dengan `pernr = '000000284848'`, maka code diubah menjadi:

    ```sql
    ...
    WHERE PERNR IN ('000000284848')
    ...
    ```

    Namun, jika user ingin melakukan inisialisasi setelah eksekusi query. Maka user bisa mengubah value pada bind parameter tab yang tampil. Pada tab tersebut, user diharuskan untuk memasukkan nilai `pernrList` pada tab tersebut dengan tanda ('), contohnya adalah `'000000284848'`
    
   4. **Eksekusi Query**: Jalankan query kedua untuk mendapatkan detail penugasan Dewan Komisaris atau Dewan Pengawas.

Maka hasil outputnya akan seperti contoh berikut:

| PERNR      | NIPP Pegawai | Nama Pegawai | Nama Penugasan   | Jabatan Penugasan  | Nomor Surat Penugasan | Tanggal Surat Penugasan | Surat Penugasan         | Tanggal Mulai Penugasan | Tanggal Selesai Penugasan | Lama Penugasan | Deskripsi |
|------------|--------------|--------------|------------------|--------------------|-----------------------|-------------------------|-------------------------|--------------------------|----------------------------|----------------|-----------|
| 000000284848 | 123456789  | John Doe     | Dewan Komisaris  | Komisaris Utama     | SP/001                | 2024-04-25              | Surat Penugasan No. SP/001 | 2024-05-01               | 2024-12-31                 | 8 bulan        |           |
| 000000284848 | 123456789  | John Doe     | Dewan Pengawas   | Anggota Dewan Pengawas | SP/002              | 2024-04-28              | Surat Penugasan No. SP/002 | 2024-05-01               | Sampai saat ini           | Sampai saat ini |           |
