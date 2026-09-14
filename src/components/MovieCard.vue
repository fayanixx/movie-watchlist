<template>
  <div class="ios-movie-card">
    <div class="card-content">
      <div class="meta-top">
        <span class="genre-pill">
          <ion-icon :icon="pricetagOutline" class="inline-icon"></ion-icon>
          {{ movie.genre }}
        </span>
        <span class="year-pill">
          <ion-icon :icon="calendarOutline" class="inline-icon"></ion-icon>
          {{ movie.year }}
        </span>
      </div>

      <h3 class="movie-title">{{ movie.title }}</h3>

      <div class="rating-row">
        <div class="stars-box">
          <span 
            v-for="s in 5" 
            :key="s" 
            class="star-mini"
            :class="{ 'lit': s <= movie.rating }"
          >★</span>
        </div>
        <span class="rating-score">{{ movie.rating }}.0</span>
      </div>

      <div class="card-footer-ios">
        <button
          class="status-ios-btn"
          :class="{ 'is-done': movie.status === 'Watched' }"
          @click="$emit('toggle', movie)"
        >
          <ion-icon :icon="movie.status === 'Watched' ? checkmarkCircle : timeOutline"></ion-icon>
          <span>{{ movie.status }}</span>
        </button>

        <div class="action-icons-row">
          <button class="ios-icon-action edit" @click="$emit('edit', movie)" title="Edit">
            <ion-icon :icon="createOutline"></ion-icon>
          </button>
          <button class="ios-icon-action delete" @click="$emit('delete', movie.id)" title="Delete">
            <ion-icon :icon="trashOutline"></ion-icon>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { IonIcon } from '@ionic/vue';
import { 
  checkmarkCircle, timeOutline, createOutline, 
  trashOutline, pricetagOutline, calendarOutline 
} from 'ionicons/icons';

defineProps<{
  movie: {
    id: string; 
    title: string;
    genre: string;
    year: number;
    rating: number;
    status: 'Watched' | 'Not Watched';
  }
}>();

defineEmits(['toggle', 'edit', 'delete']);
</script>

<style scoped>
.ios-movie-card {
  background: rgba(20, 24, 36, 0.75);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.07);
  border-radius: 14px;
  overflow: hidden;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.25);
  transition: transform 0.2s ease, border-color 0.2s ease;
  width: 100%;
}

.ios-movie-card:hover {
  border-color: rgba(248, 113, 113, 0.3);
}

.card-content {
  padding: 12px 14px;
}

.meta-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 6px;
}

.genre-pill, .year-pill {
  display: inline-flex;
  align-items: center;
  gap: 3px;
  font-family: 'Inter', sans-serif;
  font-size: 0.65rem;
  font-weight: 600;
  color: #94a3b8;
  background: rgba(255, 255, 255, 0.04);
  padding: 2px 6px;
  border-radius: 5px;
}

.genre-pill {
  color: #f87171;
  background: rgba(248, 113, 113, 0.1);
}

.inline-icon {
  font-size: 0.7rem;
}

.movie-title {
  font-family: 'Poppins', sans-serif;
  font-size: 0.98rem;
  font-weight: 600;
  color: #ffffff;
  margin: 0 0 6px 0;
  line-height: 1.3;
}

.rating-row {
  display: flex;
  align-items: center;
  gap: 6px;
  margin-bottom: 10px;
}

.stars-box {
  display: flex;
  gap: 1px;
}

.star-mini {
  font-size: 0.75rem;
  color: #334155;
}

.star-mini.lit {
  color: #fbbf24;
}

.rating-score {
  font-family: 'Poppins', sans-serif;
  font-size: 0.7rem;
  font-weight: 600;
  color: #fbbf24;
  background: rgba(251, 191, 36, 0.1);
  padding: 1px 5px;
  border-radius: 4px;
}

.card-footer-ios {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 8px;
  border-top: 1px solid rgba(255, 255, 255, 0.05);
}

.status-ios-btn {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 4px 8px;
  border-radius: 14px;
  font-family: 'Inter', sans-serif;
  font-size: 0.68rem;
  font-weight: 600;
  background: rgba(255, 255, 255, 0.04);
  color: #94a3b8;
  border: 1px solid rgba(255, 255, 255, 0.06);
  cursor: pointer;
}

.status-ios-btn.is-done {
  background: rgba(34, 197, 94, 0.12);
  color: #4ade80;
  border-color: rgba(34, 197, 94, 0.3);
}

.action-icons-row {
  display: flex;
  gap: 4px;
}

.ios-icon-action {
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.06);
  width: 26px;
  height: 26px;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #94a3b8;
  cursor: pointer;
  font-size: 0.8rem;
}

.ios-icon-action.edit:hover {
  color: #38bdf8;
  background: rgba(56, 189, 248, 0.1);
}

.ios-icon-action.delete:hover {
  color: #f87171;
  background: rgba(248, 113, 113, 0.1);
}
</style>