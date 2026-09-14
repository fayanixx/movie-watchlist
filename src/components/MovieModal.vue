<template>
  <ion-modal :is-open="isOpen" @didDismiss="$emit('close')" class="ios-popup-modal">
    <div class="modal-ios-content">
      <div class="modal-header-ios">
        <div>
          <span class="ios-subtitle">WATCHLIST COLLECTION</span>
          <h2 class="ios-title">{{ isEditing ? 'Edit Movie' : 'Add New Movie' }}</h2>
        </div>
        <button class="ios-close-btn" @click="$emit('close')">
          <ion-icon :icon="closeOutline"></ion-icon>
        </button>
      </div>

      <form @submit.prevent="$emit('save')" class="ios-form-stack">
        <div class="ios-field-group">
          <label><ion-icon :icon="filmOutline" class="label-ico"></ion-icon> Movie Title</label>
          <input
            v-model="formData.title"
            type="text"
            placeholder="e.g. Interstellar"
            required
            class="ios-text-input"
          />
        </div>

        <div class="ios-grid-2">
          <div class="ios-field-group">
            <label><ion-icon :icon="pricetagOutline" class="label-ico"></ion-icon> Genre</label>
            <div class="select-wrapper">
              <select v-model="formData.genre" class="ios-text-input ios-select">
                <option value="Action">Action</option>
                <option value="Sci-Fi">Sci-Fi</option>
                <option value="Drama">Drama</option>
                <option value="Anime">Anime</option>
                <option value="Comedy">Comedy</option>
                <option value="Horror">Horror</option>
                <option value="Romance">Romance</option>
                <option value="Thriller">Thriller</option>
              </select>
            </div>
          </div>

          <div class="ios-field-group">
            <label><ion-icon :icon="calendarOutline" class="label-ico"></ion-icon> Year</label>
            <input
              v-model.number="formData.year"
              type="number"
              min="1920"
              max="2099"
              class="ios-text-input"
              required
            />
          </div>
        </div>

        <div class="ios-field-group">
          <div class="label-row-ios">
            <label><ion-icon :icon="starOutline" class="label-ico"></ion-icon> Rating</label>
            <span class="rating-pill-tag">{{ formData.rating }}.0 Stars</span>
          </div>
          <div class="ios-star-picker">
            <span
              v-for="s in 5"
              :key="s"
              class="picker-star"
              :class="{ 'selected': s <= formData.rating }"
              @click="formData.rating = s"
            >★</span>
          </div>
        </div>

        <div class="ios-field-group">
          <label><ion-icon :icon="checkboxOutline" class="label-ico"></ion-icon> Status</label>
          <div class="ios-status-segmented">
            <button
              type="button"
              class="seg-ios-tab"
              :class="{ 'active': formData.status === 'Not Watched' }"
              @click="formData.status = 'Not Watched'"
            >
              <ion-icon :icon="timeOutline"></ion-icon> Not Watched
            </button>
            <button
              type="button"
              class="seg-ios-tab green"
              :class="{ 'active': formData.status === 'Watched' }"
              @click="formData.status = 'Watched'"
            >
              <ion-icon :icon="checkmarkCircle"></ion-icon> Watched
            </button>
          </div>
        </div>

        <button type="submit" class="ios-primary-action-btn">
          {{ isEditing ? 'Save Changes' : 'Add to Watchlist' }}
        </button>
      </form>
    </div>
  </ion-modal>
</template>

<script setup lang="ts">
import { IonModal, IonIcon } from '@ionic/vue';
import { 
  closeOutline, filmOutline, pricetagOutline, 
  calendarOutline, starOutline, checkboxOutline, 
  timeOutline, checkmarkCircle 
} from 'ionicons/icons';

defineProps<{
  isOpen: boolean;
  isEditing: boolean;
  formData: {
    title: string;
    genre: string;
    year: number;
    rating: number;
    status: 'Watched' | 'Not Watched';
  };
}>();

defineEmits(['close', 'save']);
</script>

