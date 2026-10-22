# KERIS - Analisis Sentimen User Review

Proyek pertama Grup Riset KERIS FILKOM Universitas Brawijaya untuk mengumpulkan, membersihkan, dan menyiapkan data ulasan pengguna aplikasi dari Google Play Store sebagai dataset analisis sentimen.

## Daftar Isi

- [Deskripsi Proyek](#deskripsi-proyek)
- [Struktur Berkas](#struktur-berkas)
- [Alur Pemrosesan Data](#alur-pemrosesan-data)
- [1. Scraping Data Ulasan](#1-scraping-data-ulasan)
- [2. Data Cleansing](#2-data-cleansing)
- [3. Pelabelan Sentimen](#3-pelabelan-sentimen)
- [Konfigurasi dan Lokasi Data](#konfigurasi-dan-lokasi-data)
- [Dependensi](#dependensi)
- [Catatan Penggunaan](#catatan-penggunaan)
- [Pengembangan Selanjutnya](#pengembangan-selanjutnya)

## Deskripsi Proyek

Proyek ini menyediakan notebook Google Colab untuk:

1. Mengambil data ulasan pengguna dari Google Play Store.
2. Menyimpan hasil scraping dalam format CSV.
3. Menghapus kolom yang tidak diperlukan untuk analisis teks.
4. Memeriksa data kosong dan data duplikat.
5. Melakukan normalisasi teks, penanganan slang, dan penghapusan stopword.
6. Membuat label sentimen berdasarkan skor ulasan.
7. Menyimpan dataset hasil cleansing untuk digunakan pada tahap analisis atau pemodelan berikutnya.

Seluruh proses menggunakan kolom `content` sebagai teks ulasan dan kolom `score` sebagai dasar pelabelan sentimen.

## Struktur Berkas

| Berkas | Keterangan |
|---|---|
| `Scrapping_Data_User_Review.ipynb` | Notebook untuk mengambil ulasan aplikasi dari Google Play Store. Contoh aplikasi yang digunakan adalah Rumah Pendidikan dan Schoology. |
| `Data_Cleansing_Kopken_Review.ipynb` | Notebook untuk membersihkan dan memberi label pada ulasan Kopi Kenangan. |
| `Data_Cleansing__MCD_Review.ipynb` | Notebook untuk membersihkan dan memberi label pada ulasan McDonald's. |
| `README.md` | Dokumentasi proyek dan panduan alur pemrosesan data. |

## Alur Pemrosesan Data

```text
Google Play Store
        |
        v
Scraping ulasan menggunakan google_play_scraper
        |
        v
Penyimpanan data mentah dalam CSV
        |
        v
Pemilihan kolom dan pemeriksaan kualitas data
        |
        v
Penghapusan duplikat
        |
        v
Normalisasi teks dan penanganan slang
        |
        v
Penghapusan stopword
        |
        v
Pelabelan berdasarkan skor ulasan
        |
        v
Dataset bersih siap dianalisis
```

## 1. Scraping Data Ulasan

### Teknologi yang digunakan

Proses scraping menggunakan library `google_play_scraper`, khususnya fungsi `reviews`, dengan konfigurasi bahasa Indonesia dan negara Indonesia.

Parameter utama yang digunakan:

- `lang='id'`: mengambil ulasan dalam bahasa Indonesia.
- `country='id'`: menggunakan wilayah Indonesia.
- `sort=Sort.MOST_RELEVANT`: mengurutkan ulasan berdasarkan relevansi.
- `count`: menentukan jumlah maksimum ulasan yang diminta.

### Tahapan scraping

1. Menginstal library yang diperlukan.
2. Menghubungkan Google Colab dengan Google Drive.
3. Membuat folder penyimpanan secara otomatis menggunakan `os.makedirs(..., exist_ok=True)`.
4. Menentukan `app_id` aplikasi Google Play Store.
5. Mengambil ulasan menggunakan fungsi `reviews`.
6. Mengubah hasil scraping menjadi `pandas.DataFrame`.
7. Menyimpan data ke dalam file CSV.
8. Membaca kembali file CSV untuk memeriksa hasil scraping.

### Aplikasi yang dicontohkan

Notebook scraping memuat contoh konfigurasi berikut:

| Aplikasi | App ID | Target jumlah ulasan | Hasil pada eksekusi notebook |
|---|---|---:|---:|
| Rumah Pendidikan | `id.belajar.app` | 20.000 | 20.000 ulasan |
| Schoology | `com.schoology.app` | 20.000 | 1.585 ulasan |

Jumlah hasil akhir dapat berbeda dari nilai `count` karena bergantung pada ketersediaan ulasan di Google Play Store dan respons layanan scraping.

### Contoh kode scraping

```python
import os
import pandas as pd
from google.colab import drive
from google_play_scraper import Sort, reviews

drive.mount('/content/drive')

folder_path = '/content/drive/MyDrive/KERIS-Coba/Scrap-Rumah Pendidikan'
file_path = os.path.join(folder_path, 'hasilscrap.csv')
os.makedirs(folder_path, exist_ok=True)

app_id = 'id.belajar.app'
num_reviews = 20000

app_reviews, _ = reviews(
    app_id,
    lang='id',
    country='id',
    sort=Sort.MOST_RELEVANT,
    count=num_reviews
)

df = pd.DataFrame(app_reviews)
df.to_csv(file_path, index=False)
print(f'Selesai! Berhasil menyimpan {len(df)} data ulasan ke: {file_path}')
```

### Kolom data mentah

Hasil scraping Google Play Store umumnya memuat kolom berikut:

- `reviewId`: identitas unik ulasan.
- `userName`: nama pengguna.
- `userImage`: URL gambar profil pengguna.
- `content`: isi ulasan.
- `score`: skor ulasan dari 1 sampai 5.
- `thumbsUpCount`: jumlah tanda suka pada ulasan.
- `reviewCreatedVersion`: versi aplikasi saat ulasan dibuat.
- `at`: waktu ulasan dibuat.
- `replyContent`: balasan pengembang, jika tersedia.
- `repliedAt`: waktu balasan pengembang.
- `appVersion`: versi aplikasi terkait ulasan.

## 2. Data Cleansing

Data cleansing dilakukan pada notebook `Data_Cleansing_Kopken_Review.ipynb` dan `Data_Cleansing__MCD_Review.ipynb`. Prosesnya memiliki pola yang sama, tetapi menggunakan file input dan lokasi output yang berbeda.

### 2.1 Membaca data

Dataset dibaca menggunakan `pandas.read_csv`. Pada lingkungan Google Colab, file diambil dari Google Drive. Sebelum menjalankan notebook, sesuaikan `folder_path` dan `file_path` dengan lokasi dataset yang tersedia.

### 2.2 Menghapus kolom yang tidak digunakan

Kolom metadata yang tidak diperlukan untuk analisis sentimen dihapus:

```python
drop_columns = [
    'userName',
    'userImage',
    'reviewCreatedVersion',
    'repliedAt',
    'at'
]

df = df.drop(columns=drop_columns, errors='ignore')
```

Kolom utama yang dipertahankan adalah `reviewId`, `content`, `score`, `thumbsUpCount`, `replyContent`, dan `appVersion`, serta kolom hasil pemrosesan seperti `clean_data` dan `label`.

### 2.3 Memeriksa missing value dan duplikasi

Pemeriksaan dilakukan pada kolom `content` sebagai sumber teks utama:

```python
missing_values = df['content'].isna().sum()
duplicate_values = df.duplicated(subset=['content']).sum()
```

Baris duplikat berdasarkan isi ulasan kemudian dihapus dengan mempertahankan kemunculan pertama:

```python
df = df.drop_duplicates(subset=['content'], keep='first')
```

Berdasarkan hasil eksekusi notebook:

- Dataset McDonald's memiliki 9.629 baris sebelum deduplikasi dan 7.332 baris setelah deduplikasi.
- Dataset Kopi Kenangan memiliki 36.248 baris sebelum deduplikasi dan 24.900 baris setelah deduplikasi.

### 2.4 Normalisasi teks

Normalisasi dilakukan melalui fungsi `normalisasi_teks`. Tahapannya meliputi:

1. Mengubah seluruh teks menjadi huruf kecil.
2. Menghapus URL.
3. Mengurangi karakter berulang, misalnya `baguuuus` menjadi `bagus`.
4. Menghapus angka, tanda baca, emoji, dan karakter nonalfabet.
5. Merapikan spasi berlebih.
6. Mengubah kata slang menjadi bentuk yang lebih baku.
7. Menghapus stopword yang tidak diperlukan.

Contoh implementasi inti:

```python
def normalisasi_teks(text):
    text = str(text).lower()
    text = re.sub(r'http\S+|www\S+', '', text)
    text = re.sub(r'(.)\1{2,}', r'\1', text)
    text = re.sub(r'[^a-z\s]', ' ', text)
    text = re.sub(r'\s+', ' ', text).strip()

    words = text.split()
    words = [slang_dict.get(word, word) for word in words]
    words = [word for word in words if word not in stop_words]
    return ' '.join(words)
```

Hasil normalisasi disimpan pada kolom baru `clean_data` sehingga teks asli pada kolom `content` tetap tersedia untuk keperluan audit dan perbandingan.

### 2.5 Kamus slang

Notebook menggunakan kamus normalisasi untuk mengubah kata tidak baku atau singkatan, misalnya:

| Bentuk awal | Bentuk hasil normalisasi |
|---|---|
| `gk`, `ga`, `gak` | `tidak` |
| `apk`, `app` | `aplikasi` |
| `bgt` | `banget` |
| `klo`, `kalo` | `kalau` |
| `udh`, `udah` | `sudah` |
| `pake` | `pakai` |
| `dapet` | `dapat` |
| `pesen` | `pesan` |
| `mantab`, `mantul` | `mantap` |
| `endul` | `enak` |

Kamus dapat diperluas berdasarkan kosakata yang ditemukan pada dataset tertentu. Normalisasi sebaiknya dilakukan secara hati-hati agar makna sentimen tidak berubah.

### 2.6 Stopword dan kata negasi

Stopword Bahasa Indonesia diperoleh dari `Sastrawi`. Beberapa kata yang berpengaruh terhadap sentimen sengaja dipertahankan, antara lain:

- `tidak`
- `tak`
- `bukan`
- `belum`
- `kurang`
- `jangan`
- `sangat`
- `lain`
- `bisa` pada notebook McDonald's

Kata percakapan seperti `nya`, `sih`, `dong`, `deh`, `ya`, dan `kan` kemudian ditambahkan ke daftar stopword.

### 2.7 Menghapus hasil cleansing yang kosong

Setelah normalisasi, teks yang menjadi kosong diubah menjadi missing value dan dihapus:

```python
df['clean_data'] = df['clean_data'].replace('', pd.NA)
df = df.dropna(subset=['clean_data']).copy()
```

Tahap ini mencegah data tanpa informasi teks ikut digunakan dalam analisis atau pelatihan model.

## 3. Pelabelan Sentimen

Label sentimen dibuat berdasarkan skor ulasan pada kolom `score` dengan aturan berikut:

| Skor | Label |
|---:|---|
| 1-2 | `negatif` |
| 3 | `netral` |
| 4-5 | `positif` |

Implementasinya:

```python
def beri_label(score):
    if score <= 2:
        return 'negatif'
    elif score == 3:
        return 'netral'
    return 'positif'

df['label'] = df['score'].apply(beri_label)
```

Pelabelan ini merupakan pelabelan berbasis rating, bukan hasil anotasi manual terhadap isi teks. Oleh karena itu, ulasan dengan isi yang tidak sepenuhnya sesuai dengan rating tetap akan mengikuti label dari skor yang diberikan pengguna.

### Distribusi label pada hasil notebook

#### McDonald's

- `negatif`: 3.831
- `positif`: 3.062
- `netral`: 318

#### Kopi Kenangan

- `positif`: 23.239
- `negatif`: 1.225
- `netral`: 226

Distribusi tersebut menunjukkan bahwa dataset Kopi Kenangan sangat tidak seimbang dan didominasi oleh ulasan positif. Kondisi ini perlu diperhatikan pada tahap pemodelan karena dapat menyebabkan model lebih condong memprediksi kelas mayoritas.

## Konfigurasi dan Lokasi Data

Notebook menggunakan Google Drive sebagai media penyimpanan. Contoh lokasi yang digunakan:

- Dataset McDonald's: `.../KERIS-Coba/Scrap-Kopken/LOW_Resto_MCD_review.csv`
- Dataset Kopi Kenangan: `.../KERIS-Coba/Scrap-Kopken/HIGH_Resto_Kopken_review.csv`
- Hasil cleansing McDonald's: `.../KERIS-Coba/Scrap-Kopken/cleaned_app_reviews_MCD.csv`
- Hasil cleansing Kopi Kenangan: `.../KERIS-Coba/Scrap-Kopken/cleaned_app_reviews_final.csv`
- Data Rumah Pendidikan: `.../KERIS-Coba/Scrap-Rumah Pendidikan/hasilscrap.csv`
- Data Schoology: `.../KERIS-Coba/Scrap-Schoology/hasilscrap.csv`

Lokasi tersebut merupakan contoh dari lingkungan Google Drive pemilik notebook. Sesuaikan path sebelum menjalankan kode pada akun atau lingkungan lain.

## Dependensi

Library utama yang digunakan:

```bash
pip install google_play_scraper Sastrawi pandas nltk
```

Import penting dalam notebook meliputi:

- `pandas` untuk membaca, mengolah, dan menyimpan dataset.
- `google_play_scraper` untuk mengambil ulasan Google Play Store.
- `Sastrawi` untuk stopword Bahasa Indonesia.
- `nltk` untuk kebutuhan pendukung pemrosesan teks pada notebook Kopi Kenangan.
- `re` untuk operasi regular expression.
- `google.colab.drive` untuk mengakses Google Drive dari Google Colab.

Notebook `Data_Cleansing_Kopken_Review.ipynb` juga mengunduh resource NLTK `stopwords` dan `wordnet`, walaupun proses normalisasi yang ditampilkan terutama menggunakan kamus slang dan stopword dari Sastrawi.

## Cara Menjalankan

1. Buka notebook yang ingin digunakan di Google Colab.
2. Jalankan sel instalasi dependensi.
3. Hubungkan Google Drive ketika diminta.
4. Pastikan file CSV input berada pada lokasi yang sesuai.
5. Sesuaikan `folder_path`, `file_path`, `app_id`, dan nama file output bila diperlukan.
6. Jalankan sel secara berurutan dari atas ke bawah.
7. Periksa hasil pemeriksaan data, distribusi label, dan contoh kolom `clean_data`.
8. Gunakan file CSV hasil cleansing untuk eksplorasi data, visualisasi, atau pelatihan model analisis sentimen.

## Catatan Penggunaan

- Notebook ditujukan untuk Google Colab dan menggunakan Google Drive sebagai sumber serta tujuan file.
- Data hasil scraping dapat berubah apabila proses dijalankan kembali karena ulasan di Google Play Store bersifat dinamis.
- Nilai `count` pada fungsi scraping adalah jumlah yang diminta, bukan jaminan jumlah data yang selalu diperoleh.
- Penghapusan duplikat dilakukan berdasarkan teks pada `content`, sehingga dua ulasan berbeda dengan isi identik hanya menyisakan satu baris.
- Kolom `content` dipertahankan agar hasil normalisasi dapat dibandingkan dengan teks asli.
- Kamus slang perlu ditinjau dan diperbarui secara berkala untuk setiap aplikasi atau domain data.
- Data pribadi seperti nama pengguna dan gambar pengguna tidak digunakan dalam proses analisis dan dihapus dari dataset kerja.
- Sebelum membagikan dataset, lakukan pemeriksaan privasi dan pastikan penggunaan data sesuai dengan ketentuan sumber data serta kebijakan yang berlaku.

## Pengembangan Selanjutnya

Beberapa pengembangan yang dapat dilakukan:

1. Menambahkan konfigurasi scraping agar aplikasi dan lokasi output dapat dipilih melalui parameter.
2. Memisahkan kode scraping dan cleansing menjadi modul Python yang dapat digunakan ulang.
3. Menambahkan pengujian otomatis untuk fungsi normalisasi dan pelabelan.
4. Melakukan anotasi manual atau validasi sampel untuk mengukur kesesuaian label berbasis rating.
5. Menangani ketidakseimbangan kelas menggunakan pembobotan kelas, oversampling, atau teknik lain yang sesuai.
6. Menambahkan stemming Bahasa Indonesia menggunakan Sastrawi bila dibutuhkan oleh model.
7. Menambahkan metrik kualitas dataset, seperti persentase missing value, persentase duplikat, dan jumlah teks kosong setelah cleansing.
8. Membuat dokumentasi versi dataset, tanggal scraping, aplikasi, serta parameter scraping agar eksperimen dapat direproduksi.

## Lisensi dan Kontribusi

Silakan tambahkan informasi lisensi, format kontribusi, dan kontak pemelihara proyek apabila repositori akan digunakan atau dikembangkan secara kolaboratif.
