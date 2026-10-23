<template>
  <ion-page>
    <ion-header>
      <ion-toolbar color="primary">
        <ion-title>Prakiraan Suhu Jakarta</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content>
      <!-- Tarik ke bawah untuk memuat ulang data -->
      <ion-refresher slot="fixed" @ionRefresh="onRefresh($event)">
        <ion-refresher-content />
      </ion-refresher>

      <!-- Status: memuat -->
      <div v-if="loading && !rows.length" class="state">
        <ion-spinner name="crescent" />
        <p>Memuat data cuaca…</p>
      </div>

      <!-- Status: gagal -->
      <div v-else-if="error" class="state">
        <p>{{ error }}</p>
        <ion-button @click="loadWeather">Coba lagi</ion-button>
      </div>

      <!-- Data -->
      <template v-else>
        <p class="note">
          Data per jam dari Open-Meteo (zona waktu GMT). Total {{ rows.length }} data.
        </p>

        <ion-list lines="full">
          <!-- Judul kolom -->
          <ion-item color="light" class="head">
            <ion-label>
              <strong>Waktu (time)</strong>
            </ion-label>
            <ion-label slot="end" class="ion-text-end">
              <strong>Suhu (temperature_2m)</strong>
            </ion-label>
          </ion-item>

          <!-- Baris data -->
          <ion-item v-for="row in rows" :key="row.time">
            <ion-label>
              <h3>{{ row.hour }}</h3>
              <p>{{ row.date }}</p>
            </ion-label>
            <ion-label slot="end" class="ion-text-end temp" :class="tempClass(row.temp)">
              {{ row.temp.toFixed(1) }} °C
            </ion-label>
          </ion-item>
        </ion-list>
      </template>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonList,
  IonItem,
  IonLabel,
  IonSpinner,
  IonButton,
  IonRefresher,
  IonRefresherContent,
} from '@ionic/vue';

interface WeatherRow {
  time: string; // contoh: 2026-10-19T14:00
  date: string; // contoh: 19 Okt 2026
  hour: string; // contoh: 14:00
  temp: number; // dalam °C
}

const API_URL =
  'https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&hourly=temperature_2m';

const BULAN = ['Jan', 'Feb', 'Mar', 'Apr', 'Mei', 'Jun', 'Jul', 'Agu', 'Sep', 'Okt', 'Nov', 'Des'];

const rows = ref<WeatherRow[]>([]);
const loading = ref(false);
const error = ref('');

// Ubah "2026-10-19T14:00" menjadi tanggal dan jam yang mudah dibaca
const formatTime = (iso: string) => {
  const [datePart, hour] = iso.split('T');
  const [y, m, d] = datePart.split('-');
  return { date: `${Number(d)} ${BULAN[Number(m) - 1]} ${y}`, hour };
};

// Warna suhu: makin panas makin oranye/merah
const tempClass = (t: number) => {
  if (t >= 32) return 'hot';
  if (t >= 28) return 'warm';
  return 'cool';
};

const loadWeather = async () => {
  loading.value = true;
  error.value = '';
  try {
    const res = await fetch(API_URL);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    const json = await res.json();

    // API mengembalikan dua array paralel: hourly.time dan hourly.temperature_2m
    const times: string[] = json.hourly.time;
    const temps: number[] = json.hourly.temperature_2m;

    rows.value = times.map((time, i) => ({
      time,
      ...formatTime(time),
      temp: temps[i],
    }));
  } catch (e) {
    error.value = 'Data cuaca tidak bisa dimuat. Periksa koneksi internet, lalu coba lagi.';
    console.error(e);
  } finally {
    loading.value = false;
  }
};

const onRefresh = async (event: CustomEvent) => {
  await loadWeather();
  (event.target as HTMLIonRefresherElement).complete();
};

onMounted(loadWeather);
</script>

<style scoped>
.state {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  padding: 48px 24px;
  text-align: center;
}

.note {
  margin: 0;
  padding: 12px 16px;
  font-size: 13px;
  color: var(--ion-color-medium);
}

.head {
  position: sticky;
  top: 0;
  z-index: 1;
}

.temp {
  font-size: 18px;
  font-weight: 600;
  max-width: 45%;
}

.cool {
  color: var(--ion-color-primary);
}

.warm {
  color: var(--ion-color-warning-shade);
}

.hot {
  color: var(--ion-color-danger);
}
</style>