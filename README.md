# 📚 Goodreads Book Recommendation System

## 1. Project Overview
Sistem rekomendasi telah menjadi fondasi penting dalam banyak layanan digital.
Proyek ini bertujuan untuk membangun sistem rekomendasi buku menggunakan dua pendekatan utama:
- Content-based Filtering
- Collaborative Filtering

Dataset yang digunakan adalah Goodreads, salah satu sumber data rating dan metadata buku terbesar.

Relevansi proyek ini sangat tinggi karena kebutuhan akan personalisasi dalam platform buku digital semakin meningkat. Referensi seperti "Recommender Systems Handbook" oleh Ricci et al. serta studi-studi dari ACM dan IEEE menyatakan bahwa pendekatan hybrid (gabungan CBF dan CF) mampu memberikan akurasi yang unggul.

---

## 2. Business Understanding

### Problem Statement
Banyak pengguna platform buku seperti Goodreads mengalami kesulitan dalam menemukan buku baru yang sesuai preferensi mereka.
Dengan jutaan judul yang tersedia, pengalaman menjelajah menjadi kurang efisien tanpa bantuan rekomendasi personal.

### Goals
- Mengembangkan sistem rekomendasi yang mampu menyarankan Top-N buku yang relevan.
- Meningkatkan pengalaman pengguna dengan memberikan rekomendasi yang akurat berdasarkan minat pengguna atau karakteristik buku.

### Solution Approach
1. **Content-Based Filtering**
   - Berdasarkan fitur buku seperti judul dan penulis.
   - Cocok untuk cold-start user.

2. **Collaborative Filtering (SVD)**
   - Berdasarkan rating historis dari pengguna lain.
   - Lebih akurat untuk pengguna aktif yang memiliki riwayat cukup.

---

## 3. Data Understanding

Dataset diambil dari repositori publik [Goodbooks-10k](https://github.com/zygmuntz/goodbooks-10k).

### Jumlah Data
- `books.csv`: 10.000 buku × 23 fitur
- `ratings.csv`: 6 juta+ baris × 3 fitur

### Kondisi Data
- Tidak ditemukan missing value
- Terdapat duplikat pada ratings
- Outlier berupa rating 0 dibersihkan

### Fitur Penting
- `book_id`, `title`, `authors`, `average_rating`
- `user_id`, `book_id`, `rating`

---

## 4. Data Preparation

### Langkah-langkah
- Drop rating 0
- Hapus duplikat
- Filter user aktif (>10 rating)
- Gabungkan judul + penulis sebagai konten

Semua langkah dilakukan berurutan untuk menjamin kestabilan model dan efisiensi proses rekomendasi.

---

## 5. Modeling and Result

### Content-Based Filtering
Menggunakan TF-IDF dan cosine similarity antar judul dan penulis buku.

### Collaborative Filtering
Menggunakan algoritma SVD dari library `surprise`.

### Output
- Menyediakan Top-5 rekomendasi berdasarkan input judul
- Menyediakan Top-5 rekomendasi personal berdasarkan user_id

---

## 6. Evaluation

### Metrik Evaluasi
- RMSE: root mean squared error
- MAE: mean absolute error
- Precision@5 / Recall@5

### Hasil Evaluasi Collaborative Filtering
- **RMSE**: 0.8411 ± 0.0004
- **MAE**: 0.6512 ± 0.0004

### Komparasi

| Model               | RMSE  | Precision@5 | Recall@5 |
|---------------------|-------|-------------|-----------|
| Content-Based       | 0.96  | 0.42        | 0.39      |
| Collaborative (SVD) | 0.84  | 0.51        | 0.47      |

### Formula Evaluasi
- RMSE = sqrt(mean((y_true - y_pred)^2))
- MAE  = mean(|y_true - y_pred|)

### Dampak ke Business
Model berhasil menjawab kebutuhan personalisasi user.
Goals dan solusi sesuai dengan problem statement.
Rekomendasi memperkuat pengalaman pengguna dan meningkatkan engagement.

---

## 7. Conclusion

- Content-Based cocok untuk pengguna baru
- Collaborative lebih akurat untuk pengguna aktif
- Pendekatan hybrid layak dikembangkan selanjutnya

Proyek ini membuktikan bahwa sistem rekomendasi yang akurat dapat dibangun dari dataset publik seperti Goodreads.
