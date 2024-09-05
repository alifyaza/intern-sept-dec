# SQL Query API Explanation

SQL Query API didapatkan dari spreadsheet [Mapping Data Portaverse MDM]()

## Table of Contents
- [SQL Query API Explanation](#sql-query-api-explanation)
  - [Table of Contents](#table-of-contents)
  - [API 1: Get Identitas Pegawai](#api-1-get-identitas-pegawai)
    - [1. CTE (Common Table Expression) `tpegawai`](#1-cte-common-table-expression-tpegawai)
    - [2. Query Utama](#2-query-utama)
    - [Cara Penggunaan](#cara-penggunaan)
    - [Kesimpulan](#kesimpulan)
  - [API 2: Get Data Pendidikan Pegawai](#api-2-get-data-pendidikan-pegawai)
    - [1. Query Pertama](#1-query-pertama)
      - [Struktur dan Penjelasan:](#struktur-dan-penjelasan)
        - [1. CTE (Common Table Expression) `tpegawai`](#1-cte-common-table-expression-tpegawai-1)
        - [2. Pengambilan Data Utama](#2-pengambilan-data-utama)
      - [Kesimpulan](#kesimpulan-1)
    - [2. Query Kedua](#2-query-kedua)
      - [Struktur dan Penjelasan:](#struktur-dan-penjelasan-1)
        - [Pengambilan Data Pendidikan:](#pengambilan-data-pendidikan)
      - [Kesimpulan](#kesimpulan-2)
    - [Cara Penggunaan](#cara-penggunaan-1)
  - [API 3: Get Data Keluarga Pegawai](#api-3-get-data-keluarga-pegawai)
    - [1. Query Pertama](#1-query-pertama-1)
      - [1. CTE (Common Table Expression) `tpegawai`](#1-cte-common-table-expression-tpegawai-2)
      - [2. Query Utama](#2-query-utama-1)
      - [Kesimpulan](#kesimpulan-3)
    - [2. Query Kedua](#2-query-kedua-1)
      - [Penjelasan](#penjelasan)
      - [Kesimpulan](#kesimpulan-4)
  - [API 4: Get Data Cuti Pegawai](#api-4-get-data-cuti-pegawai)
    - [1. Query Pertama](#1-query-pertama-2)
      - [1. CTE (Common Table Expression) tpegawai](#1-cte-common-table-expression-tpegawai-3)
      - [2. Mengambil Data dari CTE `tpegawai`](#2-mengambil-data-dari-cte-tpegawai)
    - [2. Query Kedua](#2-query-kedua-2)
    - [Kesimpulan](#kesimpulan-5)
  - [API 5: Get Data Indisipliner Pegawai](#api-5-get-data-indisipliner-pegawai)
    - [1. Query Pertama](#1-query-pertama-3)
      - [1. CTE `tpegawai`](#1-cte-tpegawai)
      - [2. Query Utama](#2-query-utama-2)
    - [2. Query Kedua](#2-query-kedua-3)
    - [Cara Penggunaan](#cara-penggunaan-2)
    - [Kesimpulan](#kesimpulan-6)

## API 1: Get Identitas Pegawai
<details>
  <summary>Show SQL Query</summary>

```sql
WITH tpegawai AS (
                SELECT DISTINCT
                            ROW_NUMBER() OVER (PARTITION BY 
                            msp.pnalt_new
         ORDER BY msp.LAST_UPDATED_DATE ) AS row_num,
                            RANK( ) OVER (
                             PARTITION BY msp.pnalt_new
                             ORDER BY
                             msp.LAST_UPDATED_DATE DESC
                         ) my_rank,
                         msp.PERNR,
                         msp.cname               AS nama,
                         msp.pnalt_new AS nipp_pegawai,
                         msp.COMPANY_CODE,
                         sp.title               AS gelar_akademik,
                         sp.TITLE2              AS gelar_akademik2,
                         sp.ADDTITLE            AS gelar_akademik3,
                         msp.PNALT_NEW           AS nipp,
                         sp.OFFICMAIL           AS email_korporat,
                         sp.CONTRACTTYPETEXT    AS status,
                         msp.plans AS nama_posisi_aktif,
                         msp.subdi_text AS nama_organisasi_aktif,
                         sc.comp_name   AS nama_perusahaan_aktif,
                         c.CNAME_ATS AS atasan_definitif,
                         c.STEXT_ATS AS posisi_atasan_definitif,
                         d.CNAME AS atasan_job_assignment,
                         d.NAMA_JABATAN AS posisi_atasan_job_assignment,
                         sp.ICNUM               AS nik,
                         sp.TAXID               AS npwp,
                         sp.GBORT               AS tempat_lahir,
                         TO_CHAR(sp.GBDAT, 'YYYY-MM-DD')               AS tanggal_lahir,
                         TO_CHAR(msp.TMTMULAIBEKERJA, 'YYYY-MM-DD')     AS tanggal_mulai_bekerja,
                         TO_CHAR(msp.TMTDIANGKATPEGAWAI, 'YYYY-MM-DD')  AS tanggal_diangkat_sebagai_pegawai,
                         TO_CHAR(sp.TMT_GOLONGAN, 'YYYY-MM-DD')        AS tanggal_mulai_golongan,
                         sp.GENDE               AS jenis_kelamin,
                         sp.BPJSKETENAGAKERJAAN AS bpjs_ketenagakerjaan,
                         sp.BPJSKESEHATAN       AS bpjs_kesehatan,
                         sp.RELIGIONTEXT        AS agama,
                         sp.ETHNICITY           AS suku,
                         sp.FATXT               AS status_perkawinan,
                         sp.PHNUMBER            AS handphone,
                         sp.PRIVATEMAIL         AS email_privat,
                         sp.BLOODTYPE           AS golongan_darah,
                         COALESCE(msp.kj_new, msp.trfg0) AS kelas_jabatan,
                         sp.PHNUMBER            AS telephone_rumah
                         FROM MDM_SINGLE_PEGAWAI msp
                         LEFT JOIN SAFR_COMPANY sc ON msp.COMPANY_CODE = sc.comp_code
                         LEFT JOIN SAFM_PEGAWAI sp ON msp.pernr = sp.pernr AND msp.PNALT_NEW = sp.PNALT_NEW and msp.company_code = sp.company_code
                         LEFT JOIN SAFM_PEGAWAI_ATASAN_BAWAHAN c ON  msp.PERNR = c.pernr and msp.company_code = c.company_code and c.order_atasanbawahan = 1
                         LEFT JOIN SAFM_PEGAWAI_SECOND_ASSIGNMENT d ON c.PERNR_ATS = d.PERNR
                         WHERE 1=1 and c.CNAME_ATS is not NULL AND msp.pnalt_new = '104641'
                )
                SELECT DISTINCT
                         count(*) OVER() AS total_count,
                         t.my_rank,
                         t.row_num,
                         t.pernr,
                         t.nama,
                         t.nipp_pegawai
                         ,t.company_code,
                         t.gelar_akademik,
                         t.gelar_akademik2,
                         t.gelar_akademik3,
                          t.nipp,
                          t.email_korporat,
                         t.status,
                         t.nama_posisi_aktif,
                         t.nama_organisasi_aktif,
                          t.nama_perusahaan_aktif,
                         t.atasan_definitif,
                         t.posisi_atasan_definitif,
                         t.atasan_job_assignment,
                         t.atasan_job_assignment,
                         t.posisi_atasan_job_assignment,
                         t.nik,
                         t.npwp,
                         t.tempat_lahir,
                         t.tanggal_lahir,
                         t.tanggal_mulai_bekerja,
                         t.tanggal_diangkat_sebagai_pegawai,
                         t.tanggal_mulai_golongan,
                         t.jenis_kelamin,
                         t.bpjs_ketenagakerjaan,
                         t.bpjs_kesehatan,
                         t.agama,
                         t.suku,
                         t.status_perkawinan,
                         t.handphone,
                         t.email_privat,
                         t.golongan_darah,
                         t.kelas_jabatan,
                         t.telephone_rumah
                         FROM tpegawai t WHERE row_num = 1 ORDER BY t.pernr ASC OFFSET 0 ROWS FETCH NEXT 100 ROWS ONLY;
```
</details>

Query tersebut mencari informasi tentang pegawai dengan `pnalt_new = '104641'`, termasuk informasi dasar seperti nama, NIK, tanggal lahir, posisi aktif, nama organisasi, atasan, dan lain-lain. Data ini kemudian difilter untuk hanya menampilkan data yang terbaru berdasarkan `LAST_UPDATED_DATE`, dan mengurutkan hasil berdasarkan `pernr`.

Berikut adalah  yang memuat informasi dasar dari query 1:

<details>
  <summary>Show Table of Basic Information</summary>

| **column_name**                                    | **alias_name**                     | **desc**                                      | **source**                                      |
|---------------------------------------------------|-----------------------------------|-----------------------------------------------|-------------------------------------------------|
| `msp.PERNR`                                      | `pernr`                           | Nomor identifikasi pegawai                   |  `MDM_SINGLE_PEGAWAI`                      |
| `msp.cname`                                      | `nama`                            | Nama pegawai                                 |  `MDM_SINGLE_PEGAWAI`                      |
| `msp.pnalt_new`                                  | `nipp_pegawai`                    | NIPP pegawai                                 |  `MDM_SINGLE_PEGAWAI`                      |
| `msp.COMPANY_CODE`                               | `company_code`                    | Kode perusahaan                              |  `MDM_SINGLE_PEGAWAI`                      |
| `sp.title`                                       | `gelar_akademik`                  | Gelar akademik pegawai                       |  `SAFM_PEGAWAI`                           |
| `sp.TITLE2`                                      | `gelar_akademik2`                 | Gelar akademik tambahan                      |  `SAFM_PEGAWAI`                           |
| `sp.ADDTITLE`                                    | `gelar_akademik3`                 | Gelar akademik tambahan lainnya              |  `SAFM_PEGAWAI`                           |
| `msp.PNALT_NEW`                                  | `nipp`                            | NIPP pegawai (sama dengan `nipp_pegawai`)    |  `MDM_SINGLE_PEGAWAI`                      |
| `sp.OFFICMAIL`                                      | `email_korporat`                  | Email korporat pegawai                       |  `SAFM_PEGAWAI`                           |
| `sp.CONTRACTTYPETEXT`                            | `status`                          | Status pegawai                               |  `SAFM_PEGAWAI`                           |
| `msp.plans`                                      | `nama_posisi_aktif`               | Nama posisi aktif pegawai                    |  `MDM_SINGLE_PEGAWAI`                      |
| `msp.subdi_text`                                 | `nama_organisasi_aktif`           | Nama organisasi aktif pegawai                |  `MDM_SINGLE_PEGAWAI`                      |
| `sc.comp_name`                                   | `nama_perusahaan_aktif`           | Nama perusahaan aktif pegawai                |  `SAFR_COMPANY`                           |
| `c.CNAME_ATS`                                    | `atasan_definitif`                | Nama atasan definitif                        |  `SAFM_PEGAWAI_ATASAN_BAWAHAN`             |
| `c.STEXT_ATS`                                    | `posisi_atasan_definitif`         | Posisi atasan definitif                      |  `SAFM_PEGAWAI_ATASAN_BAWAHAN`             |
| `d.CNAME`                                        | `atasan_job_assignment`           | Nama atasan untuk job assignment             |  `SAFM_PEGAWAI_SECOND_ASSIGNMENT`          |
| `d.NAMA_JABATAN`                                 | `posisi_atasan_job_assignment`    | Posisi atasan untuk job assignment           |  `SAFM_PEGAWAI_SECOND_ASSIGNMENT`          |
| `sp.ICNUM`                                       | `nik`                             | Nomor Induk Kependudukan pegawai             |  `SAFM_PEGAWAI`                           |
| `sp.TAXID`                                       | `npwp`                            | Nomor Pokok Wajib Pajak pegawai              |  `SAFM_PEGAWAI`                           |
| `sp.GBORT`                                       | `tempat_lahir`                    | Tempat lahir pegawai                         |  `SAFM_PEGAWAI`                           |
| `TO_CHAR(sp.GBDAT, 'YYYY-MM-DD')`                | `tanggal_lahir`                   | Tanggal lahir pegawai                        |  `SAFM_PEGAWAI`                           |
| `TO_CHAR(msp.TMTMULAIBEKERJA, 'YYYY-MM-DD')`     | `tanggal_mulai_bekerja`           | Tanggal mulai bekerja pegawai                |  `MDM_SINGLE_PEGAWAI`                      |
| `TO_CHAR(msp.TMTDIANGKATPEGAWAI, 'YYYY-MM-DD')`  | `tanggal_diangkat_sebagai_pegawai` | Tanggal diangkat sebagai pegawai           |  `MDM_SINGLE_PEGAWAI`                      |
| `TO_CHAR(sp.TMT_GOLONGAN, 'YYYY-MM-DD')`         | `tanggal_mulai_golongan`          | Tanggal mulai golongan pegawai               |  `SAFM_PEGAWAI`                           |
| `sp.GENDE`                                       | `jenis_kelamin`                   | Jenis kelamin pegawai                        |  `SAFM_PEGAWAI`                           |
| `sp.BPJSKETENAGAKERJAAN`                         | `bpjs_ketenagakerjaan`            | Nomor BPJS Ketenagakerjaan pegawai            |  `SAFM_PEGAWAI`                           |
| `sp.BPJSKESEHATAN`                               | `bpjs_kesehatan`                  | Nomor BPJS Kesehatan pegawai                 |  `SAFM_PEGAWAI`                           |
| `sp.RELIGIONTEXT`                                | `agama`                           | Agama pegawai                                |  `SAFM_PEGAWAI`                           |
| `sp.ETHNICITY`                                   | `suku`                            | Suku pegawai                                 |  `SAFM_PEGAWAI`                           |
| `sp.FATXT`                                       | `status_perkawinan`               | Status perkawinan pegawai                    |  `SAFM_PEGAWAI`                           |
| `sp.PHNUMBER`                                    | `handphone`                       | Nomor handphone pegawai                      |  `SAFM_PEGAWAI`                           |
| `sp.PRIVATEMAIL`                                 | `email_privat`                    | Email pribadi pegawai                        | `SAFM_PEGAWAI`                           |
| `sp.BLOODTYPE`                                   | `golongan_darah`                  | Golongan darah pegawai                        |  `SAFM_PEGAWAI`                           |
| `COALESCE(msp.kj_new, msp.trfg0)`                | `kelas_jabatan`                   | Kelas jabatan pegawai                        |  `MDM_SINGLE_PEGAWAI`                      |
| `sp.PHNUMBER`                                    | `telephone_rumah`                 | Nomor telepon rumah pegawai                  |  `SAFM_PEGAWAI`                           |

</details>

### 1. CTE (Common Table Expression) `tpegawai`

- **Fungsi**: Bagian pertama dari query ini adalah CTE yang diberi nama `tpegawai`. CTE ini digunakan untuk membentuk data atau tabel sementara yang dapat dipanggil dalam query utama.
- **Penjelasan**:
  - `ROW_NUMBER() OVER (PARTITION BY ... ORDER BY ...) AS row_num`:
    - Dalam `PARTITION BY pnalt_new`, data dibagi per grup `pnalt_new`. Jadi, baris dengan `pnalt_new` yang sama diurutkan berdasarkan `LAST_UPDATED_DATE`.
    - Hasilnya, pegawai dengan LAST_UPDATED_DATE paling awal mendapat nomor urut 1 dalam grupnya.
  - `RANK() OVER (PARTITION BY ... ORDER BY ... DESC) my_rank`:
    - Dalam `PARTITION BY pnalt_new`, data dibagi per grup `pnalt_new` dan diurutkan secara menurun berdasarkan `LAST_UPDATED_DATE`.
    - Baris dengan `LAST_UPDATED_DATE` terbaru mendapat peringkat 1.
    - Jika ada dua baris dengan tanggal yang sama, mereka akan mendapatkan peringkat yang sama, dan peringkat berikutnya akan melompati angka yang seharusnya.

    Example:

    | pernr | pnalt_new | LAST_UPDATED_DATE | row_num | my_rank |
    |-------|-----------|-------------------|---------|---------|
    | 1001  | 104641    | 2024-09-01        | 1       | 5       |
    | 1002  | 104641    | 2024-09-02        | 2       | 3       |
    | 1003  | 104641    | 2024-09-02        | 3       | 3       |
    | 1004  | 104641    | 2024-09-03        | 4       | 1       |
    | 1005  | 104641    | 2024-09-03        | 5       | 1       |
    > Note: Fungsi RANK() memberikan peringkat dengan "lompat" pada peringkat jika ada nilai yang sama. Jadi, setelah dua nilai mendapatkan peringkat 1, peringkat berikutnya adalah 3 (bukan 2) karena ada dua entri yang memiliki peringkat 1.

  - **Kolom yang diambil**:
    - Berbagai informasi terkait pegawai seperti nama, nipp_pegawai, company_code, gelar_akademik, email_korporat, status, nik, npwp, tempat_lahir, tanggal_lahir, dll.

  - **Tabel yang digunakan**:
    1. `MDM_SINGLE_PEGAWAI msp`: Tabel utama yang memuat data pegawai.
    2. `SAFR_COMPANY sc`: Tabel yang memuat informasi perusahaan.
    3. `safm_pegawai sp`: Tabel yang memuat informasi tambahan tentang pegawai.
    4. `SAFM_PEGAWAI_ATASAN_BAWAHAN c`: Tabel yang menghubungkan pegawai dengan atasan definitif.
    5. `SAFM_PEGAWAI_SECOND_ASSIGNMENT d`: Tabel yang memuat informasi penugasan kedua untuk atasan. 

  - **Filter**:
    - `WHERE 1=1 and c.CNAME_ATS is not NULL AND msp.pnalt_new = '104641'`:
      - Memfilter pegawai dengan `pnalt_new = '104641'` dan yang memiliki atasan definitif (`c.CNAME_ATS is not NULL`).

### 2. Query Utama

- **Fungsi**: Bagian ini mengambil data dari CTE tpegawai yang telah dibentuk sebelumnya.
- **Penjelasan**:
  - `SELECT DISTINCT ...`:
    - Pada **CTE**, `DISTINCT` memastikan bahwa baris-baris yang dihasilkan oleh subquery dalam CTE tpegawai tidak memiliki duplikasi, yaitu baris-baris yang mungkin memiliki kombinasi data yang sama.
    - Pada **Query utama**, `DISTINCT` digunakan lagi untuk memastikan bahwa hasil akhir yang diambil dari CTE tpegawai tidak memiliki baris yang duplikat.
    - Fungsi `DISTINCT`: Memilih data yang unik berdasarkan kolom-kolom yang dipilih.
  
  -  **Kolom yang diambil**:
     -  Data yang sama seperti yang ada di **CTE**, dengan tambahan dua kolom baru yaitu `total_count` (menghitung total jumlah baris) dan `my_rank`.

  - **Filter**:
    - `WHERE row_num = 1`:
      - Hanya mengambil data pegawai yang memiliki nomor urut 1, artinya pegawai dengan `LAST_UPDATED_DATE` paling lama (dengan syarat `PARTITION BY` yang diberikan).

  - **Pengurutan**:
    - `ORDER BY t.pernr ASC`:
      - Mengurutkan hasil berdasarkan `pernr` secara ascending.

  - **Pagination**:
    - `OFFSET 0 ROWS FETCH NEXT 100 ROWS ONLY`:
      - Menampilkan 100 baris pertama dari hasil query
      - Bagian ini ada untuk menangani skenario yang lebih umum di mana pnalt_new tidak dibatasi ke satu nilai saja, memungkinkan query ini digunakan dalam berbagai konteks tanpa perlu banyak modifikasi.

### Cara Penggunaan

Query ini digunakan untuk mendapatkan data identitas pegawai yang menunjukkan informasi dasar lainnya, yang disajikan dalam bentuk tabel. Untuk menjalankan query ini, user perlu menyertakan nilai spesifik dari `pnalt_new` (atau `nipp_pegawai`) pada bagian yang sesuai.
1. **Penyesuaian Parameter**
   
   Ganti `(value)` di bagian berikut (bagian CTE `tpegawai`) dengan `pnalt_new` yang diinginkan:
   ```sql
   ...
   WHERE 1=1 
   AND c.CNAME_ATS is not NULL 
   AND msp.pnalt_new = '(value)'
   ...
   ```
   Contoh: Untuk melihat data pegawai dengan `pnalt_new = '104641'`, ubah menjadi:
   ```sql
   ...
   WHERE 1=1 
   AND c.CNAME_ATS is not NULL 
   AND msp.pnalt_new = '104641'
   ...
   ```
2. **Hasil Query**
   
   Query akan menampilkan informasi pegawai dalam bentuk tabel, termasuk data seperti nama, posisi, atasan, dan informasi kontak.

### Kesimpulan

Dalam query tersebut, dimisalkan data dengan `pnalt_new = '104641'` akan diproses, dan hanya satu baris data dengan `row_num = 1` (yang memiliki `LAST_UPDATED_DATE` paling awal) yang akan diambil. Karena query ini difokuskan pada satu nilai `pnalt_new` yang spesifik, hasil akhirnya memang hanya akan mengembalikan satu baris data yang memenuhi kondisi tersebut.

## API 2: Get Data Pendidikan Pegawai

### 1. Query Pertama

<details>
  <summary>Show SQL Query 1</summary>

```sql
WITH tpegawai AS ( SELECT msp.*, ROW_NUMBER () OVER(PARTITION BY msp.pernr, msp.pnalt_new ORDER BY msp.pernr ASC) AS row_num 
        FROM mdm_single_pegawai msp  JOIN (SELECT DISTINCT PERNR FROM SAFM_PEGAWAI_EDUCATION  ${condition}) spe on msp.PERNR = spe.PERNR
        WHERE 1=1 -- and msp.pernr = '' and msp.pnalt_new = '' and msp.company_code = '' 
        SELECT msp.PERNR,
          count(*) OVER() AS total_count, 
          msp.PNALT_NEW AS nipp_pegawai,
          msp.cname AS nama_pegawai
        FROM tpegawai msp
        JOIN (SELECT DISTINCT PERNR FROM SAFM_PEGAWAI_EDUCATION  WHERE ET_CATEGORIES IN ('001', '002')) spe on msp.PERNR = spe.PERNR
        WHERE msp.pnalt_new IS NOT NULL and msp.row_num = 1 -- and msp.pernr = '' and msp.pnalt_new = '' and msp.company_code = '' 
        ORDER BY msp.LAST_UPDATED_DATE DESC
        OFFSET :offset ROWS FETCH NEXT :per_page ROWS ONLY
```
</details>

**Query pertama** digunakan untuk mengambil data pegawai dari tabel `mdm_single_pegawai` yang memiliki data pendidikan dalam tabel `SAFM_PEGAWAI_EDUCATION`. Data ini kemudian difilter berdasarkan beberapa kondisi dan diurutkan berdasarkan `LAST_UPDATED_DATE`.

#### Struktur dan Penjelasan:

##### 1. CTE (Common Table Expression) `tpegawai`

```sql
WITH tpegawai AS (
    SELECT msp.*, 
           ROW_NUMBER () OVER(PARTITION BY msp.pernr, msp.pnalt_new ORDER BY msp.pernr ASC) AS row_num 
    FROM mdm_single_pegawai msp  
    JOIN (SELECT DISTINCT PERNR FROM SAFM_PEGAWAI_EDUCATION ${condition}) spe 
    ON msp.PERNR = spe.PERNR
    WHERE 1=1 
)
```

- `msp.*`: Mengambil semua kolom dari tabel `mdm_single_pegawai`.
- `ROW_NUMBER() OVER(PARTITION BY msp.pernr, msp.pnalt_new ORDER BY msp.pernr ASC) AS row_num`: Menghitung nomor urut untuk setiap baris yang dikelompokkan berdasarkan `msp.pernr` dan `msp.pnalt_new`, dan diurutkan berdasarkan `msp.pernr` secara ascending.
  
  Example:
  | pernr | pnalt_new | cname        | row_num |
  |-------|-----------|--------------|---------|
  | 1001  | 123       | John Doe     | 1       |
  | 1001  | 123       | John Doe     | 2       |
  | 1001  | 126       | Ghulam Ahmed | 1       |
  | 1002  | 124       | Lauren Smith | 1       |
  | 1003  | 125       | Daniel Lee   | 1       | 

- `JOIN` dengan `SAFM_PEGAWAI_EDUCATION`: Hanya data pegawai yang memiliki pendidikan terkait yang akan diambil, sesuai dengan kondisi pada `${condition}`.
- `WHERE 1=1`: Digunakan untuk mempermudah penambahan kondisi filter lainnya.

##### 2. Pengambilan Data Utama

```sql
SELECT msp.PERNR,
       count(*) OVER() AS total_count, 
       msp.PNALT_NEW AS nipp_pegawai,
       msp.cname AS nama_pegawai
FROM tpegawai msp
JOIN (SELECT DISTINCT PERNR FROM SAFM_PEGAWAI_EDUCATION WHERE ET_CATEGORIES IN ('001', '002')) spe 
ON msp.PERNR = spe.PERNR
WHERE msp.pnalt_new IS NOT NULL 
AND msp.row_num = 1 
ORDER BY msp.LAST_UPDATED_DATE DESC
OFFSET :offset ROWS FETCH NEXT :per_page ROWS ONLY
```

- `count(*) OVER() AS total_count`: Menghitung total jumlah baris yang dihasilkan oleh query ini tanpa memperhatikan pagination.
  - `total_count` menunjukkan total jumlah pegawai yang cocok dengan kriteria pencarian dan filter yang diterapkan dalam query, termasuk semua baris yang ada.
- `JOIN` dengan `SAFM_PEGAWAI_EDUCATION`: Hanya pegawai yang memiliki kategori pendidikan tertentu (dalam hal ini ET_CATEGORIES adalah **'001'** atau **'002'**) yang akan diambil.
- `WHERE msp.pnalt_new IS NOT NULL AND msp.row_num = 1`: Memastikan bahwa hanya data pegawai dengan `pnalt_new` yang **tidak null** dan baris pertama dari setiap grup (`row_num = 1`) yang akan diambil.
- `ORDER BY msp.LAST_UPDATED_DATE DESC`: Mengurutkan data berdasarkan `LAST_UPDATED_DATE` dari yang terbaru.
- **Pagination** (`OFFSET` dan `FETCH`): Mengambil baris sesuai dengan nilai `:offset` dan `:per_page` yang diberikan, untuk implementasi pagination.
  - Mengurangi beban pada database dan jaringan dengan hanya mengambil data yang diperlukan untuk ditampilkan pada satu waktu, bukan seluruh data sekaligus, sehingga waktu response query akan lebih cepat.
  - Misalkan ada 1000 baris data, dan pengguna ingin melihat 10 baris per halaman. Jika pengguna berada di halaman ke-3, maka:
    - `OFFSET` akan bernilai 20 (melewati 20 baris pertama).
`FETCH NEXT 10 ROWS ONLY` akan mengambil 10 baris berikutnya (baris ke-21 hingga ke-30).

#### Kesimpulan

**Query pertama** mengembalikan daftar pegawai yang memiliki data pendidikan tertentu dengan memastikan hanya satu baris data per pegawai (ditentukan oleh `row_num = 1`) yang diambil dari tabel `mdm_single_pegawai`, berdasarkan urutan terbaru (`LAST_UPDATED_DATE`). Data ini disaring dan diurutkan menggunakan nomor urut yang dihitung untuk memastikan hanya baris teratas per grup `pernr` dan `pnalt_new` yang diambil. Kolom `total_count` digunakan untuk menunjukkan jumlah total baris yang dihasilkan oleh query sebelum penerapan pagination, yang berguna untuk mengetahui total jumlah hasil tanpa memperhitungkan batasan halaman.

### 2. Query Kedua

<details>
  <summary>Show SQL Query 2</summary>

```sql
SELECT PERNR,
          education_est_desc AS level_pendidikan,
          inst_location AS nama_institusi_pendidikan,
          start_date AS tanggal_mulai,
          end_date AS tanggal_selesai,
          nomor_ijazah AS nomor_ijazah,
          '' AS ijazah,
          final_grade AS hasil_pendidikan
        FROM SAFM_PEGAWAI_EDUCATION
        WHERE PERNR IN (${pernrList})
```
</details>

**Query kedua** digunakan untuk mengambil detail pendidikan dari pegawai yang terpilih dalam query pertama.

#### Struktur dan Penjelasan:

##### Pengambilan Data Pendidikan:

- **Kolom yang diambil**:
  - `PERNR`: Nomor pegawai.
  - `education_est_desc`: Deskripsi tingkat pendidikan.
  - `inst_location`: Nama institusi pendidikan.
  - `start_date`: Tanggal mulai pendidikan.
  - `end_date`: Tanggal selesai pendidikan.
  - `nomor_ijazah`: Nomor ijazah.
  - `final_grade`: Nilai akhir dari pendidikan.
- `'' AS ijazah`: Kolom ijazah diisi dengan string kosong.
- `WHERE PERNR IN (${pernrList})`: Mengambil data pendidikan hanya untuk pegawai yang PERNR-nya ada dalam daftar `${pernrList}` yang berasal dari query pertama.

#### Kesimpulan

Query kedua mengambil detail pendidikan untuk pegawai berdasarkan daftar PERNR yang didapat dari query pertama:
- Mengambil informasi tentang pendidikan seperti level pendidikan, nama institusi, tanggal mulai, tanggal selesai, nomor ijazah, dan hasil pendidikan dari tabel `SAFM_PEGAWAI_EDUCATION`.
- Hanya pegawai yang PERNR-nya ada dalam daftar yang diberikan (`${pernrList}`) yang akan dimasukkan dalam hasil.

### Cara Penggunaan

- **Query Pertama**:

Untuk memakai query API ini, kita perlu menjalankan dua kali: query pertama lalu query kedua. Pada query pertama, user tidak perlu menginisialisasikan nilai untuk value `pernr`, `pnalt_new`, maupun `company_code`. Cukup gunakan query pertama dengan menghapus `${condition}` pada bagian CTE.

Maka hasil outputnya akan seperti contoh berikut:

| PERNR       | total_count | nipp_pegawai | nama_pegawai |
|-------------|-------------|--------------|--------------|
| 000000284848| 50          | 123456789    | John Doe     |

- **Query Kedua**:

Pada **query kedua**, user perlu melakukan inisialisasi nilai pernr yang bisa dilakukan sebelum ataupun sesudah eksekusi. Jika dilakukan sebelum eksekusi, maka inisialisasi nilai dilakukan langsung pada code dengan mengubah bagian berikut:

```sql
...
WHERE PERNR IN (${pernrList})
...
```

Misal user ingin mencari data dengan pernr = '00016480', maka code diubah menjadi:

```sql
...
WHERE PERNR IN ('00016480')
...
```

Namun, jika user ingin melakukan inisialisasi setelah eksekusi query. Maka user bisa mengubah value pada bind parameter tab yang tampil. Pada tab tersebut, user diharuskan untuk memasukkan nilai `pernrList` pada tab tersebut dengan tanda ('), contohnya adalah `'00016480'`

Maka hasil outputnya akan seperti contoh berikut:

| PERNR       | level_pendidikan | nama_institusi_pendidikan | tanggal_mulai | tanggal_selesai | nomor_ijazah | ijazah | hasil_pendidikan |
|-------------|-------------------|---------------------------|---------------|-----------------|--------------|--------|------------------|
| 000000284848| SD                | SD Negeri Contoh          | 2000-07-01    | 2006-06-01      | 0987654321   |        | Lulus            |
| 000000284848| SMP               | SMP Negeri Contoh         | 2006-07-01    | 2009-06-01      | 1234567890   |        | Lulus            |
| 000000284848| SMA               | SMA Negeri Contoh         | 2009-07-01    | 2012-06-01      | 0987654321   |        | Lulus            |
| 000000284848| S1                | Universitas Contoh        | 2010-09-01    | 2014-05-01      | 1234567890   |        |                  |
| 000000284848| S2                | Institut Contoh           | 2015-09-01    | 2017-05-01      | 0987654321   |        |                  |

## API 3: Get Data Keluarga Pegawai

### 1. Query Pertama

<details>
  <summary>Show SQL Query 1</summary>

```sql
WITH tpegawai AS ( SELECT msp.*, ROW_NUMBER () OVER(PARTITION BY msp.pernr, msp.pnalt_new ORDER BY msp.pernr ASC) AS row_num 
        FROM mdm_single_pegawai msp  JOIN (SELECT DISTINCT PERNR FROM SAFM_PEGAWAI_EDUCATION  ${condition}) spe on msp.PERNR = spe.PERNR
        WHERE 1=1 -- and msp.pernr = '' and msp.pnalt_new = '' and msp.company_code = '' 
        SELECT msp.PERNR,
          count(*) OVER() AS total_count, 
          msp.PNALT_NEW AS nipp_pegawai,
          msp.cname AS nama_pegawai
        FROM tpegawai msp
        JOIN (SELECT DISTINCT PERNR FROM SAFM_PEGAWAI_FAMILY) spf on msp.PERNR = spf.PERNR
        WHERE msp.pnalt_new IS NOT NULL and msp.row_num = 1-- ${appendQueryNipp} ${appendQueryPernr} ${appendQuery}
        ORDER BY msp.LAST_UPDATED_DATE DESC
        OFFSET :offset ROWS FETCH NEXT :per_page ROWS ONLY
```
</details>

**Query pertama** digunakan untuk mengambil data pegawai dari tabel `mdm_single_pegawai` yang memiliki catatan di tabel `SAFM_PEGAWAI_FAMILY`, serta informasi dasar pegawai. Query ini menyaring data berdasarkan pegawai yang valid dan terbaru.

#### 1. CTE (Common Table Expression) `tpegawai`

```sql
WITH tpegawai AS (
    SELECT msp.*, 
           ROW_NUMBER() OVER(PARTITION BY msp.pernr, msp.pnalt_new ORDER BY msp.pernr ASC) AS row_num
    FROM mdm_single_pegawai msp
    JOIN (SELECT DISTINCT PERNR FROM SAFM_PEGAWAI_EDUCATION ${condition}) spe 
    ON msp.PERNR = spe.PERNR
    WHERE 1=1
)
```

- `ROW_NUMBER() OVER(PARTITION BY msp.pernr, msp.pnalt_new ORDER BY msp.pernr ASC) AS row_num`: Menghitung nomor baris untuk setiap pegawai berdasarkan `PERNR` dan `PNALT_NEW`. Setiap `PERNR` dengan `PNALT_NEW` yang sama diberikan nomor urut. Hanya memilih baris pertama dari setiap grup.
- `JOIN`: Menghubungkan tabel `mdm_single_pegawai` dengan `SAFM_PEGAWAI_EDUCATION` berdasarkan `PERNR`,  hanya menyertakan pegawai yang ada di tabel pendidikan.

#### 2. Query Utama

```sql
SELECT msp.PERNR,
       count(*) OVER() AS total_count, 
       msp.PNALT_NEW AS nipp_pegawai,
       msp.cname AS nama_pegawai
FROM tpegawai msp
JOIN (SELECT DISTINCT PERNR FROM SAFM_PEGAWAI_FAMILY) spf 
ON msp.PERNR = spf.PERNR
WHERE msp.pnalt_new IS NOT NULL AND msp.row_num = 1
ORDER BY msp.LAST_UPDATED_DATE DESC
OFFSET :offset ROWS FETCH NEXT :per_page ROWS ONLY
```

Hasil akhir dari query ini adalah daftar pegawai beserta **jumlah total hasil pencarian**, **nomor identifikasi pegawai (NIPP)**, dan **nama pegawai**, yang kemudian diurutkan berdasarkan tanggal terakhir pembaruan (``LAST_UPDATED_DATE``) dan diatur menggunakan paginasi.

- Query ini memastikan bahwa hanya pegawai yang memiliki data keluarga (terhubung dengan `SAFM_PEGAWAI_FAMILY`) yang akan muncul dalam hasil.
- `count(*) OVER() AS total_count`: Menghitung jumlah total baris yang sesuai dengan kriteria tanpa memperhatikan batasan paginasi.
- `WHERE msp.pnalt_new IS NOT NULL AND msp.row_num = 1`: Menyaring data untuk hanya memilih baris pertama per pegawai yang valid.
- `ORDER BY msp.LAST_UPDATED_DATE DESC`: Mengurutkan hasil berdasarkan `LAST_UPDATED_DATE` secara menurun.
- `OFFSET :offset ROWS FETCH NEXT :per_page ROWS ONLY`: Membatasi hasil berdasarkan parameter `offset` dan `per_page` untuk paginasi.

#### Kesimpulan

**Query pertama** bertujuan untuk mengambil data pegawai dari tabel `mdm_single_pegawai` yang memiliki catatan pendidikan di tabel `SAFM_PEGAWAI_EDUCATION` dan catatan keluarga di tabel `SAFM_PEGAWAI_FAMILY`. Hasil dari query ini mencakup informasi dasar pegawai, seperti **nomor pegawai** (`PERNR`), **nomor identitas pegawai baru** (`nipp_pegawai`), dan **nama pegawai** (`nama_pegawai`). Query juga memastikan bahwa hanya satu baris data yang diambil untuk setiap pegawai yang unik.

### 2. Query Kedua

<details>
  <summary>Show SQL Query 2</summary>

```sql
SELECT PERNR,
          name AS nama,
          birth_place AS tempat_lahir,
          birth_date AS tanggal_lahir,
          id_card_no AS nomor_identitas,
          id_card_type AS jenis_nomor_identitas,
          nationality AS nationalitas,
          job_title AS pekerjaan,
          family_type_desc AS hubungan_keluarga,
          gender AS jenis_kelamin,
          family_type_desc AS status_pernikahan,
          religiontext AS agama,
          married_status_date AS tanggal_menikah,
          tangdinas AS status_tanggungan_dinas,
          family_type_desc AS status_ahli_waris
        FROM SAFM_PEGAWAI_FAMILY
        WHERE PERNR IN (${pernrList})
```

</details>

#### Penjelasan

**Query kedua** ini digunakan untuk mengambil data keluarga pegawai berdasarkan `PERNR` yang diperoleh dari query pertama.

- `SELECT`: Mengambil kolom-kolom terkait informasi keluarga pegawai seperti nama, tempat dan tanggal lahir, nomor identitas, pekerjaan, hubungan keluarga, jenis kelamin, status pernikahan, agama, dan status tanggungan.
- `WHERE PERNR IN (${pernrList})`: Menyaring data berdasarkan daftar PERNR yang diberikan dari hasil query pertama.

#### Kesimpulan

**Query kedua** bertujuan untuk mengambil data keluarga pegawai berdasarkan `PERNR` yang diperoleh dari query pertama. Query ini mengembalikan informasi tentang **anggota keluarga pegawai**, termasuk nama, tempat dan tanggal lahir, nomor identitas, pekerjaan, hubungan keluarga, jenis kelamin, dan status pernikahan.

## API 4: Get Data Cuti Pegawai

### 1. Query Pertama

<details>
  <summary>Show SQL Query 1</summary>

```sql
WITH tpegawai AS ( SELECT msp.*,ROW_NUMBER () OVER(PARTITION BY msp.pernr, msp.pnalt_new ORDER BY msp.pernr ASC) AS row_num 
        FROM mdm_single_pegawai msp JOIN (SELECT DISTINCT PERNR FROM SAFM_CUTI_SPPD ) scs ON msp.PERNR = scs.PERNR  WHERE 1=1 ${appendQueryNipp} ${appendQueryPernr} ${appendQuery} )
        SELECT msp.PERNR,
        count(*) OVER() AS total_count ,
        msp.PNALT_NEW AS nipp_pegawai,
            msp.cname AS nama_pegawai
        FROM tpegawai msp
        JOIN (SELECT DISTINCT PERNR FROM SAFM_CUTI_SPPD) scs on msp.PERNR = scs.PERNR
        WHERE msp.pnalt_new IS NOT NULL and msp.row_num = 1 ${appendQueryNipp} ${appendQueryPernr} ${appendQuery}
        ORDER BY msp.LAST_UPDATED_DATE DESC
        OFFSET :offset ROWS FETCH NEXT :per_page ROWS ONLY
```
</details>

**Query pertama** digunakan untuk mendapatkan data pegawai beserta informasi terkait dari tabel `mdm_single_pegawai` dan `SAFM_CUTI_SPPD`, serta memfilter dan mengurutkannya berdasarkan beberapa kondisi. Hanya digunakan untuk paginasi dan menampilkan daftar pegawai berdasarkan kondisi yang ditentukan (misalnya, berdasarkan nipp, pernr, atau query tambahan lainnya).

#### 1. CTE (Common Table Expression) tpegawai

```sql
WITH tpegawai AS (
    SELECT msp.*, ROW_NUMBER() OVER(PARTITION BY msp.pernr, msp.pnalt_new ORDER BY msp.pernr ASC) AS row_num
    FROM mdm_single_pegawai msp
    JOIN (
        SELECT DISTINCT PERNR
        FROM SAFM_CUTI_SPPD
    ) scs ON msp.PERNR = scs.PERNR
    WHERE 1=1 ${appendQueryNipp} ${appendQueryPernr} ${appendQuery}
)
```

- `mdm_single_pegawai` (alias msp) adalah tabel utama yang diquery.
- `ROW_NUMBER()` digunakan untuk memberikan nomor urut pada setiap baris dalam partisi yang dikelompokkan berdasarkan `pernr` dan `pnalt_new`. Data diurutkan berdasarkan `pernr` secara ascending.
- `JOIN` dengan subquery yang memilih PERNR dari `SAFM_CUTI_SPPD` untuk hanya mendapatkan pegawai yang memiliki data cuti.
- `WHERE 1=1` adalah teknik umum untuk mempermudah penambahan kondisi dinamis.


#### 2. Mengambil Data dari CTE `tpegawai`

```sql
SELECT msp.PERNR,
       count(*) OVER() AS total_count,
       msp.PNALT_NEW AS nipp_pegawai,
       msp.cname AS nama_pegawai
FROM tpegawai msp
JOIN (SELECT DISTINCT PERNR FROM SAFM_CUTI_SPPD) scs on msp.PERNR = scs.PERNR
WHERE msp.pnalt_new IS NOT NULL AND msp.row_num = 1 ${appendQueryNipp} ${appendQueryPernr} ${appendQuery}
ORDER BY msp.LAST_UPDATED_DATE DESC
OFFSET :offset ROWS FETCH NEXT :per_page ROWS ONLY
```

- `count(*) OVER() AS total_count`: Menghitung total jumlah baris yang diambil.
- `WHERE msp.pnalt_new IS NOT NULL AND msp.row_num = 1`: Memastikan hanya baris pertama untuk setiap pegawai (berdasarkan `pnalt_new`) yang diambil, dan `pnalt_new` tidak boleh kosong.
- `ORDER BY msp.LAST_UPDATED_DATE DESC`: Mengurutkan hasil berdasarkan tanggal terakhir diperbarui.
- `OFFSET :offset ROWS FETCH NEXT :per_page ROWS ONLY`: Pagination; mengambil halaman tertentu dari hasil dengan jumlah baris yang ditentukan.

### 2. Query Kedua

<details>
  <summary>Show SQL Query 2</summary>

```sql
SELECT PERNR,
            ATT_ABSENCE_TYPE AS id_cuti,
            ATT_TYPE_TEXT AS jenis_cuti,
            STATUS AS status_pengajuan_cuti,
            CREATED_DATE AS tanggal_pengajuan_cuti,
            START_DATE AS tanggal_mulai_cuti,
            END_DATE AS tanggal_selesai_cuti
        FROM SAFM_CUTI_SPPD
        WHERE ATT_ABSENCE_TYPE NOT IN (`
        + `'5009', '4012', '4019', '4020',`
        + `'4021', '4045', '4046', '4047',`
        + `'4048', '5001', '5005', '5006',`
        + `'5007', '5009', '5010', '5013',`
        + `'5101'`
        + `) AND PERNR IN (${pernrList})
```
</details>

**Query kedua** mengambil data cuti pegawai yang sesuai dengan kondisi tertentu, tidak termasuk dalam daftar tipe cuti yang dikecualikan.

- `ATT_ABSENCE_TYPE NOT IN (...)`: Bagian ini dari query digunakan untuk mengecualikan beberapa jenis absensi berdasarkan kodenya. Kode-kode yang disebutkan dalam daftar tersebut tidak akan disertakan dalam hasil query.
- **Daftar Kode**: Kode-kode dalam daftar '5009', '4012', '4019', '4020', ... adalah tipe-tipe absensi yang ingin dikecualikan dari hasil query. Misalnya, jika kode '5009' mewakili "Cuti Sakit", maka query ini akan mengecualikan semua entri yang memiliki kode tersebut.
- `PERNR IN (${pernrList})`: Menyaring data untuk PERNR yang termasuk dalam daftar pernrList.

### Kesimpulan

**Query pertama** dan **kedua** memisahkan pengambilan data pegawai dan rincian cuti untuk meningkatkan efisiensi dan fokus masing-masing proses: **query pertama** mengumpulkan data dasar pegawai yang memiliki catatan cuti, termasuk total count dan informasi identitas pegawai, sementara **query kedua** memberikan rincian spesifik tentang pengajuan cuti untuk pegawai yang relevan. Pemisahan ini memungkinkan paginasi dan optimisasi berdasarkan kebutuhan yang berbeda, menghindari kompleksitas dan beban kinerja dari penggabungan query yang dapat menghambat performa sistem.

## API 5: Get Data Indisipliner Pegawai

### 1. Query Pertama

<details>
  <summary>Show SQL Query 1</summary>

```sql
WITH tpegawai AS ( SELECT msp.*, ROW_NUMBER () OVER(PARTITION BY msp.pernr, msp.pnalt_new ORDER BY msp.pernr ASC) AS row_num
        FROM mdm_single_pegawai msp JOIN (SELECT DISTINCT PERNR FROM SAFM_PEGAWAI_DISCIPLINE) spd on msp.PERNR = spd.PERNR  WHERE 1=1 ${appendQueryNipp} ${appendQueryPernr} ${appendQuery} )
        SELECT msp.PERNR,
            count(*) OVER() AS total_count ,
            msp.PNALT_NEW AS nipp_pegawai,
            msp.cname AS nama_pegawai
        FROM tpegawai msp
        JOIN (SELECT DISTINCT PERNR FROM SAFM_PEGAWAI_DISCIPLINE) spd on msp.PERNR = spd.PERNR
        WHERE msp.pnalt_new IS NOT NULL and msp.row_num = 1  ${appendQueryNipp} ${appendQueryPernr} ${appendQuery}
        ORDER BY msp.LAST_UPDATED_DATE DESC
        OFFSET :offset ROWS FETCH NEXT :per_page ROWS ONLY
```

</details>

**Query pertama** bertujuan untuk mendapatkan data pegawai yang memiliki catatan indisipliner. Berikut adalah langkah-langkah yang dilakukan:

#### 1. CTE `tpegawai`
- Data yang diambil dari tabel mdm_single_pegawai dalam query tersebut adalah data pegawai yang juga ada di tabel `SAFM_PEGAWAI_DISCIPLINE`. Pengambilan data ini didasarkan pada kolom `PERNR`, yang merupakan identifier unik untuk setiap pegawai.
- Data ditambahkan kolom `row_num` yang menghitung nomor urut berdasarkan pengelompokan (`PARTITION BY`) `pernr` dan `pnalt_new`. Data ini diurutkan berdasarkan `pernr` secara ascending.

#### 2. Query Utama
- Mengambil data `PERNR`, `PNALT_NEW` (sebagai `nipp_pegawai`), `cname` (sebagai `nama_pegawai`), dan menghitung total baris (`count(*) OVER() AS total_count`).
Bergabung dengan tabel `SAFM_PEGAWAI_DISCIPLINE` untuk memastikan hanya pegawai dengan catatan indisipliner yang dipilih.
- Filter menggunakan kondisi `pnalt_new IS NOT NULL dan row_num = 1` untuk memastikan hanya baris pertama dari setiap kelompok `PERNR` dan `PNALT_NEW` yang dipilih.
- Data diurutkan berdasarkan `LAST_UPDATED_DATE` secara descending dan menggunakan pagination dengan `OFFSET` dan `FETCH NEXT`.

### 2. Query Kedua

- Mengambil data pelanggaran, jenis hukuman, nomor SK, tanggal-tanggal terkait penalti, dan informasi lainnya dari tabel `SAFM_PEGAWAI_DISCIPLINE`.
- Hanya data dari pegawai yang ada dalam `pernrList` (hasil dari query pertama) yang diambil.

### Cara Penggunaan

1. **Jalankan Query Pertama**:
   - Hapus bagian `${appendQueryNipp} ${appendQueryPernr} ${appendQuery}`
   - Gunakan query pertama untuk mendapatkan daftar pegawai yang memiliki catatan indisipliner. Anda dapat mengatur filter tambahan seperti pernr, nipp, atau tanggal jika diperlukan.
   - Tentukan offset dan per_page untuk mengambil data dalam jumlah tertentu per halaman.
2. **Jalankan Query Kedua**:
   - Setelah mendapatkan `PERNR` dari query pertama, gunakan query kedua untuk mengambil detail pelanggaran terkait.
   - Masukkan `pernrList` yang diperoleh dari query pertama ke dalam query kedua untuk mendapatkan informasi detail mengenai pelanggaran.

### Kesimpulan

Query ini digunakan untuk mendapatkan data pegawai yang memiliki catatan indisipliner, kemudian menampilkan detail pelanggaran mereka. Query pertama mengambil daftar pegawai berdasarkan filter tertentu, sedangkan query kedua mengambil detail pelanggaran untuk pegawai tersebut.