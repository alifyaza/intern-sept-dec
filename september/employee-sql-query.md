# SQL Query API Explanation

SQL Query API didapatkan dari spreadsheet [Mapping Data Portaverse MDM](https://docs.google.com/spreadsheets/d/1su75jhJ9AiyeI3ncOPzaNgRqkYcNSAYCt2AdlEyGPTA/edit?gid=1157891821#gid=1157891821)

## Table of Contents
- [SQL Query API Explanation](#sql-query-api-explanation)
  - [Table of Contents](#table-of-contents)
  - [API 1: Get Identitas Pegawai](#api-1-get-identitas-pegawai)
    - [1. CTE (Common Table Expression) `tpegawai`](#1-cte-common-table-expression-tpegawai)
    - [2. Query Utama](#2-query-utama)
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
| `sp.OFFICMAIL`                                   | `email_korporat`                  | Email korporat pegawai                       |  `SAFM_PEGAWAI`                           |
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





