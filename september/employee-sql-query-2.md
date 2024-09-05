# SQL Query API Explanation (2)

SQL Query API didapatkan dari spreadsheet [Mapping Data Portaverse MDM]()

## Table of Contents
- [SQL Query API Explanation (2)](#sql-query-api-explanation-2)
  - [Table of Contents](#table-of-contents)
  - [API 6: Get Item Alamat Rumah Tinggal](#api-6-get-item-alamat-rumah-tinggal)
    - [1. Query Pertama](#1-query-pertama)
      - [Struktur \& Penjelasan](#struktur--penjelasan)
    - [2. Query Kedua](#2-query-kedua)
      - [Struktur dan Penjelasan:](#struktur-dan-penjelasan)
    - [Cara Penggunaan](#cara-penggunaan)
  - [API 7: Get Data Riwayat Jabatan](#api-7-get-data-riwayat-jabatan)
    - [1. Query Pertama](#1-query-pertama-1)
      - [Struktur \& Penjelasan](#struktur--penjelasan-1)
    - [2. Query Kedua](#2-query-kedua-1)
      - [Struktur dan Penjelasan:](#struktur-dan-penjelasan-1)
    - [Cara Penggunaan](#cara-penggunaan-1)
  - [API 8: Get Data Riwayat Penugasan External](#api-8-get-data-riwayat-penugasan-external)
    - [1. Query Pertama](#1-query-pertama-2)
      - [Struktur \& Penjelasan](#struktur--penjelasan-2)
    - [2. Query Kedua](#2-query-kedua-2)
      - [Struktur dan Penjelasan:](#struktur-dan-penjelasan-2)
    - [Cara Penggunaan](#cara-penggunaan-2)

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

#### Struktur dan Penjelasan:

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

#### Struktur dan Penjelasan:

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

#### Struktur dan Penjelasan:

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