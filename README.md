<div align="center">

<img width="1918" height="634" alt="image" src="https://github.com/user-attachments/assets/014b803f-4985-4353-b229-16a734537bfe" />

# ◈ Spotify Intelligence
### *Perancangan Data Warehouse untuk Analisis Perilaku Pengguna Spotify*

<p>
  <img src="https://img.shields.io/badge/Python-3.x-3776AB?style=for-the-badge&logo=python&logoColor=white"/>
  <img src="https://img.shields.io/badge/SQLite-Database-003B57?style=for-the-badge&logo=sqlite&logoColor=white"/>
  <img src="https://img.shields.io/badge/Google_Colab-Notebook-F9AB00?style=for-the-badge&logo=googlecolab&logoColor=white"/>
  <img src="https://img.shields.io/badge/pandas-ETL-150458?style=for-the-badge&logo=pandas&logoColor=white"/>
  <img src="https://img.shields.io/badge/Chart.js-Dashboard-FF6384?style=for-the-badge&logo=chartdotjs&logoColor=white"/>
</p>

<p>
  <a href="https://syzota.github.io/spotify-intelligence/">
    <img src="https://img.shields.io/badge/◉ Live Dashboard-syzota.github.io-1ED760?style=for-the-badge"/>
  </a>
  <a href="https://colab.research.google.com/drive/1Y6BFXDKCow11MJaKHHlsmObZ10pHrpz">
    <img src="https://img.shields.io/badge/◉ Open in Colab-Notebook-F9AB00?style=for-the-badge"/>
  </a>
</p>

> *Mengubah data streaming mentah menjadi insight bisnis yang actionable — dari raw CSV hingga dashboard analitik interaktif.*

<br/>

| 50.000 | 5 Tabel | 7 Analisis | 3 Dataset |
|:------:|:-------:|:----------:|:---------:|
| Total Plays | Star Schema | Query OLAP | Sumber Kaggle |

</div>

---

## ◆ Daftar Isi

