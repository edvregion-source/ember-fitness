# Ember Fitness

Aplikasi fitness tema gelap merah-putih dengan login dan sinkronisasi data lewat Supabase. Kode aplikasinya ada di `www/index.html`, dibungkus jadi aplikasi Android dengan Capacitor, lalu dibuild otomatis oleh GitHub Actions.

Data latihan selalu tersimpan di perangkat dulu, jadi aplikasi tetap jalan tanpa internet. Setelah masuk ke akun, perubahan dikirim ke Supabase dan ikut ke HP lain yang memakai akun yang sama.

## Isi proyek

| File | Fungsi |
|---|---|
| `www/index.html` | Seluruh aplikasi (tampilan dan logika) |
| `www/config.js` | Diisi otomatis saat build dari secret GitHub |
| `supabase/schema.sql` | Tabel database dan aturan keamanan |
| `assets/` | Ikon dan splash screen aplikasi |
| `debug.keystore` | Kunci tanda tangan debug yang tetap, supaya APK baru bisa menimpa APK lama |
| `.github/workflows/build-apk.yml` | Skrip build otomatis |

Skrip build juga bisa membaca file yang terunggah rata di paling atas repo (tanpa folder `www` dan `assets`).

## 1. Siapkan Supabase (sekali saja)

1. Buat project di supabase.com.
2. Buka **SQL Editor**, tempel isi `supabase/schema.sql`, lalu **Run**.
3. Di **Authentication**, matikan **Confirm email** supaya daftar akun langsung bisa dipakai.
4. Supaya **Lupa kata sandi** berfungsi, buka **Authentication → Email Templates → Reset Password**. Tambahkan satu baris ini di isi email:

   ```
   <p>Kode atur ulang kata sandi: {{ .Token }}</p>
   ```

   Tanpa baris ini, email yang terkirim hanya berisi tautan, dan aplikasi memerlukan kode.
5. Salin **Project URL** dan kunci **Publishable key** (atau anon public). Jangan pakai secret key atau `service_role`.

## 2. Secret di GitHub

Di repo: **Settings → Secrets and variables → Actions → New repository secret**.

| Nama secret | Isi |
|---|---|
| `SUPABASE_URL` | Project URL |
| `SUPABASE_ANON_KEY` | Publishable key (`sb_publishable_...`) atau anon public |

## 3. Build APK

Buka tab **Actions → Build Android APK → Run workflow**. Build juga jalan otomatis setiap push ke `main`. Setelah hijau, unduh **ember-fitness-apk** dari bagian Artifacts.

APK debug memakai kunci tetap dari `debug.keystore`, jadi versi baru bisa dipasang menimpa versi lama dan data di HP tidak hilang. Pemasangan pertama setelah kunci ini diperkenalkan mengharuskan aplikasi lama di-uninstall dulu, karena tanda tangannya berbeda. Data yang sudah masuk akun akan kembali setelah masuk lagi.

## 4. Build rilis bertanda tangan (opsional, untuk Play Store)

Langkah ini hanya perlu kalau kamu mau mempublikasikan aplikasi. Untuk dipakai sendiri, APK debug sudah cukup.

1. Buat kunci rilis sekali saja di komputer yang punya Java (JDK):

   ```
   keytool -genkeypair -v -keystore release.keystore -alias ember -keyalg RSA -keysize 2048 -validity 10000
   ```

2. Ubah file kunci jadi teks base64. Di PowerShell:

   ```
   [Convert]::ToBase64String([IO.File]::ReadAllBytes("release.keystore"))
   ```

3. Buat empat secret di GitHub:

   | Nama secret | Isi |
   |---|---|
   | `ANDROID_KEYSTORE_BASE64` | Hasil base64 tadi |
   | `ANDROID_KEYSTORE_PASSWORD` | Kata sandi keystore |
   | `ANDROID_KEY_ALIAS` | `ember` (atau alias yang kamu pilih) |
   | `ANDROID_KEY_PASSWORD` | Kata sandi kunci |

4. Jalankan build lagi. Akan muncul artifact tambahan **ember-fitness-release** berisi APK dan AAB bertanda tangan. Play Store memakai file AAB.

Simpan `release.keystore` dan kata sandinya di tempat aman dan jangan masukkan ke repo. Kalau hilang, aplikasi di Play Store tidak bisa diperbarui lagi.

## Catatan

- Penggabungan data memakai aturan "yang terakhir diubah menang". Kalau dua HP mengubah data saat sama-sama offline, perubahan yang disimpan paling akhir yang dipakai.
- Font dibungkus ke dalam aplikasi saat build, jadi tampilannya sama saat offline. Kalau pengambilan font gagal, aplikasi memakai font sistem dan build tetap jalan.
- Versi iOS butuh Mac dan akun Apple Developer, jadi belum disertakan.
