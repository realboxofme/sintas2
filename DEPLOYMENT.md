# Panduan Deploy SINTAS

## 🚀 Deploy ke GitHub, Vercel, dan Supabase

### Langkah 1: Upload ke GitHub

#### 1.1 Buat Repository Baru di GitHub
1. Buka https://github.com/new
2. Nama repository: `sintas`
3. Deskripsi: `Sistem Integrasi Administrasi Surat`
4. Pilih **Public** atau **Private**
5. Jangan centang "Add a README file" (karena sudah ada)
6. Klik **Create repository**

#### 1.2 Inisialisasi Git dan Push
```bash
# Di folder project
cd /home/z/my-project

# Inisialisasi git
git init

# Tambahkan semua file
git add .

# Commit pertama
git commit -m "Initial commit: SINTAS - Sistem Integrasi Administrasi Surat"

# Tambahkan remote repository
git remote add origin https://github.com/USERNAME/sintas.git

# Push ke GitHub
git branch -M main
git push -u origin main
```

---

### Langkah 2: Setup Supabase (PostgreSQL Database)

#### 2.1 Buat Project Supabase
1. Buka https://supabase.com dan login/daftar
2. Klik **New Project**
3. Isi form:
   - **Name**: `sintas-db`
   - **Database Password**: Buat password kuat (simpan baik-baik!)
   - **Region**: Pilih terdekat (Singapore untuk Indonesia)
4. Klik **Create new project**
5. Tunggu beberapa menit sampai project siap

#### 2.2 Dapatkan Connection String
1. Di dashboard Supabase, buka **Project Settings** (gear icon)
2. Pilih **Database**
3. Scroll ke bagian **Connection string**
4. Copy connection string **URI** format:
   ```
   postgresql://postgres.[PROJECT-REF]:[PASSWORD]@aws-0-ap-southeast-1.pooler.supabase.com:6543/postgres
   ```

#### 2.3 Dapatkan Password Database
Jika lupa password, bisa di-reset di:
- **Project Settings** > **Database** > **Database Password** > Reset

---

### Langkah 3: Deploy ke Vercel

#### 3.1 Connect GitHub ke Vercel
1. Buka https://vercel.com dan login
2. Klik **Add New** > **Project**
3. Pilih **Import Git Repository**
4. Authorize GitHub jika diminta
5. Pilih repository `sintas`
6. Klik **Import**

#### 3.2 Configure Project
Di halaman **Configure Project**:

1. **Framework Preset**: Next.js (auto-detected)
2. **Root Directory**: `./`
3. **Build Command**: `bun run build`
4. **Output Directory**: `.next`
5. **Install Command**: `bun install`

#### 3.3 Set Environment Variables
Klik **Environment Variables** dan tambahkan:

| Name | Value |
|------|-------|
| `DATABASE_URL` | `postgresql://postgres.[REF]:[PASSWORD]@aws-0-ap-southeast-1.pooler.supabase.com:6543/postgres` |
| `NEXTAUTH_SECRET` | Generate dengan: `openssl rand -base64 32` |
| `NEXTAUTH_URL` | `https://your-app.vercel.app` |

#### 3.4 Deploy
1. Klik **Deploy**
2. Tunggu proses build selesai (2-5 menit)
3. Jika berhasil, akan ada URL seperti: `https://sintas-xyz.vercel.app`

---

### Langkah 4: Setup Database di Supabase

#### 4.1 Push Schema ke Supabase
Setelah deploy berhasil, jalankan Prisma migration:

**Option A: Via Vercel CLI (Recommended)**
```bash
# Install Vercel CLI
npm i -g vercel

# Login
vercel login

# Link project
vercel link

# Pull environment variables
vercel env pull .env.local

# Push schema ke Supabase
bun run db:push
```

**Option B: Via Local dengan DATABASE_URL Supabase**
```bash
# Set DATABASE_URL ke Supabase sementara
export DATABASE_URL="postgresql://postgres.[REF]:[PASSWORD]@..."

# Push schema
bun run db:push

# Seed data awal
npx tsx prisma/seed.ts
npx tsx prisma/seed-templates.ts
npx tsx prisma/seed-sample.ts
```

#### 4.2 Verifikasi di Supabase
1. Buka **Table Editor** di dashboard Supabase
2. Pastikan semua tabel sudah terbuat:
   - User
   - KategoriSurat
   - SuratMasuk
   - SuratKeluar
   - Disposisi
   - Arsip
   - TemplateSurat
   - LogAktivitas

---

### Langkah 5: Verifikasi Deployment

#### 5.1 Test Aplikasi
1. Buka URL Vercel: `https://your-app.vercel.app`
2. Login dengan:
   - Email: `admin@sintas.go.id`
   - Password: `admin123`
3. Test semua fitur

#### 5.2 Jika Ada Error
1. Buka **Logs** di dashboard Vercel
2. Cek error message
3. Perbaiki dan push ulang ke GitHub:
   ```bash
   git add .
   git commit -m "Fix: description"
   git push
   ```
4. Vercel akan auto-deploy

---

## 🔧 Troubleshooting

### Error: Database Connection Failed
- Pastikan `DATABASE_URL` benar di Vercel environment variables
- Cek password database Supabase
- Pastikan IP tidak di-block (Supabase default allow all)

### Error: Prisma Client Not Found
- Tambahkan di `package.json`:
  ```json
  "scripts": {
    "postinstall": "prisma generate"
  }
  ```

### Error: NextAuth Secret
- Generate secret baru: `openssl rand -base64 32`
- Update di Vercel environment variables

---

## 📝 Environment Variables Summary

| Variable | Untuk | Contoh |
|----------|-------|--------|
| `DATABASE_URL` | Supabase PostgreSQL | `postgresql://postgres...` |
| `NEXTAUTH_SECRET` | NextAuth.js session | Random 32+ chars |
| `NEXTAUTH_URL` | Production URL | `https://sintas.vercel.app` |

---

## 🔄 Update & Redeploy

Setiap kali ada perubahan:
```bash
git add .
git commit -m "Update: description"
git push
```

Vercel akan otomatis redeploy dari GitHub.
