# Panduan Instalasi — Ujian Online (GAS REST API + GitHub Pages)

Aplikasi terdiri dari dua bagian:
- **Backend**: `Kode.gs` → ditempel ke Google Apps Script (berkas terpisah, TIDAK ikut ZIP).
- **Frontend**: `ujian-online.zip` → diekstrak lalu di-push ke GitHub Pages.

> Versi ini memakai **database baru** (dibuat otomatis oleh `setupAppEnvironment()`), bukan spreadsheet lama. Data lama tidak ikut pindah; soal lama bisa dimasukkan lewat **Import Soal** (format kolom sama dengan template lama).

## Yang baru
- **Gambar pada soal**: gambar untuk pertanyaan dan untuk tiap opsi A–D (otomatis diperkecil di browser, disimpan di Google Drive folder `📂 Uploads`).
- **Menu admin baru**: Dashboard, Jadwal Ujian (waktu buka/tutup, KKM, acak soal, edit/hapus), Rekap Nilai (filter, hapus hasil agar peserta bisa mengulang), Analisis Soal, Log Pelanggaran, Pengguna (tambah/edit/reset/import), Pengaturan.
- **Menu peserta baru**: Riwayat Nilai dan Profil & Ganti Password.
- Keamanan: password disimpan sebagai hash + salt, token sesi 12 jam, waktu ujian dihitung di server (menutup/refresh halaman tidak mengulang waktu).

## Versi 4: Gerbang Mata Pelajaran + Kode Akses SEBELUM Login
**Alur siswa:** buka situs → pilih **mata pelajaran** → masukkan **kode akses** (jika ujiannya memakai kode) → login dengan username & password → langsung ke ujian yang dipilih.
- Kode akses menentukan ujiannya: satu mata pelajaran boleh punya beberapa ujian dengan kode berbeda. Mata pelajaran tanpa kode langsung lanjut (jika ada lebih dari satu ujian tanpa kode, siswa memilih salah satu).
- **Akun siswa hanya melihat ujian yang dipilih** di gerbang. Server menolak membuka atau mengumpulkan ujian lain, walau API dipanggil manual. Untuk berpindah mata pelajaran siswa menekan **Pilih mata pelajaran lain** (keluar lalu lewat gerbang lagi).
- **Siswa tidak bisa login tanpa melewati gerbang** (login siswa wajib membawa tiket). Tiket ditandatangani server, berlaku 20 menit, dan tidak bisa dipakai sebagai sesi.
- **Guru/Admin** masuk lewat tautan **Masuk sebagai Guru / Admin** di bawah daftar mata pelajaran (tanpa kode).
- Yang tampil di halaman publik hanya nama mata pelajaran aktif yang punya ujian terbuka. Nama ujian, soal, dan kode tidak terlihat sebelum kode benar. Salah kode 5 kali → terkunci 5 menit di perangkat itu.
- Kode tetap dibuat lewat tombol **Kode** di Jadwal Ujian (disimpan sebagai hash di Script Properties).

**Memperbarui dari versi 3:** tempel `Kode.gs` baru → Deploy → Manage deployments → ✏️ → New version → ganti isi folder frontend dengan ZIP baru (pertahankan `GAS_URL`) → `git add .` / `git commit -m "versi 4"` / `git push` → Ctrl+Shift+R. Tidak perlu menjalankan setup lagi. Siswa yang sedang login dengan sesi lama akan diminta masuk lewat gerbang.

> Pastikan ujian yang ingin dikerjakan: status **Aktif**, sudah ada soalnya, sedang dalam jendela waktu buka–tutup, dan mata pelajarannya aktif.

