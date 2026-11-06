# Base image Node.js 22 LTS versi Alpine, sama dengan versi Node di Tahap 1
FROM node:22-alpine

# NODE_ENV=development dipilih karena alasan di server.js (lihat penjelasan Tahap 2)
# PORT 5000 sesuai default di server.js
ENV NODE_ENV=development \
    PORT=5000

# Semua perintah berikutnya dijalankan di /app
WORKDIR /app

# Salin manifest dependency lebih dulu agar layer instalasi bisa di-cache
COPY package.json package-lock.json ./

# Install dependency produksi saja, tanpa nodemon dan concurrently
RUN npm ci --omit=dev && npm cache clean --force

# Salin source code backend (frontend tidak ikut)
COPY backend ./backend

# Hanya folder upload yang perlu bisa ditulis oleh user non-root.
# node_modules dan source code cukup dibaca, jadi tidak perlu chown -R
# (chown -R menyalin ulang seluruh node_modules ke layer baru).
RUN mkdir -p /app/uploads && chown node:node /app/uploads

# Jalankan aplikasi sebagai user node, bukan root
USER node

# Dokumentasi port yang dipakai aplikasi
EXPOSE 5000

# Docker menandai container sehat kalau API produk merespons
HEALTHCHECK --interval=30s --timeout=5s --start-period=20s --retries=3 \
  CMD wget -qO- http://127.0.0.1:5000/api/products > /dev/null || exit 1

# Perintah utama container
CMD ["node", "backend/server.js"]