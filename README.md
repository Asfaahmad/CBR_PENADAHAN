#  Case-Based Reasoning (CBR) untuk Analisis Putusan Pengadilan Pidana Penadahan

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![Status](https://img.shields.io/badge/Status-Selesai-success)
![License](https://img.shields.io/badge/License-MIT-green)

Proyek ini merupakan tugas mata kuliah **Penalaran Komputer** yang mengimplementasikan metode **Case-Based Reasoning (CBR)** untuk menganalisis dan memprediksi tingkat hukuman pada putusan pengadilan pidana **penadahan**, menggunakan 50 dokumen putusan resmi sebagai *case base*.

---

##  1. Deskripsi Singkat

Sistem ini dibangun untuk membantu menemukan **kasus putusan pengadilan yang paling mirip** dengan suatu kasus baru (*query*), serta memprediksi kemungkinan **tingkat hukuman** (Ringan, Sedang, atau Berat) berdasarkan kasus-kasus serupa yang pernah diputuskan sebelumnya. Pendekatan ini mengikuti prinsip dasar penalaran berbasis kasus: *"kasus yang mirip cenderung memiliki solusi yang mirip."*

Seluruh data bersumber dari **50 file PDF putusan pidana penadahan** yang diperoleh dari **Direktori Putusan Mahkamah Agung Republik Indonesia**.

---

##  2. Tujuan Proyek

1. Membangun representasi kasus (*case base*) dari dokumen putusan pengadilan yang tidak terstruktur (PDF).
2. Mengimplementasikan mekanisme **Case Retrieval** untuk menemukan kasus paling relevan terhadap suatu kasus baru.
3. Mengimplementasikan mekanisme **Case Solution Reuse** untuk memprediksi hasil putusan berdasarkan kasus-kasus serupa.
4. Mengevaluasi performa sistem CBR menggunakan metrik klasifikasi standar (*Accuracy, Precision, Recall, F1-score*).
5. Menyediakan pipeline yang **reproducible**, terdokumentasi, dan dapat dijalankan ulang secara end-to-end.

---

##  3. Teknologi yang Digunakan

| Kategori | Tools / Library |
|---|---|
| Bahasa Pemrograman | Python 3.10+ |
| Lingkungan Pengembangan | Jupyter Notebook |
| Ekstraksi Teks PDF | `pdfminer.six` |
| Pengolahan Data | `pandas`, `numpy` |
| Machine Learning | `scikit-learn` (TF-IDF, LinearSVC, Cosine Similarity) |
| Visualisasi | `matplotlib` |
| Penyimpanan Model | `joblib` |
| NLP Lanjutan (opsional) | `transformers` |

---

##  4. Struktur Folder Proyek

```text
CBR_PENADAHAN/
│
├── data/
│   ├── raw/
│   │   ├── pdf/                     # 50 file PDF putusan asli
│   │   └── txt/                     # Hasil ekstraksi teks (Notebook 01)
│   ├── processed/
│   │   └── cases.csv                # Case base final (Notebook 02)
│   ├── eval/
│   │   ├── queries.json             # Query uji + ground truth (Notebook 03)
│   │   ├── retrieval_metrics.csv    # Metrik klasifikasi retrieval (Notebook 03)
│   │   └── prediction_metrics.csv   # Metrik evaluasi prediksi (Notebook 05)
│   └── results/
│       └── predictions.csv          # Hasil prediksi Reuse (Notebook 04)
│
├── models/
│   ├── tfidf_vectorizer.pkl         # TF-IDF Vectorizer terlatih (Notebook 03)
│   └── svm_model.pkl                # Model LinearSVC terlatih (Notebook 03)
│
├── logs/
│   └── cleaning.log                 # Log proses cleaning teks (Notebook 01)
│
├── notebooks/
│   ├── 01_case_base.ipynb
│   ├── 02_case_representation.ipynb
│   ├── 03_case_retrieval.ipynb
│   ├── 04_case_solution_reuse.ipynb
│   └── 05_model_evaluation.ipynb
│
├── README.md
└── requirements.txt
```

---

##  5. Penjelasan Setiap Notebook

###  Notebook 01 — Case Base

Tahap pembentukan basis kasus (*case base*) dari dokumen mentah.

- Membaca seluruh file PDF pada `data/raw/pdf/`
- Ekstraksi teks menggunakan `pdfminer.six`
- Pembersihan teks (*cleaning*) dan normalisasi karakter
- Menyimpan hasil ekstraksi sebagai file `.txt`
- Membentuk dataset awal `cases.csv`
- Mencatat seluruh proses pada `cleaning.log`

**Output:** `data/raw/txt/`, `data/processed/cases.csv`, `logs/cleaning.log`

###  Notebook 02 — Case Representation

Tahap representasi kasus secara terstruktur.

- Ekstraksi metadata: `nomor_perkara`, `tanggal_putusan`, `jenis_perkara`, `pasal`, `terdakwa`, `ringkasan_fakta`, `amar_putusan`
- Feature engineering: `text_length`, `word_count`, `unique_words`
- Pelabelan tingkat hukuman (`putusan`: Ringan / Sedang / Berat)

**Output:** `data/processed/cases.csv` (versi final, terdiri dari 50 kasus dengan metadata dan fitur lengkap)

###  Notebook 03 — Case Retrieval

Tahap pencarian kasus yang relevan terhadap suatu *query*.

- Representasi teks menggunakan **TF-IDF Vectorizer**
- Pembagian data **Train-Test Split** (80:20, *stratified*)
- Pelatihan model klasifikasi **LinearSVC**
- Implementasi fungsi `retrieve()` berbasis **Cosine Similarity** untuk mengambil *top-k* kasus paling mirip
- Evaluasi klasifikasi: *Confusion Matrix*, *Classification Report*, *Accuracy*, *Precision*, *Recall*, *F1-score*

**Output:** `models/tfidf_vectorizer.pkl`, `models/svm_model.pkl`, `data/eval/retrieval_metrics.csv`, `data/eval/queries.json`

###  Notebook 04 — Case Solution Reuse

Tahap penggunaan kembali solusi dari kasus-kasus serupa untuk memprediksi kasus baru.

- Memuat ulang model dari Notebook 03 (tanpa pelatihan ulang)
- Mengambil *top-5* kasus mirip menggunakan `retrieve()`
- Memprediksi solusi menggunakan dua metode:
  - **Majority Vote** — label terbanyak dari kasus-kasus mirip
  - **Weighted Similarity** — label dengan total bobot *similarity* tertinggi
- Implementasi fungsi `predict_outcome()` yang menggabungkan kedua metode

**Output:** `data/results/predictions.csv`

###  Notebook 05 — Model Evaluation

Tahap evaluasi akhir terhadap hasil prediksi.

- Menghitung *Accuracy*, *Precision*, *Recall*, *F1-score* untuk metode *Majority Vote* dan *Weighted Similarity*
- Visualisasi *Confusion Matrix* dan perbandingan metrik antar metode
- Analisis kesalahan prediksi (*error analysis*)
- Notebook ini **tidak** melakukan pelatihan ulang, retrieval ulang, maupun prediksi ulang — seluruh evaluasi murni dibaca dari output Notebook 03 dan 04

**Output:** `data/eval/prediction_metrics.csv`

---

##  6. Alur Case-Based Reasoning (CBR)

Sistem ini mengikuti siklus CBR klasik yang diadaptasi menjadi lima tahapan berurutan:

```text
┌────────────────┐    ┌──────────────────────┐    ┌─────────────────┐
│   01. Case      │ →  │   02. Case           │ →  │   03. Case      │
│   Base          │    │   Representation     │    │   Retrieval     │
│ (PDF → Teks)    │    │ (Metadata & Fitur)    │    │ (TF-IDF + SVM)  │
└────────────────┘    └──────────────────────┘    └────────┬────────┘
                                                            │
                                                            ▼
                          ┌─────────────────┐    ┌─────────────────┐
                          │   05. Model      │ ←  │   04. Case      │
                          │   Evaluation     │    │   Solution      │
                          │ (Accuracy, F1)   │    │   Reuse         │
                          └─────────────────┘    └─────────────────┘
```

1. **Retrieve** — Sistem mencari kasus-kasus historis yang paling mirip dengan kasus baru (*query*) menggunakan representasi TF-IDF dan *cosine similarity*.
2. **Reuse** — Solusi (label tingkat hukuman) dari kasus-kasus mirip tersebut digunakan kembali untuk memprediksi solusi kasus baru, melalui *Majority Vote* dan *Weighted Similarity*.
3. **Revise** — *(di luar lingkup tugas ini)* — pada implementasi CBR lengkap, prediksi awal dapat direvisi oleh pakar hukum sebelum difinalisasi.
4. **Retain** — *(di luar lingkup tugas ini)* — kasus baru yang sudah diverifikasi dapat ditambahkan kembali ke *case base* untuk memperkaya basis pengetahuan di masa depan.

Proyek ini berfokus pada implementasi **Retrieve** dan **Reuse** secara penuh, dilengkapi tahap **Evaluation** sebagai validasi performa sistem.

---

##  7. Cara Instalasi

### 7.1 Persyaratan

- Python **3.10** atau lebih baru
- Jupyter Notebook / JupyterLab

### 7.2 Clone Repository

```bash
git clone https://github.com/<username>/CBR_PENADAHAN.git
cd CBR_PENADAHAN
```

### 7.3 Buat Virtual Environment

```bash
python3 -m venv venv

# Aktivasi (Linux/Mac)
source venv/bin/activate

# Aktivasi (Windows)
venv\Scripts\activate
```

### 7.4 Install Dependencies

```bash
pip install -r requirements.txt
```

Isi `requirements.txt`:

```text
pandas>=2.0
numpy>=1.24
scikit-learn>=1.3
joblib>=1.3
matplotlib>=3.7
pdfminer.six>=20221105
jupyter>=1.0
notebook>=7.0
nbformat>=5.9
nbconvert>=7.0
transformers>=4.30   # opsional, untuk eksperimen NLP lanjutan
```

### 7.5 Jalankan Jupyter

```bash
jupyter notebook
```

Lalu buka folder `notebooks/` dari antarmuka Jupyter.

---

##  8. Cara Menjalankan Proyek (Notebook 01 – 05)

>  Notebook **wajib** dijalankan secara berurutan, karena setiap notebook bergantung pada output notebook sebelumnya.

| Langkah | Notebook | Perintah |
|---|---|---|
| 1 | `01_case_base.ipynb` | Jalankan seluruh cell (*Run All*) |
| 2 | `02_case_representation.ipynb` | Jalankan seluruh cell (*Run All*) |
| 3 | `03_case_retrieval.ipynb` | Jalankan seluruh cell (*Run All*) |
| 4 | `04_case_solution_reuse.ipynb` | Jalankan seluruh cell (*Run All*) |
| 5 | `05_model_evaluation.ipynb` | Jalankan seluruh cell (*Run All*) |

Alternatif menjalankan seluruh pipeline melalui terminal tanpa membuka antarmuka Jupyter:

```bash
cd notebooks

jupyter nbconvert --to notebook --execute --inplace 01_case_base.ipynb
jupyter nbconvert --to notebook --execute --inplace 02_case_representation.ipynb
jupyter nbconvert --to notebook --execute --inplace 03_case_retrieval.ipynb
jupyter nbconvert --to notebook --execute --inplace 04_case_solution_reuse.ipynb
jupyter nbconvert --to notebook --execute --inplace 05_model_evaluation.ipynb
```

Seluruh path pada notebook menggunakan variabel `BASE_DIR` yang dihitung relatif terhadap folder `notebooks/`, sehingga pipeline tetap berjalan dengan benar selama struktur folder proyek dipertahankan sesuai bagian 4.

---

##  9. Penjelasan Output yang Dihasilkan

| File | Dihasilkan oleh | Deskripsi |
|---|---|---|
| `data/raw/txt/*.txt` | Notebook 01 | Teks hasil ekstraksi dari setiap PDF putusan |
| `logs/cleaning.log` | Notebook 01 | Log proses pembersihan teks |
| `data/processed/cases.csv` | Notebook 02 | Case base final (50 kasus, metadata + fitur + label) |
| `models/tfidf_vectorizer.pkl` | Notebook 03 | TF-IDF Vectorizer yang sudah dilatih |
| `models/svm_model.pkl` | Notebook 03 | Model klasifikasi LinearSVC yang sudah dilatih |
| `data/eval/queries.json` | Notebook 03 | Kumpulan query uji beserta *ground truth* |
| `data/eval/retrieval_metrics.csv` | Notebook 03 | Metrik evaluasi klasifikasi tahap retrieval |
| `data/results/predictions.csv` | Notebook 04 | Hasil prediksi *Majority Vote* dan *Weighted Similarity* |
| `data/eval/prediction_metrics.csv` | Notebook 05 | Metrik evaluasi akhir terhadap hasil prediksi |

---

##  10. Ringkasan Hasil Evaluasi

Seluruh notebook berhasil dijalankan dari awal hingga akhir **tanpa error**, dengan seluruh output yang disyaratkan berhasil terbentuk.

| Komponen | Model / Metode | Keterangan |
|---|---|---|
| Case Retrieval | TF-IDF + LinearSVC | Digunakan untuk klasifikasi awal dan pencarian kasus mirip |
| Case Solution Reuse | Majority Vote & Weighted Similarity | Dua metode prediksi dibandingkan secara langsung |
| Akurasi Akhir | ± **60%** | Diperoleh dari evaluasi *predictions.csv* pada Notebook 05 |

**Catatan interpretasi:**

- Akurasi sekitar 60% tergolong wajar mengingat jumlah *case base* yang relatif kecil (50 kasus) serta distribusi label yang tidak seimbang (Sedang = 27, Ringan = 16, Berat = 7).
- Kelas **Berat** dengan jumlah sampel paling sedikit cenderung lebih sulit diprediksi secara konsisten dibandingkan kelas **Sedang** dan **Ringan**.
- Metode *Majority Vote* dan *Weighted Similarity* menunjukkan performa yang relatif sebanding pada dataset ini, mengindikasikan bahwa pembobotan berdasarkan *similarity* belum memberikan keunggulan signifikan pada skala data saat ini.

---

##  11. Kesimpulan Proyek

Proyek ini berhasil mengimplementasikan pipeline **Case-Based Reasoning (CBR)** secara lengkap untuk domain putusan pidana penadahan, mulai dari ekstraksi dokumen mentah hingga evaluasi prediksi akhir. Sistem yang dibangun mampu:

1. Mengubah dokumen PDF putusan menjadi representasi kasus yang terstruktur.
2. Menemukan kasus-kasus historis yang paling relevan terhadap suatu kasus baru menggunakan TF-IDF dan *cosine similarity*.
3. Memprediksi tingkat hukuman kasus baru melalui dua strategi *voting* yang berbeda.
4. Mengevaluasi performa sistem secara objektif menggunakan metrik klasifikasi standar.

Hasil evaluasi menunjukkan bahwa pendekatan CBR berbasis TF-IDF dan *voting* sederhana dapat memberikan prediksi yang **cukup informatif**, meskipun akurasi yang dicapai (± 60%) masih memiliki ruang perbaikan signifikan — terutama karena keterbatasan jumlah data dan ketidakseimbangan distribusi label.

---

##  12. Future Work / Pengembangan Selanjutnya

-  **Penambahan jumlah kasus** pada *case base* untuk meningkatkan keandalan statistik model, khususnya pada kelas **Berat** yang masih minim sampel.
-  **Eksplorasi representasi teks berbasis *embedding*** menggunakan model `transformers` (misalnya IndoBERT) sebagai alternatif TF-IDF untuk menangkap konteks semantik secara lebih mendalam.
-  **Penanganan ketidakseimbangan kelas** (*class imbalance*) menggunakan teknik seperti *oversampling*, *undersampling*, atau pembobotan kelas (*class_weight*) pada model klasifikasi.
-  **Implementasi tahap Revise dan Retain** secara penuh, melibatkan validasi pakar hukum terhadap prediksi sistem sebelum kasus baru ditambahkan ke *case base*.
-  **Pengembangan antarmuka pengguna (UI/dashboard)** sederhana agar sistem CBR dapat digunakan secara interaktif oleh praktisi hukum tanpa perlu membuka notebook.
-  **Eksperimen dengan algoritma klasifikasi lain** (misalnya Random Forest, Naive Bayes, atau model berbasis *deep learning*) sebagai pembanding terhadap LinearSVC.

---

##  13. License

Proyek ini dirilis di bawah lisensi **MIT License**.

```text
MIT License

Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

<p align="center">
  <i>Disusun sebagai pemenuhan tugas mata kuliah Penalaran Komputer — Case-Based Reasoning untuk Putusan Pidana Penadahan.</i>
</p>
