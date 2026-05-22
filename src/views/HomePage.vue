<!--
  Teknologi yang dipakai:
  - Ionic Vue
  - Vue 3 + TypeScript
  - Open Meteo API
  - CSS grid & Flexbox

  Teknik CSS modern:
  - Responsive design
  - Pastel Color Palette
  - Glassmorphism soft UI
  - Card-Based layout
  - Gradient & box shadow
-->
<template>
  <ion-page>
    <ion-content :fullscreen="true" class="weather-page">
      <div class="background-blob blob-one"></div>
      <div class="background-blob blob-two"></div>
      <!-- Bagian hero section dengan informasi utama tentang cuaca Jakarta -->
      <section class="hero">
        <p class="eyebrow">Tugas 2 • STSI4303 • Pemrograman Berbasis Perangkat Bergerak • Reza Rinaldi • 050601026</p>
        <h1>Cuaca Jakarta</h1>
        <p class="hero-text">
          Data prakiraan cuaca per jam dari Open-Meteo, ditampilkan dalam format rapi dan mudah dibaca.
        </p>

        <div class="hero-badges">
          <div class="badge">
            <span class="badge-label">Total data</span>
            <strong>{{ weatherData.length }} jam</strong>
          </div>
          <div class="badge">
            <span class="badge-label">Suhu terendah</span>
            <strong>{{ minTempText }}°C</strong>
          </div>
          <div class="badge">
            <span class="badge-label">Suhu tertinggi</span>
            <strong>{{ maxTempText }}°C</strong>
          </div>
        </div>
      </section>
      
      <section class="summary-grid">
        <article class="summary-card pastel-a">
          <span>Rata-rata suhu</span>
          <strong>{{ avgTempText }}°C</strong>
        </article>
        <article class="summary-card pastel-b">
          <span>Wilayah</span>
          <strong>Jakarta</strong>
        </article>
        <article class="summary-card pastel-c">
          <span>Sumber data</span>
          <strong>Open-Meteo</strong>
        </article>
      </section>

      <ion-card class="data-card">
        <ion-card-header class="card-header">
          <ion-card-title>Prakiraan Cuaca Per Jam</ion-card-title>
          <p class="card-subtitle">Menampilkan field <strong>Time</strong> dan <strong>Temperature</strong>.</p>
        </ion-card-header>

        <ion-card-content>
          <div v-if="loading" class="state-box">
            <ion-spinner name="crescent"></ion-spinner>
            <p>Memuat data cuaca...</p>
          </div>

          <div v-else-if="error" class="state-box error">
            <p>{{ error }}</p>
          </div>

          <div v-else>
            <div class="table-head">
              <span>No</span>
              <span>Time</span>
              <span>Temperature</span>
            </div>

            <div class="weather-list">
              <article
                v-for="(item, index) in weatherData"
                :key="item.time"
                class="weather-row"
                :class="index % 2 === 0 ? 'row-soft' : 'row-white'"
              >
                <div class="col index-col">{{ index + 1 }}</div>
                <div class="col time-col">{{ formatDateTime(item.time) }}</div>
                <div class="col temp-col">
                  <span class="temp-pill">{{ item.temperature_2m.toFixed(1) }}°C</span>
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
// fungsi untuk mengambil data cuaca dari API Open-Meteo dan menampilkan dalam format yang rapi
import { computed, onMounted, ref } from 'vue';
import {
  IonPage,
  IonContent,
  IonCard,
  IonCardHeader,
  IonCardTitle,
  IonCardContent,
  IonSpinner,
} from '@ionic/vue';

interface HourlyWeatherItem {
  time: string;
  temperature_2m: number;
}

interface WeatherApiResponse {
  hourly?: {
    time: string[];
    temperature_2m: number[];
  };
}
// State untuk menyimpan data cuaca, status loading, dan error message
const weatherData = ref<HourlyWeatherItem[]>([]);
const loading = ref(true);
const error = ref('');

