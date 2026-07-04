# Named Entity Recognition (NER) & Social Network Analysis (SNA) untuk Bahasa Indonesia

Writer
- Davi Rizky Madani – 2411502061@student.budiluhur.ac.id

Co-Writer
- Dr. Indra, S.Kom, M.T.I. – indra@budiluhur.ac.id


Repositori ini berisi *script* Python dalam format Jupyter Notebook (`NER_with_InNER_NetworkOptions.ipynb`) yang dirancang untuk melakukan **ekstraksi entitas teks** (Named Entity Recognition) secara otomatis menggunakan pendekatan *Rule-Based*, sekaligus memformat hasilnya menjadi **grafik/jaringan** yang siap divisualisasikan menggunakan alat Social Network Analysis (SNA) seperti **Gephi**.

## ✨ Fitur Utama
1. **Rule-Based NER**: Mendeteksi 3 jenis entitas utama (`PERSON`, `LOCATION`, `ORGANIZATION`) berdasarkan morfologi kata, *Part-of-Speech* (POS), dan kamus konteks Bahasa Indonesia (seperti awalan "pak", "gubernur", dll).
2. **Generasi Jaringan Entitas (Undirected Graph)**: Memetakan hubungan antar-topik/entitas berdasarkan kemunculannya secara bersamaan (ko-okurensi) dalam satu dokumen.
3. **Generasi Jaringan Penulis (Directed Graph)**: Memetakan hubungan satu arah dari pembuat teks (Author) menuju entitas yang sedang ia bicarakan (Bipartite Graph).
4. **Deteksi Komunitas Otomatis**: Menggunakan algoritma *Louvain Modularity* dari pustaka `networkx` untuk mengelompokkan topik dominan dan menemukan *echo chambers* atau kubu pengguna tanpa memerlukan aplikasi tambahan.
5. **Gephi-Ready Export**: Mengekspor file dalam format CSV yang menggunakan standar kolom baku Gephi (`Source`, `Target`, `Weight`, `Type`, `Id`, `Label`).

---

## 🧠 Detail Aturan NER (Rule-Based Logics)

Pendekatan *Rule-Based* yang digunakan pada script ini mengevaluasi setiap kata/token berdasarkan gabungan **Morfologi** (Bentuk Huruf), **Konteks Sekitar** (Kata awalan), dan **Kamus Statis**. Berikut adalah penjelasan dari setiap aturan (rule) yang diterapkan:

### 1. Aturan Kamus Eksplisit (Direct Matching)
Jika kata cocok dengan nama spesifik yang umum dibicarakan pada dataset, maka langsung ditetapkan tipenya:
* **LOCATION**: `jakarta`, `bandung`, `jateng`, `bogor`, `bekasi`, `depok`, `aceh`, `surabaya`, `ikn`, `nusantara`, `ciliwung`, `jkt`, `jakut`, `bayam`, `angke`.
* **PERSON**: `anies`, `anis`, `jokowi`, `joko`, `ahok`, `heru`, `prabowo`, `gibran`, `pramono`, `rano`, `karno`, `doel`, `budi`.

### 2. Aturan Pola Lokasi (Location Rules)
Jika sebuah kata bukan merupakan bagian dari kamus eksplisit, kata akan ditandai sebagai **LOCATION** jika memenuhi syarat:
* Kata berawalan huruf kapital (`TitleCase`), **DAN**
* Kata tersebut muncul **setelah** preposisi lokasi (`LOPP` seperti: *di, ke, dari, wilayah*) atau awalan lokasi (`LPRE` seperti: *kota, provinsi, jalan*).
* **Aturan Khusus Isu Spesifik (Banjir)**: Jika sebelum atau sesudah kata tersebut terdapat kata "banjir", dan kata tersebut menggunakan huruf kapital di awal, maka kata itu diasumsikan sebagai nama tempat kejadian banjir.

### 3. Aturan Pola Organisasi (Organization Rules)
Sebuah kata akan ditandai sebagai **ORGANIZATION** jika:
* Terdapat di dalam kamus awalan organisasi (`ORGANIZATION_LIST` seperti *PSI, KPU, DPR, dsb*).
* **ATAU**, semua huruf pada kata tersebut kapital (`UpperCase`) **DAN** kata tersebut terdeteksi sebagai `OOV` (Out-of-Vocabulary / singkatan tak lazim dalam pembicaraan bahasa sehari-hari).

