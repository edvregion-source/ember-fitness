# Ember Fitness

Aplikasi fitness tema gelap merah-putih dengan login dan sinkronisasi data lewat Supabase. Kode aplikasinya ada di `www/index.html`, lalu dibungkus jadi aplikasi Android dengan Capacitor dan dibuild otomatis oleh GitHub Actions.

Data latihan selalu tersimpan di perangkat dulu, jadi aplikasi tetap jalan tanpa internet. Kalau sudah masuk ke akun, perubahan dikirim ke Supabase dan ikut ke HP lain yang masuk dengan akun yang sama.

## 1. Siapkan Supabase (sekali saja)

1. Buat akun dan project baru di supabase.com.
2. Buka **SQL Editor**, buat query baru, tempel isi `supabase/schema.sql`, lalu **Run**. Ini membuat tabel `user_data` yang hanya bisa dibaca dan diubah pemiliknya.
3. Supaya daftar akun langsung bisa dipakai tanpa konfirmasi email, buka **Authentication**, cari pengaturan provider **Email**, lalu matikan **Confirm email**. Kalau tetap dinyalakan, pengguna harus klik tautan di email dulu sebelum bisa masuk.
4. Buka **Project Settings → API**. Salin **Project URL** dan kunci **anon public**.

Jangan pernah memakai kunci `service_role` di aplikasi ini.

## 2. Pasang rahasia di GitHub

1. Buat repo baru di GitHub, lalu unggah semua isi folder ini (termasuk folder `.github`).
2. Di repo, buka **Settings → Secrets and variables → Actions → New repository secret**, lalu buat dua secret:
   - `SUPABASE_URL` berisi Project URL, misalnya `https://abcd1234.supabase.co`
   - `SUPABASE_ANON_KEY` berisi kunci anon public

## 3. Build APK

1. Buka tab **Actions**, pilih **Build Android APK**, lalu **Run workflow**. Build juga jalan otomatis setiap push ke `main`.
2. Tunggu sekitar 5 menit sampai hijau.
3. Unduh **ember-fitness-apk** dari bagian Artifacts, ekstrak, lalu kirim `app-debug.apk` ke HP Android dan pasang. Izinkan pemasangan dari sumber tidak dikenal.
4. Buka aplikasi, masuk ke **Pengaturan**, lalu daftar atau masuk dengan email dan kata sandi.

## Mencoba di laptop

Isi `www/config.js` dengan URL dan kunci anon, lalu buka `www/index.html` di browser. Jangan commit kunci ke repo publik kalau tidak perlu. Kunci anon memang dirancang untuk dipakai di sisi klien, tapi keamanan datanya bergantung pada aturan RLS di `schema.sql`.

## Catatan

- APK ini bertipe debug, cukup untuk dipakai sendiri. Untuk Play Store perlu versi release yang ditandatangani.
- Penggabungan data memakai aturan "yang terakhir diubah menang". Kalau dua HP mengubah data saat sama-sama offline, perubahan yang disimpan paling akhir yang dipakai.
- Versi iOS butuh Mac dan akun Apple Developer, jadi belum disertakan.