## Sekolah / Instansi sebagai dropdown (versi 5)
- Setelah memilih mata pelajaran, siswa memilih **sekolah / instansi dari dropdown**. Pilihan dicatat pada **setiap hasil ujian** (kolom `instansi` di sheet `HasilUjian`), tampil di **Rekap Nilai** (kolom + filter "Semua sekolah / instansi"), ekspor Excel, cetak, beranda, dan profil siswa.
- **Mengisi dropdown**: lewat menu admin **Sekolah / Instansi** (tambah banyak nama sekaligus, satu per baris; edit; nonaktifkan; hapus), **atau** otomatis dari siswa: dropdown punya pilihan **"Lainnya — ketik nama sekolah"**. Nama yang diketik siswa (dan lolos gerbang) langsung masuk daftar dan menjadi pilihan dropdown untuk siswa berikutnya. Entri dari siswa diberi label **"Ditambah siswa"** di menu admin, jadi mudah dirapikan/dihapus.
- **Pengaturan di menu Sekolah / Instansi**: *Siswa boleh menambah sekolah baru* = **Ya** (default) atau **Tidak** (hanya pilihan dari daftar; opsi "Lainnya" hilang). Bila daftar kosong dan izin Ya, siswa pertama langsung mengetik nama sekolahnya.
- **Perlindungan dari nama sembarangan**: hanya huruf/angka/spasi/tanda baca umum (3–100 karakter); ejaan dibakukan terhadap daftar (tidak ada duplikat karena beda huruf besar/kecil); nama baru hanya ditambahkan setelah siswa lolos gerbang; maksimal 3 nama baru per perangkat per jam dan 30 per jam secara keseluruhan.
- Nama sekolah dibawa di tiket gerbang bertanda tangan, sehingga tidak bisa diubah siswa setelah lolos gerbang. Mengubah/menghapus nama di daftar **tidak** mengubah hasil lama.
- **Pembaruan dari versi sebelumnya:** tempel `Kode.gs` baru → Deploy **New version** → ganti folder frontend dengan ZIP baru → push → Ctrl+Shift+R. Tidak perlu setup ulang (sheet/kolom baru dibuat otomatis). Sesi siswa yang sedang aktif berakhir, jadi hindari memperbarui di tengah ujian.

## Gerbang sebelum login (versi 4)
Halaman depan sekarang berupa **gerbang** empat langkah untuk siswa:
1. **Pilih mata pelajaran** (kartu berwarna; hanya mata pelajaran yang punya ujian terbuka yang tampil).
2. **Sekolah / Instansi** (dropdown; lihat bagian di atas).
3. **Masukkan kode akses** (dilewati bila ujiannya tidak memakai kode).
4. **Login** dengan username dan password.

Akun siswa kemudian **hanya melihat dan hanya bisa mengerjakan ujian yang dipilih di gerbang** — ujian mata pelajaran lain ditolak oleh server, bukan sekadar disembunyikan. Riwayat nilai milik siswa sendiri tetap bisa dilihat. Untuk pindah mata pelajaran, siswa menekan **Pilih mata pelajaran lain** (keluar lalu masuk lewat gerbang lagi).

- **Kode menentukan ujian**: jika satu mata pelajaran punya beberapa ujian, kode yang dimasukkan menunjukkan ujian mana yang dibuka. Ujian tanpa kode: kosongkan kolom kode; bila ada lebih dari satu ujian tanpa kode siswa diminta memilih.
- **Guru/Admin** masuk lewat tombol **Masuk sebagai Guru / Admin** di halaman depan (tanpa mata pelajaran dan kode).
- **Keamanan**: kode diperiksa di server sebelum login dan hasilnya berupa *tiket* bertanda tangan (berlaku 20 menit, hanya bisa dipakai untuk login siswa). Tebak kode dibatasi: 5 kali salah → perangkat terkunci 5 menit, dan ada batas total per mata pelajaran. Kode tetap disimpan sebagai hash di Script Properties dan tidak pernah dikirim ke browser.
- **Pembaruan dari versi 3**: tempel `Kode.gs` baru → Deploy **New version** → ganti folder frontend dengan ZIP baru (tambah berkas `js/gate.js`) → push → Ctrl+Shift+R. Tidak perlu menjalankan setup lagi. Semua sesi siswa lama berakhir otomatis dan siswa masuk lewat gerbang.

