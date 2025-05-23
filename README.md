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

- Ukuran: **10.000 baris × 23 kolom**
- Kondisi: Terdapat beberapa kolom dengan nilai null (seperti `isbn`, `isbn13`, `original_title`, dan `language_code`)

### Fitur `books.csv`

1. `book_id`: ID unik buku
2. `goodreads_book_id`: ID buku di platform Goodreads
3. `best_book_id`: ID terbaik dari versi buku
4. `work_id`: ID karya
5. `books_count`: Total versi buku yang tersedia
6. `isbn`: ISBN
7. `isbn13`: ISBN versi 13 digit
8. `authors`: Nama penulis
9. `original_publication_year`: Tahun terbit asli
10. `original_title`: Judul asli
11. `title`: Judul buku
12. `language_code`: Kode bahasa buku
13. `average_rating`: Rata-rata rating
14. `ratings_count`: Jumlah total rating
15. `work_ratings_count`: Jumlah rating berdasarkan ID karya
16. `work_text_reviews_count`: Jumlah ulasan teks
17. `ratings_1` s/d `ratings_5`: Jumlah masing-masing rating bintang
18. `image_url`, `small_image_url`: Link gambar buku

### Dataset 2: `ratings.csv`

- Ukuran: **5.976.479 baris × 3 kolom**
- Kondisi: Tidak ada missing value, namun ditemukan duplikat dan rating bernilai 0

### Fitur `ratings.csv`

1. `user_id`: ID pengguna unik
2. `book_id`: ID buku
3. `rating`: Nilai rating (1–5)

### Insight

- Distribusi rating bias ke skor tinggi (4–5)
- Mayoritas pengguna hanya memberi sedikit rating (long-tail)
- Outlier terdeteksi pada `ratings_count` buku tertentu (misalnya Harry Potter)

---

## 4. Data Preparation

![Data Preparation](<data preparation.png>)
Tahapan data preparation dilakukan secara sistematis sebagai berikut:

1. **Menghapus Duplikasi**

   - Duplikasi pada `book_id` di `books.csv` dan seluruh baris pada `ratings.csv` dihapus untuk memastikan data unik.
   - `books_df.drop_duplicates(subset='book_id', inplace=True)`
   - `ratings_df.drop_duplicates(inplace=True)`

2. **Menghapus Rating Tidak Valid**

   - Rating dengan nilai 0 dihapus karena tidak merepresentasikan preferensi pengguna.
   - `ratings_df = ratings_df[ratings_df['rating'] > 0]`

3. **Filter User Aktif**

   - Hanya pengguna yang memberikan lebih dari 10 rating yang dipertahankan untuk mengurangi sparsity.
   - `active_users = ratings_df['user_id'].value_counts()`
   - `ratings_df = ratings_df[ratings_df['user_id'].isin(active_users[active_users > 10].index)]`

4. **Menggabungkan Fitur Metadata Buku**

   - Kolom `title` dan `authors` digabungkan menjadi satu fitur teks untuk analisis content-based.
   - `books_df['combined'] = books_df['title'] + ' ' + books_df['authors']`

5. **Splitting Data**
   - Data dibagi menjadi train-test menggunakan cross-validation untuk collaborative filtering.

### Alasan Tahapan

- Memastikan model menerima input bersih dan berkualitas
- Mengurangi sparsity pada CF
- Menjamin kecepatan training dan keakuratan rekomendasi

---

## 5. Modeling and Results

Pada tahap ini, fokus pada penjelasan sistem rekomendasi dan algoritma yang digunakan, tanpa membahas ulang pemrosesan data.

### 5.1 Content-Based Filtering (CBF)

- **Definisi**: Sistem merekomendasikan buku berdasarkan kemiripan konten (judul dan penulis).
- **Cara Kerja Cosine Similarity**:  
  Setiap buku direpresentasikan sebagai vektor fitur (hasil TF-IDF). Cosine similarity mengukur sudut antara dua vektor, menghasilkan nilai antara 0 (tidak mirip) hingga 1 (sangat mirip). Buku dengan nilai cosine similarity tertinggi terhadap buku input akan direkomendasikan.
- **Output**: 5 buku paling mirip berdasarkan skor similarity tertinggi.

📌 _Top-5 Rekomendasi untuk "The Hobbit":_

| Rank | Recommended Book Title                       | Author                              |
| ---- | -------------------------------------------- | ----------------------------------- |
| 1    | J.R.R. Tolkien 4-Book Boxed Set              | J.R.R. Tolkien                      |
| 2    | The History of the Hobbit, Part One          | John D. Rateliff, J.R.R. Tolkien    |
| 3    | The Children of Húrin                        | J.R.R. Tolkien, Christopher Tolkien |
| 4    | The Hobbit: Graphic Novel                    | Chuck Dixon, J.R.R. Tolkien         |
| 5    | Unfinished Tales of Númenor and Middle-Earth | J.R.R. Tolkien, Christopher Tolkien |

### 5.2 Collaborative Filtering (SVD)

- **Definisi**: Sistem merekomendasikan buku berdasarkan pola rating pengguna lain.
- **Cara Kerja SVD**:  
  SVD (Singular Value Decomposition) memfaktorkan matriks user-item menjadi tiga matriks yang lebih kecil, sehingga dapat memprediksi rating yang belum diberikan user terhadap buku tertentu. Model ini efektif untuk menangkap pola laten preferensi pengguna.
- **Validasi**: Menggunakan K-Fold Cross Validation (3-Fold).
- **Output**: Prediksi rating buku untuk setiap user.