<style scoped>
.ios-popup-modal {
  --background: rgba(8, 11, 18, 0.65);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  --width: 92%;
  --max-width: 440px;
  --height: auto;
  --max-height: 90vh;
  --border-radius: 22px;
}

.modal-ios-content {
  padding: 20px 20px 22px 20px;
  background: rgba(14, 18, 28, 0.98);
  backdrop-filter: blur(30px);
  -webkit-backdrop-filter: blur(30px);
  color: #ffffff;
  display: flex;
  flex-direction: column;
  box-sizing: border-box;
  border: 1px solid rgba(255, 255, 255, 0.1);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.6);
  border-radius: 22px;
}

.modal-header-ios {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 14px;
}

.ios-subtitle {
  font-family: 'Inter', sans-serif;
  font-size: 0.62rem;
  letter-spacing: 1.5px;
  font-weight: 700;
  color: #f87171;
}

.ios-title {
  margin: 2px 0 0 0;
  font-family: 'Poppins', sans-serif;
  font-size: 1.2rem;
  font-weight: 700;
  color: #ffffff;
}

.ios-close-btn {
  background: rgba(255, 255, 255, 0.06);
  border: 1px solid rgba(255, 255, 255, 0.1);
  color: #94a3b8;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-size: 1rem;
}

.ios-form-stack {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.ios-field-group {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.ios-field-group label {
  display: flex;
  align-items: center;
  gap: 5px;
  font-family: 'Inter', sans-serif;
  font-size: 0.68rem;
  font-weight: 600;
  text-transform: uppercase;
  color: #94a3b8;
  letter-spacing: 0.4px;
}

.label-ico {
  font-size: 0.8rem;
  color: #f87171;
}

.label-row-ios {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.rating-pill-tag {
  font-family: 'Poppins', sans-serif;
  font-size: 0.72rem;
  font-weight: 600;
  color: #fbbf24;
}

.ios-text-input {
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.09);
  border-radius: 10px;
  padding: 10px 12px;
  color: #ffffff;
  font-family: 'Inter', sans-serif;
  font-size: 0.85rem;
  outline: none;
  width: 100%;
  box-sizing: border-box;
}

.ios-text-input:focus {
  border-color: #f87171;
  background: rgba(255, 255, 255, 0.08);
}

.ios-select {
  appearance: none;
  -webkit-appearance: none;
  -moz-appearance: none;
  background-color: rgba(255, 255, 255, 0.05);
  cursor: pointer;
}

.ios-select option {
  background-color: #121622;
  color: #ffffff;
  padding: 10px;
}

.ios-grid-2 {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}

.ios-star-picker {
  display: flex;
  gap: 8px;
  padding: 6px 12px;
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.06);
  border-radius: 10px;
  align-items: center;
}

.picker-star {
  font-size: 1.3rem;
  color: #334155;
  cursor: pointer;
}

.picker-star.selected {
  color: #fbbf24;
}

.ios-status-segmented {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 6px;
}

.seg-ios-tab {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 5px;
  padding: 8px;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.07);
  color: #94a3b8;
  font-family: 'Inter', sans-serif;
  font-size: 0.74rem;
  font-weight: 600;
  cursor: pointer;
}

.seg-ios-tab.active {
  background: rgba(255, 255, 255, 0.1);
  color: #ffffff;
  border-color: rgba(255, 255, 255, 0.2);
}

.seg-ios-tab.green.active {
  background: rgba(34, 197, 94, 0.15);
  color: #4ade80;
  border-color: rgba(34, 197, 94, 0.3);
}

.ios-primary-action-btn {
  margin-top: 6px;
  width: 100%;
  padding: 11px;
  border-radius: 10px;
  border: none;
  background: linear-gradient(135deg, #ef4444 0%, #dc2626 100%);
  color: #ffffff;
  font-family: 'Poppins', sans-serif;
  font-weight: 600;
  font-size: 0.88rem;
  cursor: pointer;
  box-shadow: 0 4px 14px rgba(239, 68, 68, 0.4);
}
</style>