const API_URL =
  'https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&hourly=temperature_2m';

const fetchWeather = async () => {
  try {
    loading.value = true;
    error.value = '';

    const response = await fetch(API_URL);

    if (!response.ok) {
      throw new Error(`Gagal mengambil data. Status: ${response.status}`);
    }

    const data: WeatherApiResponse = await response.json();

    if (!data.hourly?.time?.length || !data.hourly?.temperature_2m?.length) {
      throw new Error('Struktur data API tidak sesuai.');
    }

    weatherData.value = data.hourly.time.map((time, index) => ({
      time,
      temperature_2m: data.hourly!.temperature_2m[index],
    }));
  } catch (err: unknown) {
    error.value =
      err instanceof Error ? err.message : 'Terjadi kesalahan saat memuat data.';
  } finally {
    loading.value = false;
  }
};
// Fungsi untuk memformat tanggal dan waktu menjadi format yang lebih mudah dibaca
const formatDateTime = (dateTime: string) => {
  const date = new Date(dateTime);
  return date.toLocaleString('id-ID', {
    dateStyle: 'medium',
    timeStyle: 'short',
  });
};

const minTempText = computed(() => {
  if (!weatherData.value.length) return '-';
  return Math.min(...weatherData.value.map((item) => item.temperature_2m)).toFixed(1);
});

const maxTempText = computed(() => {
  if (!weatherData.value.length) return '-';
  return Math.max(...weatherData.value.map((item) => item.temperature_2m)).toFixed(1);
});

const avgTempText = computed(() => {
  if (!weatherData.value.length) return '-';
  const total = weatherData.value.reduce((sum, item) => sum + item.temperature_2m, 0);
  return (total / weatherData.value.length).toFixed(1);
});

onMounted(() => {
  fetchWeather();
});
</script>


