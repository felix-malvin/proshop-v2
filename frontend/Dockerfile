# ---------- Stage 1: build aplikasi React ----------
FROM node:22-alpine AS build

WORKDIR /app

# Pengaturan jaringan npm agar tahan terhadap koneksi lambat atau tidak stabil
#   fetch_timeout : batas idle koneksi dinaikkan dari 5 menjadi 10 menit
#   fetch_retries : setiap unduhan yang gagal dicoba ulang sampai 5 kali
#   maxsockets    : jumlah koneksi paralel dikurangi dari 15 menjadi 6
ENV npm_config_fetch_timeout=600000 \
    npm_config_fetch_retries=5 \
    npm_config_fetch_retry_mintimeout=20000 \
    npm_config_fetch_retry_maxtimeout=120000 \
    npm_config_maxsockets=6

# Salin manifest dulu agar layer instalasi dependency bisa di-cache
COPY package.json package-lock.json ./

# Cache mount menyimpan paket yang sudah terunduh di luar image.
# Kalau build gagal di tengah, percobaan berikutnya tidak mengunduh ulang dari nol.
RUN --mount=type=cache,target=/root/.npm npm ci --no-audit --no-fund

# Salin seluruh source frontend lalu build versi produksi
COPY . .
RUN npm run build

# ---------- Stage 2: sajikan hasil build dengan Nginx ----------
FROM nginx:1.30-alpine

# Ganti konfigurasi default Nginx dengan konfigurasi project
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Ambil HANYA folder build dari stage 1. Node.js dan node_modules tidak ikut.
COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 80

# Container dianggap sehat kalau Nginx menyajikan halaman utama
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD wget -qO- http://127.0.0.1/ > /dev/null || exit 1