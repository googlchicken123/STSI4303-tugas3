<template>
  <ion-page>
    <ion-content :fullscreen="true" class="crypto-page">
      <div class="background-blob blob-one"></div>
      <div class="background-blob blob-two"></div>
      
<!-- fungsi fetchCrypto() dipanggil saat tombol Refresh diklik, dan juga saat komponen pertama dimuat (onMounted) -->
      <section class="hero">
        <div class="hero-copy">
          <p class="eyebrow">Tugas 3 • STSI4303 • Reza Rinaldi • 050601026</p>
          <h1>Cryptocurrency Market</h1>
          <p class="hero-text">
            Data cryptocurrency populer dari Coinlore API ditampilkan dalam format yang rapi,
            nyaman dibaca, dan responsive untuk tampilan mobile.
          </p>
        </div>
        <!-- Tombol untuk memperbarui data -->
        <button class="refresh-btn" @click="fetchCrypto" :disabled="loading">
          <span v-if="loading">Loading...</span>
          <span v-else>Refresh</span>
        </button>
      </section>

      <section class="stats-grid">
        <article class="stat-card pastel-a">
          <span class="stat-label">Total data</span>
          <strong class="stat-value">{{ cryptoData.length }}</strong>
        </article>

        <article class="stat-card pastel-b">
          <span class="stat-label">Status</span>
          <strong class="stat-value">{{ loading ? 'Memuat' : 'Siap' }}</strong>
        </article>

        <article class="stat-card pastel-c">
          <span class="stat-label">Sumber data</span>
          <strong class="stat-value">Coinlore</strong>
        </article>
      </section>
      <!-- Kartu untuk menampilkan data cryptocurrency, dengan kondisi untuk loading, error, atau data yang berhasil dimuat -->
      <ion-card class="data-card">
        <ion-card-header class="card-header">
          <ion-card-title>Daftar Cryptocurrency</ion-card-title>
          <p class="card-subtitle">
            Menampilkan field <strong>rank</strong>, <strong>name</strong>, <strong>symbol</strong>,
            dan <strong>price_usd</strong>.
          </p>
        </ion-card-header>
        <!-- Menampilkan spinner saat loading, pesan error jika terjadi kesalahan, atau tabel data jika berhasil -->
        <ion-card-content>
          <div v-if="loading" class="state-box">
            <ion-spinner name="crescent"></ion-spinner>
            <p>Memuat data cryptocurrency...</p>
          </div>

          <div v-else-if="error" class="state-box error">
            <p>{{ error }}</p>
          </div>

          <div v-else>
            <div class="table-head">
              <span>Rank</span>
              <span>Name</span>
              <span>Symbol</span>
              <span>Price USD</span>
            </div>
            <!-- Menampilkan daftar cryptocurrency dalam format yang rapi dan responsif -->
            <div class="crypto-list">
              <article
                v-for="item in cryptoData"
                :key="item.id"
                class="crypto-row"
              >
                <div class="col rank-col">
                  <span class="rank-badge">#{{ item.rank }}</span>
                </div>

                <div class="col name-col">
                  <span class="crypto-name">{{ item.name }}</span>
                </div>

                <div class="col symbol-col">
                  <span class="symbol-pill">{{ item.symbol }}</span>
                </div>

                <div class="col price-col">
                  <span class="price-text">${{ formatPrice(item.price_usd) }}</span>
                </div>
              </article>
            </div>
          </div>
        </ion-card-content>
      </ion-card>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';
import {
  IonPage,
  IonContent,
  IonCard,
  IonCardHeader,
  IonCardTitle,
  IonCardContent,
  IonSpinner,
} from '@ionic/vue';
// Mendefinisikan tipe data untuk item cryptocurrency dan respons API
interface CryptoItem {
  id: string;
  rank: string;
  name: string;
  symbol: string;
  price_usd: string;
}

interface CoinloreResponse {
  data?: CryptoItem[];
}
// Mendefinisikan state untuk menyimpan data cryptocurrency, status loading, dan pesan error
const cryptoData = ref<CryptoItem[]>([]);
const loading = ref(true);
const error = ref('');