- [Tentang Proyek](#-tentang-proyek)
- [Sumber Data](#-sumber-data)
- [Arsitektur Sistem](#-arsitektur-sistem)
- [ETL Pipeline](#-etl-pipeline)
- [Struktur Data Warehouse](#-struktur-data-warehouse)
- [Analisis & Insight](#-analisis--insight)
- [Dashboard](#-dashboard)
- [Struktur Repositori](#-struktur-repositori)
- [Tech Stack](#-tech-stack)
- [Tim Pengembang](#-tim-pengembang)

---

## ▸ Tentang Proyek

Proyek ini adalah bagian dari **Ujian Akhir Semester mata kuliah Business Intelligence** — membangun data warehouse lengkap untuk menganalisis perilaku pengguna platform streaming Spotify, dengan fokus khusus pada fenomena **churn**.

Platform streaming menghadapi tantangan serius mempertahankan pengguna. Biaya akuisisi pelanggan baru bisa mencapai **5–7× lebih mahal** dibanding mempertahankan yang sudah ada. Analisis perilaku pengguna menjadi sangat strategis untuk keberlangsungan bisnis.

**Rumusan Masalah yang Dijawab:**

- Bagaimana merancang data warehouse yang mengintegrasikan data pengguna Spotify secara terstruktur?
- Bagaimana mengimplementasikan ETL pipeline menggunakan Google Colab?
- Pola perilaku apa yang berpotensi menyebabkan churn pengguna?

---

## ▸ Sumber Data

Seluruh data bersumber dari dataset publik Spotify di **Kaggle**, format CSV, diolah dengan Google Colab.

| Dataset | Sumber | Baris | Kolom | Peran |
|---------|--------|:-----:|:-----:|-------|
| Spotify Churn Analysis | [nabihazahid](https://www.kaggle.com/datasets/nabihazahid/spotify-dataset-for-churn-analysis) | 8.000 | 12 | Tabel Fakta utama |
| Spotify SQL Exploration | [ambaliyagati](https://www.kaggle.com/datasets/ambaliyagati/spotify-dataset-for-playing-around-with-sql) | 1.686 | 29 | Dim Track & Playlist |
| Spotify Music Dataset | [solomonameh](https://www.kaggle.com/datasets/solomonameh/spotify-music-dataset) | 6.300 | 8 | Enrichment Track |

### Detail Atribut

**Churn Analysis** — user_id, gender, age, country, subscription_type, listening_time, songs_played_per_day, skip_rate, device_type, ads_listened_per_week, offline_listening, is_churned

**SQL Exploration** — track_id, track_name, artist, album, playlist_name, playlist_genre, energy, danceability, tempo, loudness, valence, acousticness, instrumentalness, speechiness, + 15 atribut lainnya

**Spotify Music** — track_id, track_name, artist, album, genre, popularity, duration_ms, explicit

---

## ▸ Arsitektur Sistem

```
┌─────────────┐     ┌──────────────────────────┐     ┌───────────────┐     ┌─────────────┐
│  Kaggle CSV  │────▶│     ETL · Google Colab    │────▶│  SQLite · DW  │────▶│  Dashboard  │
│  3 datasets  │     │  Extract · Transform · Load│     │  Star Schema  │     │  HTML + JS  │
└─────────────┘     └──────────────────────────┘     └───────────────┘     └─────────────┘
```

---

## ▸ ETL Pipeline

### Extract

Data dibaca langsung dari Google Drive menggunakan `pandas.read_csv()`. Validasi awal mencakup pengecekan shape, missing values, dan duplikasi.

```python
df_tracks   = pd.read_csv(path + "spotify_tracks.csv")                  # (6300, 8)
df_playlist = pd.read_csv(path + "high_popularity_spotify_data.csv")    # (1686, 29)
df_users    = pd.read_csv(path + "spotify_churn_dataset.csv")           # (8000, 12)
```

### Transform

```python
# Standarisasi nama kolom
df_tracks_clean = df_tracks.rename(columns={
    "id": "track_id", "name": "track_name",
    "artists": "artist_name", "popularity": "track_popularity"
})

# Atribut turunan
df_tracks_clean["duration_min"]    = (df_tracks_clean["duration_ms"] / 60000).round(2)
df_tracks_clean["popularity_level"] = pd.cut(df_tracks_clean["track_popularity"],
    bins=[-1, 30, 70, 100], labels=["Low", "Medium", "High"])

df_users_clean["age_group"]       = pd.cut(df_users_clean["age"],
    bins=[0, 18, 25, 35, 50, 100], labels=["<18", "18-25", "26-35", "36-50", "50+"])
df_users_clean["engagement_level"] = pd.cut(df_users_clean["listening_time"],
    bins=[-1, 120, 240, 10000], labels=["Low", "Medium", "High"])
```

**Transformasi yang dilakukan:**
- Standarisasi nama kolom antar dataset
- Konversi tipe data: `explicit` → boolean, tanggal → datetime, fitur audio → numeric
- Penambahan atribut turunan: `duration_min`, `popularity_level`, `age_group`, `engagement_level`
- Penanganan missing values & deduplikasi berdasarkan primary key
- Join antar dataset via `track_id` — ditemukan 65 overlapping records

### Load

```python
# Buat struktur Star Schema di SQLite
cursor.executescript("""
    CREATE TABLE Fact_Listening (
        listening_id INTEGER PRIMARY KEY,
        user_id INTEGER,  track_id TEXT,  playlist_id TEXT,  time_id INTEGER,
        listening_time REAL,  songs_played_per_day REAL,  skip_rate REAL,
        FOREIGN KEY (user_id) REFERENCES Dim_User(user_id),
        FOREIGN KEY (track_id) REFERENCES Dim_Track(track_id), ...
    );
""")

# Load semua tabel
dim_time.to_sql("Dim_Time", conn, if_exists="append", index=False)
dim_user.to_sql("Dim_User", conn, if_exists="append", index=False)
dim_playlist.to_sql("Dim_Playlist", conn, if_exists="append", index=False)
dim_track.to_sql("Dim_Track", conn, if_exists="append", index=False)
fact_listening.to_sql("Fact_Listening", conn, if_exists="append", index=False)
```

---

## ▸ Struktur Data Warehouse

Model **Star Schema** — satu tabel fakta sentral yang dikelilingi empat tabel dimensi, disimpan dalam satu file `spotify_dw.db`.

```
                    ┌─────────────┐
                    │   Dim_User  │
                    │  8.000 rows │
                    └──────┬──────┘
                           │ user_id
          ┌────────────────▼────────────────┐
          │          Fact_Listening          │     ◄── 50.000 rows
          │  listening_id · user_id          │
          │  track_id · playlist_id          │
          │  time_id · listening_time        │
          │  songs_played_per_day · skip_rate│
          └──┬──────────┬──────────┬─────────┘
             │track_id  │playlist_id│time_id
    ┌────────▼──┐  ┌────▼───────┐  ┌──▼──────────┐
    │ Dim_Track │  │Dim_Playlist│  │  Dim_Time   │
    │ 7.559 rows│  │  72 rows   │  │ 26.298 rows │
    └───────────┘  └────────────┘  └─────────────┘
```

| Tabel | Jenis | Jumlah Data | Keterangan |
|-------|:-----:|:-----------:|------------|
| `Fact_Listening` | Fakta | 50.000 baris | Aktivitas streaming: user, track, playlist, waktu |
| `Dim_User` | Dimensi | 8.000 baris | Demografi, subscription, engagement level |
| `Dim_Track` | Dimensi | 7.559 baris | Metadata lagu + 11 fitur audio |
| `Dim_Playlist` | Dimensi | 72 baris | Info playlist, genre, subgenre |
| `Dim_Time` | Dimensi | 26.298 baris | Kalender harian 1954–2025 |

---

## ▸ Analisis & Insight

Query OLAP multidimensional langsung ke SQLite menggunakan fungsi `run_query()`.

### ① Genre Popularity

```sql
SELECT t.genre, COUNT(*) AS total_plays,
       ROUND(AVG(f.track_popularity),2) AS avg_popularity,
       ROUND(AVG(f.skip_rate),4) AS avg_skip_rate
FROM Fact_Listening f JOIN Dim_Track t ON f.track_id = t.track_id
WHERE t.genre != 'Unknown'
GROUP BY t.genre ORDER BY avg_popularity DESC LIMIT 10
```

> **◆** Rock memimpin dengan avg popularity **70,20** dan listening time tertinggi **207,98 dtk**  
> **◆** Country paling loyal — skip rate terendah **0,2807**  
> **◆** Pop memiliki skip rate tertinggi **0,3329** meski popularitasnya tinggi

---

### ② Churn by Subscription

| Subscription | Churn Rate | Avg Skip Rate | Avg Listening Time |
|:------------:|:----------:|:-------------:|:-----------------:|
| Family | **26,79%** ▲ | 0,3028 | 197,87 dtk |
| Student | 26,60% | 0,3020 | 201,07 dtk |
| Premium | 25,33% | **0,2909** ▼ | **201,21 dtk** ▲ |
| Free | 24,54% | 0,3001 | 200,41 dtk |

> **◆** Family plan paling berisiko churn dengan engagement paling rendah  
> **◆** Premium paling engaged: skip rate terendah, listening time tertinggi

---

### ③ Listening Behavior by Country

| Negara | Total Plays | Churn Rate | Avg Skip Rate |
|:------:|:-----------:|:----------:|:-------------:|
| US (Amerika Serikat) | **6.507** ▲ | 27,06% | 0,2945 |
| DE (Jerman) | 6.401 | 26,15% | 0,2998 |
| AU (Australia) | 6.346 | 25,76% | 0,3000 |
| PK (Pakistan) | 6.337 | **27,54%** ▲ | 0,3030 |

> **◆** US memimpin volume, tapi Pakistan paling berisiko churn

---

### ④ Popularity Level vs Skip Rate

| Popularity Level | Avg Skip Rate | Total Plays |
|:----------------:|:-------------:|:-----------:|
| Medium | **0,2979** ▼ | 13.590 |
| Low | 0,2980 | 6.623 |
| High | 0,2993 ▲ | 29.787 |

> **◆** Lagu *High* popularity belum tentu lebih didengarkan penuh — Medium justru paling jarang di-skip

---

### ⑤ Trend Plays per Tahun (2020–2025)

| Tahun | Total Plays | Avg Listening Time | Avg Skip Rate |
|:-----:|:-----------:|:-----------------:|:-------------:|
| 2020 | 8.232 | 201,56 dtk | 0,2990 |
| 2021 | **8.495** ▲ | 199,64 dtk | 0,3010 |
| 2022 | 8.206 | **201,71 dtk** ▲ | 0,2973 |
| 2023 | 8.299 | 198,24 dtk | 0,2988 |
| 2024 | 8.397 | 199,96 dtk | **0,2972** ▼ |
| 2025 | 8.371 | 200,08 dtk | 0,2993 |

> **◆** Aktivitas pengguna stabil — selisih antar tahun tidak signifikan

---

### ⑥ Active vs Churned Users

| Status | Avg Skip Rate | Avg Listening Time | Total Plays |
|:------:|:-------------:|:-----------------:|:-----------:|
| Active | 0,2979 | **200,77 dtk** | **37.108** |
| Churned | 0,3013 | 198,52 dtk | 12.892 |

> **◆** Churn lebih didorong **rendahnya engagement**, bukan karakteristik audio lagu  
> **◆** Fitur audio (energy, danceability, valence, tempo) hampir identik antara dua kelompok

---

### ⑦ Playlist Genre by Age Group

| Age Group | Top Genre | Total Plays | Avg Listening Time | Avg Skip Rate |
|:---------:|:---------:|:-----------:|:-----------------:|:-------------:|
| 18–25 | Hip-hop | **1.182** | 201,82 dtk | 0,2982 |
| 18–25 | Rock | 909 | **205,02 dtk** | 0,2972 |
| 18–25 | Electronic | 904 | 203,67 dtk | **0,2940** |
| <18 | Indie | 42 | 187,29 dtk | 0,3576 |
| <18 | Turkish | 40 | 184,15 dtk | 0,3465 |

> **◆** Hip-hop dominan di 18–25 tapi Rock & Electronic unggul dari durasi dengar  
> **◆** Usia <18 punya skip rate tertinggi — preferensi lebih volatile

---

## ▸ Dashboard

Dashboard interaktif berbasis HTML + Chart.js dengan tema dark Spotify.

> ◉ **Live:** [syzota.github.io/spotify-intelligence](https://syzota.github.io/spotify-intelligence/)

**Komponen Dashboard:**

| Visualisasi | Deskripsi |
|-------------|-----------|
| KPI Cards | Total plays, genre terpopuler, churn rate tertinggi, negara paling aktif |
| Bar + Line | Top 10 genre: avg popularity & avg listening time |
| Doughnut | Distribusi churn rate per tipe langganan |
| Grouped Bar | Listening behavior per negara vs churn rate |
| Line Chart | Trend plays per tahun 2020–2025 |
| Bar Chart | Popularity level vs skip rate |
| Bar + Line | Active vs Churned users comparison |
| Data Table | Playlist genre × kelompok umur (140 kombinasi) |

---

## ▸ Struktur Repositori

```
spotify-intelligence/
├── dataset/
│   ├── spotify_churn_dataset.csv
│   ├── high_popularity_spotify_data.csv
│   └── spotify_tracks.csv
├── notebook/
│   └── spotify_bi_etl.ipynb          ← Google Colab notebook
├── database/
│   └── spotify_dw.db                 ← SQLite data warehouse
├── dashboard/
│   └── index.html                    ← Dashboard interaktif
├── docs/
│   └── Laporan_UTS_BI_Spotify.pdf
└── README.md
```

---

## ▸ Tech Stack

<p>
  <img src="https://img.shields.io/badge/Python-pandas · numpy · matplotlib · seaborn-3776AB?style=flat-square&logo=python&logoColor=white"/>
  <img src="https://img.shields.io/badge/SQLite-sqlite3-003B57?style=flat-square&logo=sqlite&logoColor=white"/>
  <img src="https://img.shields.io/badge/Google_Colab-Jupyter_Notebook-F9AB00?style=flat-square&logo=googlecolab&logoColor=white"/>
  <img src="https://img.shields.io/badge/HTML_+_CSS-Dashboard-E34F26?style=flat-square&logo=html5&logoColor=white"/>
  <img src="https://img.shields.io/badge/Chart.js-Visualisasi-FF6384?style=flat-square&logo=chartdotjs&logoColor=white"/>
</p>

| Layer | Tools |
|-------|-------|
| Data Processing | Python 3, pandas, numpy |
| Database | SQLite, sqlite3 |
| Environment | Google Colab, Google Drive |
| Visualization | matplotlib, seaborn, Chart.js |
| Dashboard | HTML, CSS, JavaScript |

---

## ▸ Tim Pengembang

| NIM | Nama | Kontribusi |
|-----|------|------------|
| 2409116002 | **Nurhidayah** | ETL Pipeline, Transformasi Data |
| 2409116007 | **Jen Agresia Misti** | Analisis OLAP & Visualisasi |
| 2409116015 | **Putri Syafana Afrillia** | Data Warehouse Design, Dashboard HTML |
| 2409116040 | **Dhita Olivia R. K.** | Dokumentasi & Laporan |

> **Mata Kuliah** — Business Intelligence  
> **Dosen Pengampu** — Dr. Akhmad Irsyad S.T.,M.Kom.
> **Program Studi** — Sistem Informasi · Fakultas Teknik · Universitas Mulawarman · 2025/2026

---

<div align="center">

*© 2025 · Sistem Informasi · Fakultas Teknik · Universitas Mulawarman*

</div>