## Fitur versi 3: Mata Pelajaran, Kode Akses, Tampilan Baru
- **Menu Mata Pelajaran (admin)**: tambah/ubah mata pelajaran (nama, ikon, warna). Pilih mata pelajaran tiap ujian di **Jadwal Ujian**. Ujian tanpa mata pelajaran muncul di kartu "Lainnya".
- **Alur siswa (versi 4, lihat bagian "Gerbang sebelum login")**: halaman depan → pilih mata pelajaran → kode akses → login → hanya ujian terpilih yang terlihat.
- **Kode akses ujian**: di Jadwal Ujian klik **Kode** → buat kode acak (6 karakter) atau tentukan sendiri (4–12 huruf/angka). Kode **hanya terlihat saat dibuat** (ditampilkan besar untuk dibacakan/ditulis di papan). Lupa? Buat kode baru; siswa yang sudah masuk tidak terpengaruh.
- **Keamanan kode (Script Properties)**: kode disimpan sebagai salt+hash di Script Properties (`KODE_<ID ujian>`), bukan di Sheets, tidak pernah dikirim ke browser siswa, tidak masuk log. Salah memasukkan 5 kali → terkunci 5 menit per siswa per ujian.
- **Tampilan baru**: tema ceria (indigo + amber, font bulat), ikon SVG, kartu mata pelajaran beranimasi, ruang ujian dengan bar kemajuan, timer yang berubah warna saat waktu menipis, layar hasil dengan cincin skor, bintang, dan konfeti; mode terang/gelap (tombol di pojok kanan atas); mendukung "kurangi gerakan" dan navigasi keyboard.

**Cara memperbarui dari versi 2:**
1. Tempel `Kode.gs` baru di Apps Script.
2. Jalankan **setupAppEnvironment** SEKALI lagi (aman: tidak menghapus data/rahasia; menambah sheet `MataPelajaran`, kolom baru di `JadwalUjian`, dan contoh mata pelajaran SD).
3. Deploy → Manage deployments → ✏️ → New version → Deploy.
4. Ganti isi folder frontend dengan isi ZIP baru (pertahankan `GAS_URL` di `js/config.js`) → `git add .` → `git commit -m "versi 3"` → `git push` → Ctrl+Shift+R.
5. Di menu **Jadwal Ujian**, edit tiap ujian untuk memilih mata pelajarannya; klik **Kode** bila ingin memakai kode akses.

## Peningkatan kecepatan (versi 2)
- **Pindah menu tanpa loading**: semua data menu diambil sekali saat login (satu panggilan `app.boot`), lalu halaman tampil instan dari cache browser dan disegarkan diam-diam di latar belakang.
- **Perubahan langsung terlihat** (hapus soal/ujian/hasil/pengguna, aktif-nonaktif ujian, simpan soal): layar berubah seketika, server menyusul; jika gagal, perubahan dikembalikan dengan pesan.
- **Gambar diunggah sejak dipilih** (bukan saat menekan Simpan), jadi Simpan Soal terasa langsung.
- Filter ujian di Rekap Nilai, Analisis, dan Log berjalan di browser (tanpa menunggu server).
- Pustaka Excel dimuat hanya saat dibutuhkan; halaman peserta jauh lebih ringan.
- Server: cache soal/jadwal/pengguna, satu pembacaan sheet per permintaan.
- **Opsional — kurangi jeda pertama setelah lama tidak dipakai**: di editor Apps Script jalankan fungsi `aktifkanPemanas` sekali (izinkan akses trigger). Matikan dengan `matikanPemanas`.

**Cara memperbarui dari versi sebelumnya:** tempel `Kode.gs` baru (database lama tetap dipakai, tidak perlu setup ulang) → Deploy → Manage deployments → ✏️ → New version. Ganti isi folder frontend dengan isi ZIP baru (pertahankan `GAS_URL` Anda di `js/config.js`), lalu `git add .` → `git commit -m "versi cepat"` → `git push`. Tekan Ctrl+Shift+R di browser.

