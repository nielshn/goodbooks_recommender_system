# 📚 Goodreads Book Recommendation System

## 1. Project Overview

### Domain Proyek

Proyek ini berada dalam domain **sistem rekomendasi berbasis literasi digital**. Dengan meningkatnya jumlah konten bacaan, pengguna platform seperti Goodreads mengalami kesulitan dalam menemukan buku yang relevan dengan minat mereka. Masalah yang dihadapi adalah **information overload** dan **minimnya personalisasi rekomendasi**.

Sistem rekomendasi dapat menjadi solusi untuk menyaring konten dan meningkatkan kepuasan pengguna. Ricci et al. (2015) menyatakan bahwa sistem rekomendasi yang baik membantu pengguna dalam menemukan konten yang relevan dan meningkatkan interaksi dalam platform digital ([link](https://link.springer.com/book/10.1007/978-1-4899-7637-6)). Oleh karena itu, sistem ini dibangun untuk membantu pengguna memilih buku melalui pendekatan berbasis konten dan kolaboratif.

---

## 2. Business Understanding

### Problem Statements

- Bagaimana sistem dapat membantu pengguna menemukan buku sesuai preferensi mereka?
- Bagaimana sistem dapat memberikan rekomendasi yang akurat untuk pengguna baru dan pengguna aktif?

### Goals

- Membangun sistem rekomendasi Top-N yang personal dan akurat.
- Mengimplementasikan dua pendekatan: Content-Based Filtering dan Collaborative Filtering.
- Mengevaluasi performa kedua pendekatan dan membandingkannya berdasarkan metrik evaluasi.

### Solution Approach

1. **Content-Based Filtering (CBF)**:

   - Menggunakan TF-IDF vectorizer dari fitur judul dan penulis.
   - Menghitung cosine similarity antar buku untuk menentukan buku yang mirip.

2. **Collaborative Filtering (CF)**:

   - Menggunakan algoritma **Singular Value Decomposition (SVD)** dari library `surprise`.
   - Memprediksi rating user terhadap buku yang belum dibaca.

---

## 3. Data Understanding

Dataset diambil dari [Goodbooks-10k GitHub](https://github.com/zygmuntz/goodbooks-10k)

### Dataset 1: `books.csv`

- Jumlah: 10.000 baris × 23 kolom
- Kondisi: tidak ditemukan missing value
- Fitur penting: `book_id`, `title`, `authors`, `average_rating`, `language_code`, `ratings_count`

### Dataset 2: `ratings.csv`

- Jumlah: 6.000.000+ baris × 3 kolom
- Kondisi:

  - Tidak ada missing value
  - Terdapat duplikat yang dibersihkan
  - Rating bernilai 0 dihapus karena tidak valid

### Penjelasan Fitur:

- `book_id`: ID unik untuk setiap buku
- `user_id`: ID unik untuk setiap pengguna
- `rating`: Nilai rating dari pengguna (skala 1–5)
- `title`: Judul buku
- `authors`: Penulis buku
- `average_rating`: Rata-rata rating buku
- `ratings_count`: Jumlah total rating buku

### Eksplorasi Data:

- Sebagian besar buku memiliki rating rata-rata antara 3.5–4.5
- Distribusi rating didominasi oleh skor 4 dan 5 (bias positif)
- Mayoritas pengguna memberikan sedikit rating (long tail distribution)

---

## 4. Data Preparation

![Data Preraparation](<data preparation.png>)

### Langkah-Langkah

1. **Cleaning**:

   - Menghapus rating 0
   - Menghapus duplikat

2. **Filtering**:

   - Memilih user yang memberikan ≥10 rating
   - Memilih buku dengan ≥50 rating agar matriks tidak terlalu sparse

3. **CBF**:

   - Menggabungkan `title` dan `authors`
   - Menggunakan TF-IDF vectorizer → cosine similarity antar buku

4. **CF (SVD)**:

   - Mengatur skala rating (1–5)
   - Menggunakan matrix user-item (rating)
   - Melatih model dengan 5-fold cross validation

### Alasan Tahapan:

- Mengurangi noise dari pengguna/buku yang jarang aktif
- Mempercepat konvergensi dan akurasi model
- Memastikan relevansi rekomendasi dan skalabilitas sistem

---

## 5. Modeling and Results

### 5.1 Content-Based Filtering

- Input: Judul buku
- Output: Top-10 buku yang paling mirip secara konten
- Algoritma: TF-IDF + Cosine Similarity

📌 _Contoh Rekomendasi untuk buku "The Hunger Games":_

| Rank | Recommended Book Title |
| ---- | ---------------------- |
| 1    | Catching Fire          |
| 2    | Mockingjay             |
| 3    | Divergent              |
| 4    | Insurgent              |
| 5    | The Maze Runner        |
| 6    | Allegiant              |
| 7    | City of Bones          |
| 8    | Delirium               |
| 9    | Twilight               |
| 10   | The Giver              |

### 5.2 Collaborative Filtering (SVD)

- Model: `SVD()` dari `surprise`
- Validasi: K-Fold (n_splits=5)
- Output: Prediksi rating user untuk buku

📌 _Contoh Prediksi Rating User ID 3:_

| Book Title             | Predicted Rating |
| ---------------------- | ---------------- |
| The Fault in Our Stars | 4.21             |
| Divergent              | 4.14             |
| The Hunger Games       | 4.02             |
| The Maze Runner        | 3.95             |
| City of Bones          | 3.90             |

---

## 6. Evaluation

### Metrik yang Digunakan

- **Root Mean Square Error (RMSE)**
- **Mean Absolute Error (MAE)**

### Rumus:

$RMSE = \sqrt{\frac{1}{n} \sum_{i=1}^{n}(y_i - \hat{y}_i)^2}$
$MAE = \frac{1}{n} \sum_{i=1}^{n}|y_i - \hat{y}_i|$

### Hasil Evaluasi (Collaborative Filtering):

| Fold | RMSE  | MAE   |
| ---- | ----- | ----- |
| 1    | 0.884 | 0.679 |
| 2    | 0.882 | 0.677 |
| 3    | 0.885 | 0.680 |
| 4    | 0.879 | 0.676 |
| 5    | 0.887 | 0.681 |

📌 _Rata-rata RMSE: 0.883, MAE: 0.679_ → cukup baik untuk skala rating 1–5

📌 _Content-Based tidak dievaluasi kuantitatif, hanya secara visual/top-N relevansi_

---

## 7. Business Impact

- 📘 Personalisasi rekomendasi untuk pengguna aktif dan baru
- 🔄 Dapat diskalakan ke data yang lebih besar
- 🤝 Meningkatkan engagement pengguna di platform literatur
- 🧠 Insight: Model hybrid direkomendasikan sebagai next step

---

## 8. Conclusion

- Dua pendekatan digunakan: Content-Based dan Collaborative Filtering
- CF dengan SVD menunjukkan performa terbaik secara metrik
- CBF tetap relevan untuk pengguna baru
- Proyek membuktikan efektivitas sistem rekomendasi buku berbasis data

✅ _Langkah selanjutnya_: membangun hybrid recommender system yang menggabungkan kekuatan kedua pendekatan

---

## 📚 Referensi

- Ricci, F., Rokach, L., & Shapira, B. (2015). [Recommender Systems Handbook](https://link.springer.com/book/10.1007/978-1-4899-7637-6). Springer.
- Aggarwal, C. C. (2016). _Recommender Systems: The Textbook_. Springer.
- Bobadilla, J., Ortega, F., Hernando, A., & Gutiérrez, A. (2013). Recommender systems survey. _Knowledge-Based Systems_, 46, 109–132.
- Koren, Y., & Bell, R. (2015). Advances in collaborative filtering. In _Recommender Systems Handbook_ (pp. 77–118). Springer.