<style scoped>
/* Warna pastel untuk background utama */
.weather-page {
  --background: linear-gradient(180deg, #f8fbff 0%, #f5f8ff 40%, #fdf7ff 100%);
  --color: #243047;
}

.weather-page::part(background) {
  background: linear-gradient(180deg, #f8fbff 0%, #f5f8ff 40%, #fdf7ff 100%);
}

.background-blob {
  position: fixed;
  border-radius: 999px;
  filter: blur(18px);
  opacity: 0.45;
  pointer-events: none;
  z-index: 0;
}

.blob-one {
  width: 260px;
  height: 260px;
  top: -80px;
  right: -80px;
  background: #c9d8ff;
}

.blob-two {
  width: 320px;
  height: 320px;
  bottom: -120px;
  left: -120px;
  background: #dff6ef;
}

.hero {
  position: relative;
  z-index: 1;
  padding: 24px 18px 10px;
}

.eyebrow {
  margin: 0 0 8px;
  font-size: 12px;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: #6d7a95;
}

.hero h1 {
  margin: 0;
  font-size: 30px;
  font-weight: 800;
  line-height: 1.1;
  color: #243047;
}

.hero-text {
  margin: 10px 0 0;
  max-width: 760px;
  font-size: 15px;
  line-height: 1.6;
  color: #5f6b84;
}

.hero-badges {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 12px;
  margin-top: 18px;
}

.badge {
  padding: 14px 16px;
  border-radius: 18px;
  background: rgba(255, 255, 255, 0.8);
  border: 1px solid rgba(155, 170, 210, 0.22);
  box-shadow: 0 12px 30px rgba(69, 87, 134, 0.08);
  backdrop-filter: blur(10px);
}

.badge-label {
  display: block;
  margin-bottom: 8px;
  font-size: 12px;
  color: #6c7890;
}

.badge strong {
  font-size: 20px;
  color: #243047;
}

.summary-grid {
  position: relative;
  z-index: 1;
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 12px;
  padding: 8px 18px 18px;
}

.summary-card {
  padding: 18px;
  border-radius: 22px;
  box-shadow: 0 14px 34px rgba(69, 87, 134, 0.08);
  border: 1px solid rgba(255, 255, 255, 0.7);
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.summary-card span {
  font-size: 13px;
  color: #65708a;
}

.summary-card strong {
  font-size: 22px;
  color: #243047;
}

/* Warna pastel untuk kartu summary */
.pastel-a {
  background: linear-gradient(135deg, #eef4ff 0%, #ffffff 100%);
}

.pastel-b {
  background: linear-gradient(135deg, #f0fbf7 0%, #ffffff 100%);
}

.pastel-c {
  background: linear-gradient(135deg, #fff4f8 0%, #ffffff 100%);
}

/* Kartu data & warna background utama */
.data-card {
  position: relative;
  z-index: 1;
  margin: 0 18px 22px;
  border-radius: 28px;
  background: rgba(255, 255, 255, 0.88);
  box-shadow: 0 18px 44px rgba(69, 87, 134, 0.12);
  border: 1px solid rgba(255, 255, 255, 0.85);
}
/* Header kartu dengan judul dan subtitle */
.card-header {
  padding-bottom: 0;
}

ion-card-title {
  color: #243047;
  font-size: 22px;
  font-weight: 800;
}

.card-subtitle {
  margin: 8px 0 0;
  font-size: 14px;
  color: #6a768f;
}

.state-box {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  padding: 28px 12px;
  color: #556176;
  text-align: center;
}

.state-box.error {
  color: #b42318;
  font-weight: 700;
}

/* Tabel cuaca & warna per jam */
.table-head {
  display: grid;
  grid-template-columns: 90px 1fr 150px;
  gap: 12px;
  padding: 14px 16px;
  margin-top: 10px;
  border-radius: 16px;
  background: linear-gradient(135deg, #eef4ff 0%, #f7fbff 100%);
  border: 1px solid rgba(155, 170, 210, 0.18);
  font-size: 13px;
  font-weight: 700;
  color: #5e6a84;
}

.weather-list {
  margin-top: 12px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.weather-row {
  display: grid;
  grid-template-columns: 90px 1fr 150px;
  gap: 12px;
  align-items: center;
  padding: 14px 16px;
  border-radius: 18px;
  border: 1px solid rgba(155, 170, 210, 0.16);
}
/* Warna latar belakang bergantian untuk setiap baris cuaca */
.row-soft {
  background: linear-gradient(135deg, #ffffff 0%, #f8fbff 100%);
}

.row-white {
  background: linear-gradient(135deg, #fbfcff 0%, #ffffff 100%);
}

.col {
  color: #344157;
  font-size: 14px;
}

.index-col {
  font-weight: 700;
}

.time-col {
  line-height: 1.4;
}

.temp-col {
  display: flex;
  justify-content: flex-start;
}
/* Warna pastel pill suhu */
.temp-pill {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-width: 78px;
  padding: 8px 12px;
  border-radius: 999px;
  background: linear-gradient(135deg, #6f87ff 0%, #8fa2ff 100%);
  color: white;
  font-weight: 800;
  box-shadow: 0 10px 22px rgba(111, 135, 255, 0.22);
}
/* Responsive design untuk tampilan mobile */
@media (max-width: 900px) {
  .hero-badges,
  .summary-grid {
    grid-template-columns: 1fr;
  }

  .table-head,
  .weather-row {
    grid-template-columns: 64px 1fr;
  }

  .temp-col {
    grid-column: 1 / -1;
    justify-content: flex-start;
    margin-top: 4px;
  }
}

@media (max-width: 520px) {
  .hero {
    padding: 20px 14px 8px;
  }

  .summary-grid {
    padding: 8px 14px 16px;
  }

  .data-card {
    margin: 0 14px 18px;
  }

  .hero h1 {
    font-size: 26px;
  }

  .weather-row {
    padding: 12px 14px;
  }
}
</style>