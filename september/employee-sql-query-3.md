# SQL Query API Explanation (2)

SQL Query API didapatkan dari spreadsheet [Mapping Data Portaverse MDM]()

Dokumen ini memberikan penjelasan untuk Query API nomor 11 sampai dengan nomor 13.

## Table of Contents
- [SQL Query API Explanation (2)](#sql-query-api-explanation-2)
  - [Table of Contents](#table-of-contents)
  - [API 11: Get Data Master Posisi](#api-11-get-data-master-posisi)
    - [Penjelasan](#penjelasan)
    - [Cara Kerja](#cara-kerja)
    - [Cara Penggunaan](#cara-penggunaan)
  - [API 12: Get Data Struktur Organisasi](#api-12-get-data-struktur-organisasi)
    - [Penjelasan](#penjelasan-1)
      - [1. CTE `t1`](#1-cte-t1)
      - [2. Query Utama](#2-query-utama)
    - [Cara Penggunaan](#cara-penggunaan-1)
  - [API 13: Get Data Perusahaan](#api-13-get-data-perusahaan)
    - [Penjelasan](#penjelasan-2)
    - [Cara Kerja](#cara-kerja-1)
    - [Cara Penggunaan](#cara-penggunaan-2)

## API 11: Get Data Master Posisi

<details>
  <summary>Show SQL Query</summary>

```sql
SELECT DISTINCT count(*) OVER() AS total_count,
        OBJID AS id ,
        JOB_NAME  AS nama_posisi ,
        KJ_POSISI_NEW AS kelas_jabatan_posisi ,
        '' nilai_jabatan,
        '' fulltime_equivalent,
        '' status_job_assignment,
        COALESCE(SUBDI_LVL_5_TEXT, SUBDI_LVL_4_TEXT, SUBDI_LVL_3_TEXT,SUBDI_LVL_2_TEXT, SUBDI_LVL_1_TEXT) AS unit_kerja,
        '' level_urgensi_posisi,
        PERSG_TEXT AS tipe_posisi,
        --TO_CHAR(BEGDA, 'YYYY-MM-DD') AS waktu_mulai,
        BEGDA AS waktu_mulai,
        ENDDA AS waktu_selesai,
        company_code
        FROM SAFM_STRUKTUR_JABATAN 
        WHERE 1=1  ${appendQuery} 
      OFFSET :offset ROWS FETCH NEXT :per_page ROWS ONLY"
```
</details>

### Penjelasan

Query ini bertujuan untuk mengambil data Master Posisi dari tabel `SAFM_STRUKTUR_JABATAN`. Data yang diambil mencakup informasi terkait posisi jabatan, seperti nama posisi, kelas jabatan, unit kerja, tipe posisi, dan periode aktifnya. Query ini juga mendukung pagination dengan penggunaan OFFSET dan FETCH NEXT untuk mengambil data secara bertahap.

### Cara Kerja

- `SELECT DISTINCT count(*) OVER() AS total_count` digunakan untuk Menghitung total jumlah baris yang memenuhi kriteria, tanpa memperhatikan pagination, sehingga hasilnya tidak dipengaruhi oleh OFFSET dan FETCH NEXT. Hasil ini akan disertakan dalam setiap baris yang dikembalikan oleh query.

- **Pemilihan Kolom**:
  - `OBJID AS id`: Mengambil `OBJID` sebagai ID dari posisi.
  - `JOB_NAME AS nama_posisi`: Mengambil nama posisi dari `JOB_NAME`.
  - `KJ_POSISI_NEW AS kelas_jabatan_posisi`: Mengambil kelas jabatan dari KJ_POSISI_NEW.
  - Kolom lainnya seperti nilai_jabatan, fulltime_equivalent, dan `status_job_assignment` diinisialisasi dengan nilai kosong ('').
  - **Unit Kerja**: Mengambil nama unit kerja dari salah satu level struktur organisasi yang tersedia, menggunakan `COALESCE` untuk memilih teks yang tidak null dari level paling rendah hingga paling tinggi.
  - `PERSG_TEXT AS tipe_posisi`: Mengambil tipe posisi dari `PERSG_TEXT`.
  - `Periode Jabatan`: Mengambil `BEGDA` sebagai waktu_mulai dan ENDDA sebagai waktu_selesai.
  - `company_code`: Mengambil kode perusahaan.
  - `WHERE 1=1 ${appendQuery}`: Kondisi ini memungkinkan penambahan kriteria filter dinamis dengan menggunakan variabel `${appendQuery}`. Misalnya, filter berdasarkan company_code.

- **Pagination**:
  - `OFFSET :offset ROWS`: Mengabaikan sejumlah baris pertama sesuai nilai `offset`.
  - `FETCH NEXT :per_page ROWS ONLY`: Mengambil sejumlah baris yang ditentukan oleh `per_page`.

### Cara Penggunaan

Query ini digunakan dalam API untuk mendapatkan data posisi dengan pagination, filter, dan data spesifik yang diminta oleh user. Permintaan ini dikirim melalui request curl, yang mengirimkan parameter seperti page, per_page, dan company_code untuk menentukan halaman dan jumlah data yang diambil serta menyaring data berdasarkan kode perusahaan. User hanya perlu menginput value untuk offset dan per_page.

Berikut contoh hasil outputnya:

| id   | nama_posisi                        | kelas_jabatan_posisi | nilai_jabatan | fulltime_equivalent | status_job_assignment | unit_kerja               | level_urgensi_posisi | tipe_posisi | waktu_mulai | waktu_selesai | company_code |
|------|------------------------------------|----------------------|---------------|---------------------|-----------------------|--------------------------|----------------------|-------------|-------------|---------------|--------------|
| 1273 | Junior Administrator Layanan Umum  |                      |               |                     | Aktif                | Departemen Layanan Umum  |                      |             | 2024-05-15  | 2024-12-31    | 7100         |

## API 12: Get Data Struktur Organisasi

<details>
  <summary>Show SQL Query</summary>

```sql
WITH t1 AS (
          SELECT DISTINCT
             OBJID AS id_master_group ,
             company_code AS id_master_perusahaan_terkait,
             PARID AS id_parent_group ,
             STEXT AS nama_group ,
             LEVELORGANISASI AS level_group ,
             DESCBOBOTORGANISASI  AS tipe_group ,
             BEGDA AS waktu_mulai,
             ENDDA AS waktu_selesai,
             LAST_UPDATED_DATE
             FROM SAFM_STRUKTUR_ORGANISASI a  WHERE  a.OTYPE = 'O' AND TRUNC(SYSDATE) BETWEEN TRUNC(a.begda) AND TRUNC(a.endda) ${appendQuery}
             AND (
               a.COMPANY_CODE IN (
                 SELECT DISTINCT z.COMPANY_CODE FROM SAFR_COMPANY_D z 
                 WHERE 
                   z.STATUS = 1
                   AND Z.IS_EXCLUDE = 0
                   AND TRUNC(SYSDATE) BETWEEN TRUNC(z.begda) AND trunc(z.enda)
                   AND z.COMPANY_CODE IS NOT NULL
                   AND z.WERKS IS NOT NULL 
                   AND z.BTRTL IS NULL
               ) OR 
               a.COMPANY_CODE || '-' || a.WERKS IN (
                 SELECT DISTINCT z.COMPANY_CODE || '-' || z.WERKS FROM SAFR_COMPANY_D z 
                 WHERE 
                   z.STATUS = 1
                   AND Z.IS_EXCLUDE = 0
                   AND TRUNC(SYSDATE) BETWEEN TRUNC(z.begda) AND trunc(z.enda)
                   AND z.COMPANY_CODE IS NOT NULL
                   AND z.WERKS IS NOT NULL 
                   AND z.BTRTL IS NULL
               ) OR 
               a.COMPANY_CODE || '-' || a.WERKS || '-' || a.BTRTL IN (
                 SELECT DISTINCT z.COMPANY_CODE || '-' || z.WERKS || '-' || z.BTRTL FROM SAFR_COMPANY_D z 
                 WHERE 
                   z.STATUS = 1
                   AND Z.IS_EXCLUDE = 0
                   AND TRUNC(SYSDATE) BETWEEN TRUNC(z.begda) AND trunc(z.enda)
                   AND z.COMPANY_CODE IS NOT NULL
                   AND z.WERKS IS NOT NULL 
                   AND z.BTRTL IS NOT NULL
               ) OR
               a.WERKS IN (
                 SELECT DISTINCT z.WERKS FROM SAFR_COMPANY_D z 
                 WHERE 
                   z.STATUS = 1
                   AND Z.IS_EXCLUDE = 0
                   AND TRUNC(SYSDATE) BETWEEN TRUNC(z.begda) AND trunc(z.enda)
                   AND z.COMPANY_CODE IS NULL
                   AND z.WERKS IS NOT NULL 
                   AND z.BTRTL IS NULL 
               ) OR 
           a.WERKS || '-' || a.BTRTL IN (
             SELECT DISTINCT z.WERKS || '-' || z.BTRTL FROM SAFR_COMPANY_D z
             WHERE
               z.STATUS = 1
               AND Z.IS_EXCLUDE = 0
               AND TRUNC(SYSDATE) BETWEEN TRUNC(z.begda) AND trunc(z.enda)
               AND z.COMPANY_CODE IS NULL
               AND z.WERKS IS NOT NULL 
               AND z.BTRTL IS NOT NULL 
             )
             ) 
             AND NOT (
               a.COMPANY_CODE IN (
                 SELECT DISTINCT z.COMPANY_CODE FROM SAFR_COMPANY_D z 
                 WHERE 
                   z.STATUS = 1
                   AND Z.IS_EXCLUDE = 1
                   AND TRUNC(SYSDATE) BETWEEN TRUNC(z.begda) AND trunc(z.enda)
                   AND z.COMPANY_CODE IS NOT NULL
                   AND z.WERKS IS NULL 
                   AND z.BTRTL IS NULL
               ) OR 
               a.COMPANY_CODE || '-' || a.WERKS IN (
                 SELECT DISTINCT z.COMPANY_CODE || '-' || z.WERKS FROM SAFR_COMPANY_D z 
                 WHERE 
                   z.STATUS = 1
                   AND Z.IS_EXCLUDE = 1
                   AND TRUNC(SYSDATE) BETWEEN TRUNC(z.begda) AND trunc(z.enda)
                   AND z.COMPANY_CODE IS NOT NULL
                   AND z.WERKS IS NOT NULL 
                   AND z.BTRTL IS NULL
               ) OR 
               a.COMPANY_CODE || '-' || a.WERKS || '-' || a.BTRTL IN (
                 SELECT DISTINCT z.COMPANY_CODE || '-' || z.WERKS || '-' || z.BTRTL FROM SAFR_COMPANY_D z 
                 WHERE 
                   z.STATUS = 1
                   AND Z.IS_EXCLUDE = 1
                   AND TRUNC(SYSDATE) BETWEEN TRUNC(z.begda) AND trunc(z.enda)
                   AND z.COMPANY_CODE IS NOT NULL
                   AND z.WERKS IS NOT NULL 
                   AND z.BTRTL IS NOT NULL
               ) OR
               a.WERKS IN (
                 SELECT DISTINCT z.WERKS FROM SAFR_COMPANY_D z 
                 WHERE 
                   z.STATUS = 1
                   AND Z.IS_EXCLUDE = 1
                   AND TRUNC(SYSDATE) BETWEEN TRUNC(z.begda) AND trunc(z.enda)
                   AND z.COMPANY_CODE IS NULL
                   AND z.WERKS IS NOT NULL 
                   AND z.BTRTL IS NULL 
               ) OR 
           a.WERKS || '-' || a.BTRTL IN (
             SELECT DISTINCT z.WERKS || '-' || z.BTRTL FROM SAFR_COMPANY_D z
             WHERE
               z.STATUS = 1
               AND Z.IS_EXCLUDE = 1
               AND TRUNC(SYSDATE) BETWEEN TRUNC(z.begda) AND trunc(z.enda)
               AND z.COMPANY_CODE IS NULL
               AND z.WERKS IS NOT NULL 
               AND z.BTRTL IS NOT NULL 
             )
           )
           ORDER BY LAST_UPDATED_DATE DESC
            ) SELECT t1.*, count(*) OVER() AS total_count FROM t1 START WITH 
       t1.id_parent_group IS NULL
      CONNECT BY 
      PRIOR t1.id_master_group||t1.id_master_perusahaan_terkait = t1.id_parent_group||t1.id_master_perusahaan_terkait
      OFFSET :offset ROWS FETCH NEXT :per_page ROWS ONLY
```
</details>

### Penjelasan

#### 1. CTE `t1`

- Query mengambil data dari tabel `SAFM_STRUKTUR_ORGANISASI` dan melakukan filtering berdasarkan tanggal saat ini (`SYSDATE`) untuk mengambil data yang masih berlaku (`begda` dan `endda`) dan data dengan tipe objek `O`.
- Filter AND:
  - Menyaring data berdasarkan kombinasi `COMPANY_CODE`, `WERKS`, dan `BTRTL` dalam berbagai kondisi:
  - Ada di dalam `SAFR_COMPANY_D` dengan `STATUS = 1` dan `IS_EXCLUDE = 0`.
  - Menyaring data dengan mengecualikan data yang memenuhi syarat dari subquery kedua yang mengandung `IS_EXCLUDE = 1`.
- Data yang memenuhi syarat diurutkan berdasarkan `LAST_UPDATED_DATE` dalam urutan menurun (`DESC`).

#### 2. Query Utama

```sql
SELECT t1.*, COUNT(*) OVER() AS total_count
FROM t1
START WITH t1.id_parent_group IS NULL
CONNECT BY PRIOR t1.id_master_group || t1.id_master_perusahaan_terkait = t1.id_parent_group || t1.id_master_perusahaan_terkait
OFFSET :offset ROWS FETCH NEXT :per_page ROWS ONLY
```

- Mengambil semua kolom dari hasil CTE `t1`.
- Menambahkan kolom `total_count` yang menunjukkan jumlah total baris yang dihasilkan oleh query.
- Menggunakan **hierarchical query** (`START WITH dan CONNECT BY`) untuk membangun hierarki berdasarkan `id_master_group` dan `id_parent_group`.
- Hasil akhir diambil dengan menggunakan pagination (`OFFSET` dan `FETCH NEXT`) berdasarkan nilai `:offset` dan `:per_page` yang diberikan.

- **Pagination**:
  - `OFFSET :offset ROWS`: Mengabaikan sejumlah baris pertama sesuai nilai `offset`.
  - `FETCH NEXT :per_page ROWS ONLY`: Mengambil sejumlah baris yang ditentukan oleh `per_page`.

### Cara Penggunaan

- Tentukan nilai untuk :offset dan :per_page sesuai dengan kebutuhan pagination. :offset menunjukkan baris pertama yang akan diambil, dan :per_page menunjukkan jumlah baris yang akan diambil per halaman.
- Eksekusi query di SQL editor atau platform database yang Anda gunakan.
- Query ini akan menghasilkan data dalam format tabel dengan informasi struktur organisasi berdasarkan kriteria yang telah ditentukan.

Berikut contoh hasil outputnya:

| id_master_group | id_master_perusahaan_terkait | id_parent_group | nama_group              | level_group | tipe_group | waktu_mulai | waktu_selesai |
|-----------------|------------------------------|-----------------|-------------------------|-------------|------------|-------------|---------------|
| 70009759        | 7100                         | 70009757        | Departemen Layanan Umum | 4           | Departemen | 2023-10-01  | 9999-12-31    |


## API 13: Get Data Perusahaan

<details>
  <summary>Show SQL Query</summary>

```sql
SELECT DISTINCT count(*) OVER() AS total_count,
        COMP_CODE AS id_master_perusahaan,
        COMP_CODE_PARENT AS id_parent_perusahaan,
        COMP_NAME AS nama_perusahaan,
        '' AS lokasi,
        ABBREVIATION AS kode_perusahaan,
        '' AS tier_perusahaan,
        '' AS tipe_perusahaan,
        '' AS status_jenis_perusahaan_internal,
        '' AS waktu_mulai,
        '' AS waktu_selesai
        FROM SAFR_COMPANY a  WHERE 1=1 AND TRUNC(SYSDATE) BETWEEN TRUNC(a.VALID_FROM) AND trunc(a.VALID_TO) ${appendQuery}
      OFFSET :offset ROWS FETCH NEXT :per_page ROWS ONLY
```
</details>

### Penjelasan

 Query ini digunakan untuk mengambil data perusahaan dari tabel `SAFR_COMPANY`. Query ini menghasilkan beberapa informasi terkait perusahaan seperti `id_master_perusahaan`, `id_parent_perusahaan`, `nama_perusahaan`, dan `kode_perusahaan`. Selain itu, terdapat beberapa kolom yang diisi dengan nilai kosong ('') yang bisa diisi atau dihitung berdasarkan kebutuhan tambahan.

### Cara Kerja

- Query memilih kolom yang relevan seperti `COMP_CODE` sebagai `id_master_perusahaan`, `COMP_CODE_PARENT` sebagai `id_parent_perusahaan`, dan `COMP_NAME` sebagai `nama_perusahaan`, serta beberapa kolom tambahan yang diset kosong seperti lokasi, `tier_perusahaan`, `tipe_perusahaan`, `status_jenis_perusahaan_internal`, `waktu_mulai`, dan `waktu_selesai`.
- Query memfilter data berdasarkan waktu validitas (`VALID_FROM` dan `VALID_TO`) dengan memastikan bahwa data perusahaan yang diambil adalah yang berlaku pada tanggal sekarang (`TRUNC(SYSDATE)`).
- Hasil query diatur untuk mendukung paginasi menggunakan `OFFSET` dan `FETCH NEXT`, yang berarti query hanya akan mengembalikan sejumlah baris data tertentu pada setiap kali pemanggilan.
- Kolom `total_count` menghitung jumlah total baris yang sesuai dengan kriteria pencarian, yang berguna untuk mengetahui total data meskipun hanya sebagian kecil yang ditampilkan karena paginasi.

### Cara Penggunaan

- Tentukan nilai untuk :offset dan :per_page sesuai dengan kebutuhan pagination. :offset menunjukkan baris pertama yang akan diambil, dan :per_page menunjukkan jumlah baris yang akan diambil per halaman.
- Eksekusi query di SQL editor atau platform database yang Anda gunakan.

Berikut contoh hasil outputnya:

| total_count | id_master_perusahaan | id_parent_perusahaan | nama_perusahaan | lokasi | kode_perusahaan | tier_perusahaan | tipe_perusahaan | status_jenis_perusahaan_internal | waktu_mulai | waktu_selesai |
|-------------|----------------------|----------------------|-----------------|--------|-----------------|-----------------|-----------------|---------------------------------|-------------|---------------|
| 150         | 1001                 | 5000                 | Perusahaan A     |         | PERSA            |                 |                 |                                 |             |               |
| 150         | 1002                 | 5000                 | Perusahaan B     |         | PERSB            |                 |                 |                                 |             |               |