const API_URL = 'https://api.coinlore.net/api/tickers/';
// Fungsi untuk mengambil data cryptocurrency dari API, dengan penanganan error dan status loading
const fetchCrypto = async () => {
  try {
    loading.value = true;
    error.value = '';

    const response = await fetch(API_URL);

    if (!response.ok) {
      throw new Error(`Gagal mengambil data. Status: ${response.status}`);
    }

    const data: CoinloreResponse = await response.json();

    if (!data.data || !Array.isArray(data.data)) {
      throw new Error('Struktur data API tidak sesuai.');
    }

    cryptoData.value = data.data.slice(0, 10);
  } catch (err: unknown) {
    error.value =
      err instanceof Error ? err.message : 'Terjadi kesalahan saat memuat data.';
  } finally {
    loading.value = false;
  }
};
//  Fungsi untuk memformat harga cryptocurrency agar lebih mudah dibaca, dengan penanganan untuk nilai yang tidak valid
const formatPrice = (value: string) => {
  const num = Number(value);
  if (Number.isNaN(num)) return value;

  return new Intl.NumberFormat('en-US', {
    minimumFractionDigits: 2,
    maximumFractionDigits: 6,
  }).format(num);
};

onMounted(() => {
  fetchCrypto();
});
</script>

<style scoped>
/* Fungsi untuk memberikan gaya pada halaman, termasuk latar belakang, tata letak, dan responsivitas */
.crypto-page {
  --background: linear-gradient(180deg, #f8fbff 0%, #eef6ff 34%, #ffffff 100%);
  --color: #243047;
}

.crypto-page::part(background) {
  background: linear-gradient(180deg, #f8fbff 0%, #eef6ff 34%, #ffffff 100%);
}
/* Blob latar belakang untuk efek visual yang menarik */
.background-blob {
  position: fixed;
  border-radius: 999px;
  filter: blur(20px);
  opacity: 0.35;
  pointer-events: none;
  z-index: 0;
}

.blob-one {
  width: 260px;
  height: 260px;
  top: -90px;
  right: -90px;
  background: rgba(127, 200, 248, 0.65);
}

.blob-two {
  width: 320px;
  height: 320px;
  bottom: -120px;
  left: -120px;
  background: rgba(147, 129, 255, 0.25);
}
/* Bagian hero untuk judul dan deskripsi, dengan tata letak fleksibel dan responsif */
.hero {
  position: relative;
  z-index: 1;
  padding: 22px 18px 14px;
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 14px;
}

.hero-copy {
  min-width: 0;
}

.eyebrow {
  margin: 0 0 8px;
  font-size: 12px;
  font-weight: 800;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: rgba(36, 48, 71, 0.66);
}

.hero h1 {
  margin: 0;
  font-size: 30px;
  line-height: 1.1;
  font-weight: 900;
  color: #243047;
}

.hero-text {
  margin: 10px 0 0;
  max-width: 720px;
  font-size: 15px;
  line-height: 1.6;
  color: rgba(36, 48, 71, 0.78);
}
/* Gaya untuk tombol refresh, dengan efek hover dan kondisi disabled */
.refresh-btn {
  flex: 0 0 auto;
  border: 0;
  padding: 14px 22px;
  border-radius: 16px;
  background: linear-gradient(135deg, #7fc8f8 0%, #5aa9e6 100%);
  color: #ffffff;
  font-weight: 800;
  box-shadow: 0 14px 30px rgba(36, 48, 71, 0.12);
  cursor: pointer;
  transition: transform 0.15s ease, box-shadow 0.15s ease, opacity 0.15s ease;
}
/* Efek hover untuk tombol refresh */
.refresh-btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 18px 34px rgba(36, 48, 71, 0.16);
}

.refresh-btn:disabled {
  opacity: 0.7;
  cursor: not-allowed;
  transform: none;
}

.stats-grid {
  position: relative;
  z-index: 1;
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 12px;
  padding: 6px 18px 18px;
}
/* Kartu untuk menampilkan statistik, dengan gaya yang konsisten dan latar belakang pastel yang berbeda untuk setiap kartu */
.stat-card {
  padding: 16px 18px;
  border-radius: 22px;
  border: 1px solid rgba(255, 255, 255, 0.78);
  box-shadow: 0 14px 30px rgba(36, 48, 71, 0.08);
  backdrop-filter: blur(12px);
}

.pastel-a {
  background: rgba(127, 200, 248, 0.18);
}

.pastel-b {
  background: rgba(255, 228, 94, 0.22);
}

.pastel-c {
  background: rgba(248, 247, 255, 0.9);
}

.stat-label {
  display: block;
  margin-bottom: 8px;
  font-size: 12px;
  font-weight: 700;
  color: rgba(36, 48, 71, 0.7);
}

.stat-value {
  font-size: 22px;
  font-weight: 900;
  color: #243047;
}
/* Kartu untuk menampilkan data cryptocurrency, dengan kondisi untuk loading, error, atau data yang berhasil dimuat */
.data-card {
  position: relative;
  z-index: 1;
  margin: 0 18px 22px;
  border-radius: 26px;
  background: rgba(255, 255, 255, 0.82);
  border: 1px solid rgba(255, 255, 255, 0.9);
  box-shadow: 0 18px 44px rgba(36, 48, 71, 0.1);
  backdrop-filter: blur(14px);
}

.card-header {
  padding-bottom: 0;
}

ion-card-title {
  color: #243047;
  font-size: 22px;
  font-weight: 900;
}

.card-subtitle {
  margin: 8px 0 0;
  font-size: 14px;
  color: rgba(36, 48, 71, 0.72);
}

.state-box {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  padding: 28px 12px;
  color: rgba(36, 48, 71, 0.75);
  text-align: center;
}

.state-box.error {
  color: #b42318;
  font-weight: 700;
}
/* Gaya untuk header tabel dan baris data cryptocurrency, dengan tata letak grid yang responsif dan estetika yang konsisten */
.table-head {
  display: grid;
  grid-template-columns: 76px 1.2fr 96px 1fr;
  gap: 12px;
  padding: 14px 16px;
  margin-top: 12px;
  border-radius: 16px;
  background: linear-gradient(135deg, rgba(127, 200, 248, 0.18) 0%, rgba(255, 228, 94, 0.16) 100%);
  border: 1px solid rgba(36, 48, 71, 0.08);
  font-size: 13px;
  font-weight: 800;
  color: rgba(36, 48, 71, 0.76);
}

.crypto-list {
  margin-top: 12px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.crypto-row {
  display: grid;
  grid-template-columns: 76px 1.2fr 96px 1fr;
  gap: 12px;
  align-items: center;
  padding: 14px 16px;
  border-radius: 18px;
  border: 1px solid rgba(36, 48, 71, 0.08);
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.96) 0%, rgba(248, 247, 255, 0.82) 100%);
  box-shadow: 0 10px 22px rgba(36, 48, 71, 0.05);
}

.col {
  min-width: 0;
}
/* Gaya untuk badge peringkat, nama cryptocurrency, simbol, dan harga, dengan estetika yang konsisten dan responsif */
.rank-badge {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-width: 40px;
  padding: 7px 10px;
  border-radius: 14px;
  background: linear-gradient(135deg, #7fc8f8 0%, #5aa9e6 100%);
  color: white;
  font-weight: 900;
  box-shadow: 0 10px 18px rgba(90, 169, 230, 0.18);
}

.crypto-name {
  display: block;
  font-size: 15px;
  font-weight: 800;
  color: #243047;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
/* Gaya untuk simbol cryptocurrency dalam bentuk pil, dengan latar belakang pastel dan border yang halus */
.symbol-pill {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-width: 70px;
  padding: 8px 12px;
  border-radius: 16px;
  background: rgba(248, 247, 255, 0.94);
  border: 1px solid rgba(36, 48, 71, 0.08);
  font-weight: 900;
  color: #243047;
}

.price-text {
  display: block;
  font-size: 15px;
  font-weight: 900;
  color: #243047;
  text-align: left;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

@media (max-width: 900px) {
  .hero {
    flex-direction: column;
  }

  .refresh-btn {
    width: 100%;
  }

  .stats-grid {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 640px) {
  .hero {
    padding: 18px 14px 12px;
  }

  .hero h1 {
    font-size: 26px;
  }

  .data-card {
    margin: 0 14px 18px;
  }

  .stats-grid {
    padding: 6px 14px 14px;
  }

  .table-head,
  .crypto-row {
    grid-template-columns: 64px 1fr;
  }

  .table-head span:nth-child(3),
  .table-head span:nth-child(4),
  .crypto-row .symbol-col,
  .crypto-row .price-col {
    grid-column: 1 / -1;
  }

  .price-col {
    margin-top: 2px;
  }
}
</style>