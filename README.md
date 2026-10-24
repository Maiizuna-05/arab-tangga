# ArabTangga — Website Pembelajaran Bahasa Arab

ArabTangga adalah website belajar Bahasa Arab berbasis permainan ular tangga, dibuat dengan React + Vite. Website dapat dipublikasikan di GitHub dan Netlify. Supabase bersifat opsional untuk autentikasi dan sinkronisasi progres ant perangkat.

## Fitur
- 10 topik materi Bahasa Arab dengan kosakata berharakat, arti, penjelasan, contoh kalimat, dan text-to-speech browser.
- Bank soal yang diperluas otomatis dari kosakata tiap materi (lebih dari 250 soal, tergantung jumlah kosakata pada data), selain soal konsep yang ditulis khusus. Setiap soal kosakata mempunyai empat opsi dan pembahasan.
- Permainan ular tangga 100 kotak, dadu, ular/tangga, mode satu pemain dan dua pemain bergantian.
- Kuis yang dapat difilter berdasarkan materi, skor dan progres lokal.
- Papan peringkat online melalui Netlify Functions + Netlify Blobs.
- Dashboard progres lokal, aktivitas terbaru, rekap per materi, dan ekspor CSV.
- Akun siswa opsional melalui Supabase Auth; progres tersimpan ke database Supabase. Akun guru dapat melihat ringkasan progres siswa setelah peran guru diatur oleh administrator.
- Desain responsif untuk desktop dan ponsel.

## A. Jalankan di komputer
1. Instal Node.js versi LTS dari https://nodejs.org/.
2. Ekstrak ZIP dan buka terminal pada folder proyek `arab_tangga`.
3. Jalankan:
   ```bash
   npm install
   npm run dev
   ```
4. Buka URL lokal yang ditampilkan Vite (biasanya `http://localhost:5173`).

Untuk mengetes Netlify Functions di komputer:
```bash
npm install -g netlify-cli
netlify dev
```

## B. Unggah repository ke GitHub
1. Masuk ke https://github.com/ dan buat repository bernama `arab-tangga`.
2. Unggah seluruh ISI folder proyek (jangan ZIP saja). Pastikan `package.json`, `netlify.toml`, `src/`, `netlify/`, dan `supabase/` berada di root repository.
3. Commit perubahan. Repository boleh Public atau Private. Jangan unggah file `.env` yang berisi konfigurasi rahasia.

Alternatif dengan Git:
```bash
git init
git add .
git commit -m "Buat website ArabTangga"
git branch -M main
git remote add origin https://github.com/USERNAME/arab-tangga.git
git push -u origin main
```
Ganti `USERNAME` dengan nama akun GitHub.

## C. Deploy ke Netlify
1. Buka https://app.netlify.com/ lalu pilih **Add new project / Import an existing project**.
2. Hubungkan GitHub dan pilih repository `arab-tangga`.
3. Build command: `npm run build`; publish directory: `dist`. Konfigurasi Functions sudah ditetapkan di `netlify.toml`.
4. Klik **Deploy** dan tunggu build selesai.
5. Buka URL `.netlify.app` yang disediakan Netlify.

Setiap kali mengubah kode dan melakukan commit ke GitHub, Netlify akan membangun ulang website secara otomatis jika repository terhubung.

## D. Mengaktifkan akun siswa/guru dan database online (opsional, disarankan untuk penggunaan kelas)
Akun online memerlukan proyek Supabase. Tanpa langkah ini, website tetap berfungsi dalam mode lokal dan papan skor online Netlify; progres lokal tidak otomatis pindah antarperangkat.

1. Buat proyek di https://supabase.com/.
2. Di proyek Supabase, buka **SQL Editor → New query**.
3. Salin seluruh isi `supabase/schema.sql`, jalankan, dan pastikan tidak ada error.
4. Buka **Project Settings → API** (atau bagian Connect), salin Project URL dan anon/public key.
5. Untuk uji lokal, buat file `.env` di root berdasarkan `.env.example`:
   ```env
   VITE_SUPABASE_URL=https://PROJECT-REF.supabase.co
   VITE_SUPABASE_ANON_KEY=ANON_PUBLIC_KEY
   ```
6. Restart server `npm run dev` setelah mengubah `.env`.
7. Untuk website Netlify, buka **Site configuration → Environment variables** dan tambahkan variabel `VITE_SUPABASE_URL` serta `VITE_SUPABASE_ANON_KEY` dengan nilai yang sama. Kemudian lakukan **Trigger deploy → Deploy site**.
8. Buka menu **Akun** di website, lalu daftar sebagai siswa atau masuk. Jika Supabase mengharuskan konfirmasi email, buka email konfirmasi terlebih dahulu.
9. Untuk menjadikan akun sebagai guru, daftar akun dengan email guru, lalu administrator membuka SQL Editor dan menjalankan query berikut dengan email yang sesuai:
   ```sql
   update public.profiles
   set role = 'teacher'
   where id = (select id from auth.users where email = 'guru@sekolah.sch.id');
   ```
   Keluar dan masuk kembali agar peran baru terbaca. Jangan membuka hak pengubahan peran kepada pendaftaran publik.

**Keamanan:** gunakan hanya Project URL dan anon/public key pada frontend. Jangan pernah memasukkan Supabase `service_role` key ke file `.env` frontend, repository publik, atau variabel yang diawali `VITE_`. Kebijakan RLS di `schema.sql` membatasi siswa pada progres miliknya dan mengizinkan guru membaca progres siswa.

## E. Isi materi dan bank soal
Materi tersimpan di `src/content-data.js`, array `LESSONS`. Setiap topik memiliki `id`, judul Arab, subtitle, tingkat, ikon, kosakata, penjelasan, dan contoh kalimat. Bank soal khusus berada pada array `raw`; soal kosakata tambahan dibuat otomatis dari seluruh kosakata dalam `LESSONS`.

Saat menambah topik, pastikan `id` unik dan setiap soal memakai `lessonId` yang cocok. Periksa harakat, terjemahan, dan kunci jawaban dengan guru Bahasa Arab sebelum digunakan secara resmi.

## F. Catatan batasan
- Dashboard lokal dan ekspor CSV hanya menggunakan data browser yang sedang digunakan. Rekap lintas perangkat tersedia setelah Supabase dikonfigurasi dan pengguna masuk.
- Papan skor online adalah leaderboard sederhana. Untuk pemakaian sekolah dalam skala besar, pertimbangkan pembatasan pengiriman skor, moderasi nama, kebijakan privasi, dan persetujuan wali sesuai aturan sekolah.
- Audio pelafalan memakai text-to-speech browser; suara Arab bergantung pada perangkat dan browser.
- Materi awal disiapkan sebagai konten edukasi dan perlu ditinjau kembali sebelum digunakan sebagai bahan ajar resmi.