### 4. Aturan Pola Tokoh (Person Rules)
Jika sebuah kata belum memiliki label, ia akan ditandai sebagai **PERSON** jika:
* Kata berawalan huruf kapital (`TitleCase`), **DAN**
* Tidak termasuk dalam kata umum jabatan penguasa (seperti *gubernur, presiden* — kecuali bila jabatan ini adalah konteks awalan), **DAN**
* Muncul **setelah** kata sapaan/awalan orang (`PPRE` seperti: *pak, bapak, ibu, prof, dokter, gubernur, pj, presiden*), atau setelah kata sapaan gaul (*bang, mas, si*).

*(Kata-kata yang tidak memenuhi satupun syarat dari pola di atas tidak akan dianggap sebagai entitas).*

---

## 🛠️ Persyaratan (Dependencies)
Pastikan Anda telah menginstal *library* berikut sebelum menjalankan *notebook*:
```bash
pip install pandas networkx
```

## 🚀 Cara Penggunaan

1. **Siapkan Dataset Anda:** 
   Siapkan file teks mentah Anda dan simpan dengan nama `raw_full_data.csv` dalam folder yang sama dengan *notebook* ini. Pastikan file CSV tersebut memiliki setidaknya dua kolom berikut (menggunakan pemisah `;`):
   - `author_name` : Nama pengguna/penulis dokumen (untuk analisis jaringan *Author*).
   - `full_text` : Isi kalimat/dokumen utama.
   
2. **Jalankan Notebook:**
   Buka file `NER_with_InNER_NetworkOptions.ipynb` menggunakan Jupyter Notebook, lalu jalankan semua sel kode dari atas ke bawah (*Run All*).

3. **Periksa Hasil Analisis Otomatis:**
   Pada sel kode paling bawah, program akan mencetak hasil analisis komunitas langsung di layar Anda. 

## 📁 Output File & Kegunaan Jaringan (Directed vs Undirected)

Setelah kode dieksekusi, beberapa file CSV siap pakai akan dibuat. Terdapat dua jenis jaringan (SNA) yang dihasilkan untuk menjawab tujuan analisis yang berbeda:

### 1. Jaringan Topik / Ko-okurensi (`Undirected`)
* **File Output:** `nodes_undirected_entity.csv` & `edges_undirected_entity.csv`
* **Kegunaan Utama:** Memetakan peta narasi dan konteks pembicaraan. Jaringan ini **tidak memiliki arah panah** (*Undirected*) karena hanya menghubungkan dua entitas apabila mereka disebutkan secara bersamaan di dalam satu dokumen/teks. 
* **Tujuan Analisis:** Membantu Anda melihat isu apa yang paling sentral, serta topik/tokoh apa saja yang sering dikait-kaitkan oleh publik (misal: "Jokowi" dan "IKN" sering muncul bersamaan dengan garis relasi yang tebal).

### 2. Jaringan Perilaku Penulis / Echo Chambers (`Directed`)
* **File Output:** `nodes_directed_author.csv` & `edges_directed_author.csv`
* **Kegunaan Utama:** Memetakan siapa membicarakan apa (*Bipartite Graph*). Jaringan ini **memiliki arah panah** (*Directed*) yang menunjuk dari **Akun Penulis (Author) ➔ Entitas (Tokoh/Tempat)**.
* **Tujuan Analisis:** Membantu Anda mendeteksi polarisasi atau *echo chambers*. Melalui algoritma modularity Gephi, Anda bisa melihat apakah ada sekumpulan akun (*cluster*) yang secara masif dan eksklusif hanya menggaungkan satu tokoh politik spesifik. Hal ini sangat berguna untuk mengidentifikasi kelompok loyalis, *influencer*, maupun *buzzer*.

### 3. Data Lengkap Hasil NER
* **File Output:** `hasil_ner_full_network.csv`
* **Kegunaan Utama:** Dataset *raw* milik Anda yang utuh, namun telah ditambahkan dengan satu kolom ekstra yang berisi rangkuman entitas dari setiap baris, untuk keperluan rekapitulasi data non-SNA.

## 🕸️ Visualisasi ke Gephi
1. Buka aplikasi **Gephi**, masuk ke tab *Data Laboratory*.
2. Klik **Import Spreadsheet**, pilih file *nodes* (`nodes_undirected_entity.csv`). Set *Import as: Nodes table*.
3. Ulangi langkah no 2 untuk file *edges* (`edges_undirected_entity.csv`). Set *Import as: Edges table*.
4. Gunakan algoritma *ForceAtlas2* pada tab *Overview* untuk merapikan visualisasi jaringan Anda.
