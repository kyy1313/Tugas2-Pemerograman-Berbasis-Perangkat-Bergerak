<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Prakiraan Suhu Kota Jakarta</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <p v-if="loading">Memuat data cuaca...</p>
      <p v-else-if="error" class="error">{{ error }}</p>

      <ion-list v-else>
        <ion-list-header>
          <ion-label>10 Data Pertama (per jam)</ion-label>
        </ion-list-header>

        <ion-item v-for="item in dataCuaca" :key="item.time">
          <ion-label>
            <p class="label-kecil">Waktu</p>
            <h2 class="waktu">{{ formatWaktu(item.time) }}</h2>
          </ion-label>
          <ion-note slot="end" class="suhu">
            {{ item.temperature_2m }} °C
          </ion-note>
        </ion-item>
      </ion-list>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import axios from 'axios';
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonList,
  IonListHeader,
  IonItem,
  IonLabel,
  IonNote,
} from '@ionic/vue';

interface DataCuaca {
  time: string;
  temperature_2m: number;
}

const API_URL =
  'https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&hourly=temperature_2m';

const dataCuaca = ref<DataCuaca[]>([]);
const loading = ref(true);
const error = ref('');

const formatWaktu = (waktu: string) => waktu.replace('T', ' ');

onMounted(async () => {
  try {
    const response = await axios.get(API_URL);
    const waktu: string[] = response.data.hourly.time;
    const suhu: number[] = response.data.hourly.temperature_2m;

    // Ambil 10 data pertama, lalu gabungkan time dan temperature_2m
    dataCuaca.value = waktu.slice(0, 10).map((t, i) => ({
      time: t,
      temperature_2m: suhu[i],
    }));
  } catch (e) {
    error.value = 'Gagal memuat data cuaca. Periksa koneksi internet Anda.';
  } finally {
    loading.value = false;
  }
});
</script>

<style scoped>
.label-kecil {
  font-size: 12px;
  color: #c4d4f4;
}

.waktu {
  font-size: 17px;
  color: #585e68;
}

.suhu {
  font-size: 17px;
  font-weight: bold;
  color: #263452;
}

.error {
  color: #dc2626;
}
</style>