# Analisis Graf Jaringan Transfer dan Diversitas Pemain Klub Besar Eropa

Proyek ini merupakan tugas besar untuk mata kuliah **Graf** yang berfokus pada integrasi data dari *Linked Open Data* (LOD) dan analisis jaringan menggunakan *Graph Database*. Kami memetakan distribusi kewarganegaraan atlet pada klub raksasa Liga Inggris untuk melihat pola rekrutmen dan dominasi talenta global.

---

## 👥 Anggota
* **Fachreza Aptadhi Kurniawan** (5026231135)
* **Sultan Alamsyah Lintang Mubarok** (5026231188)

---

## 📖 Sinopsis Studi Kasus
Dalam industri sepak bola modern, keberagaman kewarganegaraan dalam sebuah tim mencerminkan strategi rekrutmen dan jangkauan *scouting* sebuah klub. Proyek ini mengambil studi kasus pada klub "Big Six" (fokus pada **Chelsea**, **Manchester United**, dan **Arsenal**) untuk menjawab:
1. Klub mana yang memiliki kapasitas rekrutmen paling masif?
2. Negara mana yang menjadi penyuplai talenta terbesar bagi klub-klub elit tersebut?
3. Seberapa mirip strategi rekrutmen antar klub raksasa berdasarkan basis geografis pemain mereka?

Kami mengintegrasikan data dari **Wikidata** (untuk detail kewarganegaraan) dan **DBpedia Indonesia** (untuk metadata entitas lokal) untuk membangun sebuah graf tripartit yang menghubungkan **Atlet**, **Klub**, dan **Negara**.

---

## 🛠️ Teknologi & Alur Kerja
* **Ekstraksi Data:** Menggunakan kueri **SPARQL** untuk menarik data dari Wikidata Query Service dan DBpedia Indonesia.
* **Integrasi Data:** Menggunakan **Python (Pandas)** untuk proses *cleaning*, normalisasi *string*, dan deduplikasi entitas lintas sumber.
* **Penyimpanan Graf:** Data diimpor ke **Neo4j Desktop** dengan skema relasi `(:Athlete)-[:PLAYS_FOR]->(:Club)` dan `(:Athlete)-[:FROM]->(:Country)`.
* **Analisis Graf:** Menerapkan algoritma **Weighted Degree Centrality** dan **Jaccard Similarity** melalui library GDS atau APOC.

---

## 📊 Hasil Analisis & Temuan Utama

### 1. Dominansi Kapasitas Rekrutmen (Weighted Degree)
Berdasarkan algoritma *Weighted Degree Centrality*, ditemukan dominasi volume data pada entitas berikut:

| Peringkat | Nama Klub | Weighted Degree |
| :--- | :--- | :--- |
| 1 | Chelsea F.C. | 816 |
| 2 | Manchester United F.C. | 628 |
| 3 | Arsenal F.C. | 398 |

### 2. Geopolitik Penyuplai Pemain
Negara dengan kontribusi pemain terbanyak dalam jaringan:
* **United Kingdom:** 1.267 (Dominasi domestik)
* **France:** 41
* **Scotland:** 32
* **Brazil:** 28

### 3. Kemiripan Strategi (Jaccard Similarity)
Kami mengukur kemiripan profil negara asal pemain antar klub (skor 0-1):

| Pasangan Klub | Shared Countries | Jaccard Score |
| :--- | :--- | :--- |
| Chelsea & Manchester United | 34 | **0.453** |
| Arsenal & Chelsea | 30 | 0.380 |
| Arsenal & Manchester United | 22 | 0.379 |

**Insight:** Chelsea dan Manchester United memiliki skor tertinggi (0.453), menunjukkan bahwa kedua klub bersaing di pasar talenta yang sangat identik.

---

## 🚀 Cara Menjalankan
1.  Jalankan notebook `gabung_csv_oke.ipynb` untuk menghasilkan `combined_athlete_club.csv`.
2.  Buka **Neo4j Desktop** dan buat database baru.
3.  Instal plugin **APOC** melalui menu Plugins.
4.  Pindahkan file CSV ke folder `import` pada project Neo4j kamu.
5.  Gunakan perintah Cypher berikut untuk memuat data:

```cypher
LOAD CSV WITH HEADERS FROM 'file:///combined_athlete_club.csv' AS row
MERGE (a:Athlete {name: row.athlete})
MERGE (co:Country {name: row.country})
MERGE (cl:Club {name: row.club})
MERGE (a)-[:FROM]->(co)
MERGE (a)-[:PLAYS_FOR]->(cl);

MATCH (a:Athlete)-[r1:PLAYS_FOR]->(cl:Club)
MATCH (a)-[r2:FROM]->(co:Country)
RETURN a, r1, cl, r2, co
LIMIT 1000;
