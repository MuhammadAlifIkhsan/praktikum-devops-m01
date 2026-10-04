# Blameless Postmortem - Insiden Kegagalan Deployment Manual
 
> Dokumen ini tidak mencari pihak yang salah. Fokusnya adalah celah pada sistem kerja.
> Bagian bertanda **[ISI]** harus diganti dengan data hasil pengukuran JOB 2-4 kelompok sendiri.
 
## Ringkasan Insiden
 
Pada simulasi serah-terima aplikasi `app-sentra` (PT Sentra Digital Batam), tim Operations (Ops) tidak berhasil menjalankan aplikasi di lingkungan yang bersih hanya dengan berpegang pada dokumen `HANDOVER.md` dari tim Development (Dev). Aplikasi berjalan normal di mesin Dev, tetapi gagal di mesin Ops. Insiden ini adalah contoh nyata fenomena "it works on my machine" dan Wall of Confusion. Status akhir percobaan manual: **[ISI: Berhasil / Gagal]**.
 
## Kronologi (timeline)
 
| Waktu | Peristiwa |
|---|---|
| **[ISI: hh:mm]** | Dev menyelesaikan aplikasi, memverifikasi di mesin sendiri, dan menyerahkan folder `serah-terima/` (berisi `src/` dan `HANDOVER.md`). |
| **[ISI: hh:mm]** | Ops menerima artefak dan mulai mengikuti instruksi 3 langkah di `HANDOVER.md`. |
| **[ISI: hh:mm]** | Kegagalan pertama: pesan galat **[ISI: mis. ModuleNotFoundError: No module named 'flask']**. |
| **[ISI: hh:mm]** | Ops mencoba memperbaiki secara mandiri; terjadi kegagalan berikutnya: **[ISI: mis. externally-managed-environment / konflik port / versi Python]**. |
| **[ISI: hh:mm]** | Percobaan dihentikan (berhasil, atau batas 15 menit tercapai). |
| **[ISI: hh:mm]** | Otomasi `setup.sh` diterapkan pada JOB 4; aplikasi berhasil berjalan dan lulus health check dengan satu perintah `./setup.sh`. |
 
## Dampak (waktu terbuang, jumlah kegagalan)
 
- Lead Time manual: **[ISI] menit** (JOB 2) dibandingkan **[ISI] menit** setelah otomasi (JOB 4).
- Jumlah kegagalan (failed attempts) pada prosedur manual: **[ISI]**.
- Jumlah pertanyaan yang seharusnya diajukan Ops kepada Dev tetapi tidak dapat disampaikan: **[ISI]**.
- Flow Efficiency (JOB 3): **[ISI]%**; aktivitas dengan %C/A terendah: **[ISI]**.
- Selain waktu, insiden ini menimbulkan ketergantungan pada komunikasi lisan dan menurunkan kepercayaan antar peran.
## Akar Masalah pada SISTEM (bukan pada orang)
 
1. **Pengetahuan lingkungan hanya ada di kepala dan laptop Dev.** Tidak ada mekanisme yang memaksa seluruh langkah penyiapan (virtual environment, dependensi, versi Python) tertulis lengkap dan dapat diuji.
2. **Serah-terima bergantung pada dokumen manual yang tidak divalidasi.** Tidak ada langkah verifikasi bahwa artefak yang diserahkan dapat dijalankan dari kondisi bersih sebelum sampai ke Ops (daftar dependensi `requirements.txt` bahkan tidak ikut terkemas).
3. **Pemisahan peran dengan kanal komunikasi satu arah.** Struktur kerja membuat Ops tidak dapat bertanya cepat, sehingga celah informasi kecil berubah menjadi kegagalan panjang.
4. **Tidak ada umpan balik otomatis.** Kegagalan baru diketahui setelah artefak berpindah tangan, bukan pada saat dibuat.
Dokumen yang tidak sempurna adalah gejala, bukan penyebab: sistem kerja yang sama akan menghasilkan kegagalan serupa meskipun personelnya diganti.
 
## Tindakan Perbaikan (action items) + penanggung jawab peran
 
| No | Tindakan | Penanggung jawab (peran) | Status |
|---|---|---|---|
| 1 | Menyertakan `requirements.txt` dengan versi terkunci pada setiap rilis | Dev | Selesai (JOB 4) |
| 2 | Mengganti instruksi manual dengan skrip `setup.sh` yang idempotent, ber-`set -euo pipefail`, dan memiliki health check | Dev | Selesai (JOB 4) |
| 3 | Menjalankan `./setup.sh` pada lingkungan bersih sebagai syarat sebelum serah-terima | Dev dan Ops | Direncanakan |
| 4 | Menambahkan `.gitignore` sebelum commit agar `.venv` dan berkas rahasia tidak masuk repository | Dev | Selesai (JOB 4) |
| 5 | Mengukur Lead Time dan Change Failure Rate secara rutin sebagai dasar keputusan | Dev dan Ops | Direncanakan |
| 6 | Menetapkan kanal komunikasi dua arah (mis. issue repository) untuk pertanyaan serah-terima | Dev dan Ops | Direncanakan |
 
## Pelajaran yang Diambil
 
- Otomasi menghapus ketergantungan pada ingatan dan kebiasaan individu, sehingga hasil menjadi konsisten dan dapat diulang.
- Perubahan kecil yang diverifikasi otomatis lebih murah diperbaiki daripada kegagalan yang baru ditemukan di tahap hilir.
- Mengukur dengan angka (Lead Time, jumlah kegagalan, Flow Efficiency) membuat perdebatan beralih dari "siapa yang salah" ke "bagian sistem mana yang perlu diperbaiki".
- Kegagalan diperlakukan sebagai bahan pembelajaran tim, sejalan dengan The Third Way dan pilar Culture pada CALMS.