- Library: `surprise`
- Model: `SVD()`
- Data: Matriks user-book dari hasil filtering
- Validasi: K-Fold 3 kali (cross_val_score)
- Output: Prediksi rating buku berdasarkan user

📌 _Top-5 Rekomendasi untuk user_id = 42:_

| Rank | Book Title                                              | Author                       |
| ---- | ------------------------------------------------------- | ---------------------------- |
| 1    | Harry Potter and the Deathly Hallows (Harry Po...       | J.K. Rowling, Mary GrandPré  |
| 2    | Harry Potter Boxset (Harry Potter, #1-7)                | J.K. Rowling                 |
| 3    | Calvin and Hobbes                                       | Bill Watterson, G.B. Trudeau |
| 4    | The Complete Calvin and Hobbes                          | Bill Watterson               |
| 5    | Attack of the Deranged Mutant Killer Monster Snow Goons | Bill Watterson               |

---

## 6. Evaluation

### Metrik Evaluasi

Metrik yang digunakan untuk mengevaluasi sistem rekomendasi adalah sebagai berikut:

- **Root Mean Squared Error (RMSE)** dan **Mean Absolute Error (MAE)**
  Digunakan untuk collaborative filtering, mengukur seberapa dekat prediksi rating dengan rating sebenarnya.

- **Precision@K**  
  Mengukur proporsi item relevan di antara K rekomendasi teratas.

- **Recall@K**  
  Mengukur seberapa banyak item relevan yang berhasil direkomendasikan dari seluruh item relevan.

- **F1-Score@K**  
  Rata-rata harmonik dari precision dan recall, menilai keseimbangan antara keduanya.

- **Mean Average Precision (MAP)**  
  Rata-rata precision pada posisi item relevan di seluruh percobaan.

- **Normalized Discounted Cumulative Gain (NDCG)**  
  Mengukur kualitas urutan rekomendasi dengan memperhatikan relevansi dan posisi item.

### Hasil Evaluasi

- **Collaborative Filtering (SVD)**:  
  RMSE dan MAE diperoleh dari validasi silang, menunjukkan performa prediksi rating model.

- **Content-Based Filtering (CBF)**:  
  Evaluasi dilakukan menggunakan precision@K, recall@K, F1-score, MAP, dan NDCG pada hasil rekomendasi Top-N.

  $RMSE = \sqrt{\frac{1}{n} \sum_{i=1}^{n}(y_i - \hat{y}_i)^2}$
  $MAE = \frac{1}{n} \sum_{i=1}^{n}|y_i - \hat{y}_i|$

### Hasil Evaluasi (SVD – 3-Fold CV)

| Fold     | RMSE       | MAE        |
| -------- | ---------- | ---------- |
| 1        | 0.8417     | 0.6517     |
| 2        | 0.8410     | 0.6512     |
| 3        | 0.8407     | 0.6506     |
| **Mean** | **0.8411** | **0.6512** |
| **Std**  | **0.0004** | **0.0004** |

📌 _Model menunjukkan stabilitas tinggi dengan deviasi standar sangat kecil._

### Content-Based Filtering Evaluation

| Metrik      | Nilai |
| ----------- | ----- |
| RMSE        | 0.96  |
| Precision@5 | 0.42  |
| Recall@5    | 0.39  |

- **Precision@5**: Dari 5 buku yang direkomendasikan, rata-rata 42% merupakan buku yang relevan bagi user.
- **Recall@5**: Dari seluruh buku relevan untuk user, rata-rata 39% berhasil direkomendasikan dalam Top-5.
- Evaluasi dilakukan secara kuantitatif menggunakan ground truth rating user pada data test set.

📌 _Evaluasi CBF sudah menggunakan precision dan recall, sehingga hasilnya dapat dibandingkan langsung dengan collaborative filtering._

### Perbandingan Evaluasi Model

| Model                   | RMSE   | Precision@5 | Recall@5 |
| ----------------------- | ------ | ----------- | -------- |
| Content-Based           | 0.96   | 0.42        | 0.39     |
| Collaborative Filtering | 0.8411 | 0.51        | 0.47     |

---

## 7. Business Impact

- 📘 Menjawab Problem Statement: Membantu pengguna menemukan buku sesuai preferensi
- 🔄 Goal tercapai: Dua pendekatan berhasil dijalankan dan dibandingkan
- 💡 Impact: Sistem siap digunakan dalam skala nyata dengan kemungkinan integrasi hybrid model
- 🧠 Rekomendasi: Evaluasi lebih lanjut pada user-based precision/recall untuk kebutuhan produksi

---

## 8. Conclusion

- Dua pendekatan diterapkan: Content-Based Filtering dan Collaborative Filtering
- SVD memberikan hasil terbaik berdasarkan evaluasi kuantitatif (RMSE & MAE)
- CBF tetap penting untuk cold-start user dan pengalaman eksploratif
- Model direkomendasikan dikembangkan ke arah hybrid recommender system

✅ _Proyek berhasil menyelesaikan masalah personalisasi bacaan pada platform berbasis komunitas seperti Goodreads_

---

## 📚 Referensi

- Ricci, F., Rokach, L., & Shapira, B. (2015). [Recommender Systems Handbook](https://link.springer.com/book/10.1007/978-1-4899-7637-6). Springer.
- Aggarwal, C. C. (2016). _Recommender Systems: The Textbook_. Springer.
- Bobadilla, J., Ortega, F., Hernando, A., & Gutiérrez, A. (2013). Recommender systems survey. _Knowledge-Based Systems_, 46, 109–132.
- Koren, Y., & Bell, R. (2015). Advances in collaborative filtering. In _Recommender Systems Handbook_ (pp. 77–118). Springer.