## A. Backend (Google Apps Script)
1. Buka https://script.google.com → **Proyek baru**.
2. Ganti nama berkas `Code.gs` menjadi `Kode`, hapus isinya, tempel seluruh isi `Kode.gs`. Simpan.
3. Pilih fungsi **setupAppEnvironment** di dropdown → **▶ Run** — **hanya sekali**. Klik *Review permissions* dan izinkan akses Drive & Sheets.
4. Buka **Execution log**. Catat baris:
   - `Username : admin`
   - `Password : ...` ← **hanya tampil sekali**. Segera ganti setelah login pertama.
5. **Deploy → New deployment → Web app** · *Execute as*: **Me** · *Who has access*: **Anyone** → Deploy → **salin URL berakhiran `/exec`**.
6. Setiap kali `Kode.gs` diubah: Deploy → **Manage deployments → ✏️ → Version: New version → Deploy**.

Rahasia otomatis (`SESSION_SECRET`, ID spreadsheet/folder) tersimpan di **Script Properties**, bukan di kode. Jangan menambah editor proyek Apps Script sembarangan dan jangan bagikan spreadsheet database. Lupa password admin: jalankan fungsi `resetPasswordAdmin` dari editor.

## B. Frontend (GitHub Pages)
1. Ekstrak `ujian-online.zip`. Hasilnya folder **`ujian-online`** — **itulah folder tempat `git init` dijalankan**.
2. Buka `js/config.js`, ganti `GANTI_DENGAN_ID_DEPLOYMENT` sehingga `GAS_URL` berisi URL `/exec` dari langkah A.5. Simpan.
3. Buka terminal **di dalam folder `ujian-online`**, lalu pastikan `index.html` terlihat:
   - Windows PowerShell: `dir` · Mac/Linux: `ls`
4. Buat repository **Public** di github.com (jangan centang README/.gitignore/license), lalu:
```
git init
git add .
git commit -m "Upload pertama"
git branch -M main
git remote add origin https://github.com/USERNAME/NAMA-REPO.git
git push -u origin main
```
   Saat diminta password, pakai **Personal Access Token** (bukan password akun); layar tidak menampilkan karakter apa pun saat Anda mengetik/menempel — itu normal.
5. GitHub → repo → **Settings → Pages** → Source: *Deploy from a branch* → `main` / `(root)` → Save. Tunggu 1–2 menit, buka `https://USERNAME.github.io/NAMA-REPO/`.
6. Update berikutnya: `git add .` → `git commit -m "update"` → `git push`.

> Jangan upload lewat tombol "Upload files" di web GitHub — folder `css/` dan `js/` jadi rata dan situs kehilangan tampilan.

## C. Uji coba
1. Login `admin` + password dari log → **Pengaturan** → ganti password.
2. **Jadwal Ujian** → ada ujian contoh (boleh dihapus) atau buat baru.
3. **Bank Soal → + Tambah Soal** → isi pertanyaan, klik **Tambah gambar** untuk soal/opsi, pilih bulatan kunci → Simpan. Klik **Pratinjau** untuk melihat tampilan peserta.
4. **Pengguna** → tambah peserta (atau **Import Peserta** dari Excel) → login sebagai peserta di jendela lain dan kerjakan ujian.

## D. Masalah umum
| Gejala | Solusi |
|---|---|
| "Konfigurasi belum lengkap" | `GAS_URL` di `js/config.js` belum diisi / belum di-push |
| Login gagal "Terjadi kesalahan server… Database belum disiapkan" | `setupAppEnvironment()` belum dijalankan |
| Perubahan `Kode.gs` tidak berefek | Deploy **New version** (langkah A.6) |
| Upload gambar gagal "tidak bisa dibuat dapat dilihat peserta" | Akun Workspace/sekolah membatasi berbagi "siapa saja dengan link"; minta admin domain mengizinkan, atau pakai akun Gmail pribadi untuk menjalankan script |
| Gambar tidak tampil di peserta | Cek file di Drive folder Uploads: berbagi harus "Siapa saja yang memiliki link" |
| Situs 404 | `index.html` tidak di root repo — folder yang di-push salah |
| Situs tanpa tampilan | Struktur `css/` `js/` rusak — push ulang dari terminal |
| Tampil versi lama | Ctrl+Shift+R |
