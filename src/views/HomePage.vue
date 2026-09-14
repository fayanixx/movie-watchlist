<template>
  <ion-page class="ios-app-container">
    <div class="ios-ambient-orb red-orb"></div>
    <div class="ios-ambient-orb blue-orb"></div>

    <transition name="toast-slide">
      <div v-if="toast.show" class="ios-toast-pill" :class="toast.type">
        <ion-icon :icon="toast.icon" class="toast-ico"></ion-icon>
        <span class="toast-txt">{{ toast.message }}</span>
      </div>
    </transition>

    <ion-header class="ion-no-border">
      <ion-toolbar class="ios-glass-toolbar">
        <div class="ios-nav-inner">
          <div class="brand-group-ios" @click="scrollToTop" title="Back to top">
            <ion-icon :icon="listOutline" class="brand-emblem-icon"></ion-icon>
            <h1 class="ios-app-title">
              <span class="flix-red">FLIX</span><span class="list-white">LIST</span>
            </h1>
          </div>
          <button class="ios-text-add-btn" @click="openAddModal">
            + Add
          </button>
        </div>
      </ion-toolbar>
    </ion-header>

    <ion-content ref="contentRef" class="ios-content-view">
      <div class="ios-wrapper">
        <div class="ios-banner-card">
          <div class="banner-top-row">
            <span class="ios-badge-tag">PERSONAL VAULT</span>
            <ion-icon :icon="filmOutline" class="banner-ico"></ion-icon>
          </div>
          <h2 class="banner-heading">My Cinematic Library</h2>
          <p class="banner-subtext">Manage your personal collection, ratings, and streaming queue seamlessly.</p>
        </div>

        <div class="ios-stats-card">
          <div class="stat-cell">
            <span class="stat-val">{{ movies.length }}</span>
            <span class="stat-lbl">Total</span>
          </div>
          <div class="stat-div"></div>
          <div class="stat-cell">
            <span class="stat-val green">{{ watchedCount }}</span>
            <span class="stat-lbl">Watched</span>
          </div>
          <div class="stat-div"></div>
          <div class="stat-cell">
            <span class="stat-val red">{{ movies.length - watchedCount }}</span>
            <span class="stat-lbl">Queue</span>
          </div>
        </div>

        <div class="ios-search-bar">
          <ion-icon :icon="searchOutline" class="ios-s-icon"></ion-icon>
          <input 
            v-model="searchQuery" 
            type="text" 
            placeholder="Search movie title..." 
            class="ios-s-input"
          />
          <button v-if="searchQuery" class="ios-s-clear" @click="searchQuery = ''">✕</button>
        </div>

        <div class="ios-segmented-control">
          <button 
            class="ios-seg-item" 
            :class="{ 'selected': activeFilter === 'all' }" 
            @click="activeFilter = 'all'"
          >All</button>
          <button 
            class="ios-seg-item" 
            :class="{ 'selected': activeFilter === 'not-watched' }" 
            @click="activeFilter = 'not-watched'"
          >Queue</button>
          <button 
            class="ios-seg-item" 
            :class="{ 'selected': activeFilter === 'watched' }" 
            @click="activeFilter = 'watched'"
          >Watched</button>
        </div>

        <div class="ios-genre-chips">
          <button
            v-for="genre in availableGenres"
            :key="genre"
            class="ios-chip"
            :class="{ 'active': selectedGenre === genre }"
            @click="selectedGenre = genre"
          >
            {{ genre }}
          </button>
        </div>

        <div v-if="filteredMovies.length === 0" class="ios-empty-state">
          <div class="ios-empty-icon-wrap">
            <ion-icon :icon="filmOutline"></ion-icon>
          </div>
          <p class="ios-empty-text">No movies found in this view</p>
          <button class="ios-reset-btn" @click="resetFilters">Reset Filters</button>
        </div>

        <div v-else class="ios-movie-grid">
          <MovieCard
            v-for="movie in filteredMovies"
            :key="movie.id"
            :movie="movie"
            @toggle="toggleStatus"
            @edit="openEditModal"
            @delete="confirmDelete"
          />
        </div>
      </div>

      <div v-if="isDeleteAlertOpen" class="ios-alert-backdrop" @click="isDeleteAlertOpen = false">
        <div class="ios-alert-card" @click.stop>
          <div class="ios-alert-icon-box">
            <ion-icon :icon="trashOutline" class="alert-trash-ico"></ion-icon>
          </div>
          <h3 class="alert-title">Remove Title?</h3>
          <p class="alert-desc">
            Are you sure you want to drop <b>"{{ movieToDeleteTitle }}"</b> from your FlixList collection?
          </p>
          <div class="ios-alert-actions">
            <button class="ios-alert-btn cancel" @click="isDeleteAlertOpen = false">Cancel</button>
            <button class="ios-alert-btn confirm" @click="executeDelete">Delete</button>
          </div>
        </div>
      </div>

      <MovieModal
        :is-open="isModalOpen"
        :is-editing="isEditing"
        :form-data="formData"
        @close="isModalOpen = false"
        @save="saveMovie"
      />
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { IonPage, IonHeader, IonToolbar, IonContent, IonIcon } from '@ionic/vue';
import { filmOutline, searchOutline, trashOutline, checkmarkCircle, createOutline, addCircle, listOutline } from 'ionicons/icons';
import { db } from '@/firebase';
import { ref as dbRef, push, update, remove, onValue } from 'firebase/database';
import MovieCard from '@/components/MovieCard.vue';
import MovieModal from '@/components/MovieModal.vue';

interface Movie {
  id: string; 
  title: string;
  genre: string;
  year: number;
  rating: number;
  status: 'Watched' | 'Not Watched';
}

const contentRef = ref<any>(null);
const scrollToTop = () => {
  contentRef.value?.$el.scrollToTop(400);
};

const movies = ref<Movie[]>([]);
const activeFilter = ref<'all' | 'not-watched' | 'watched'>('all');
const searchQuery = ref('');
const selectedGenre = ref('All');
const availableGenres = ['All', 'Action', 'Sci-Fi', 'Anime', 'Drama', 'Horror', 'Comedy'];

const isModalOpen = ref(false);
const isEditing = ref(false);
const currentEditId = ref<string | null>(null);

const isDeleteAlertOpen = ref(false);
const movieToDeleteId = ref<string | null>(null);
const movieToDeleteTitle = ref('');

const formData = ref({
  title: '',
  genre: 'Action',
  year: new Date().getFullYear(),
  rating: 5,
  status: 'Not Watched' as 'Watched' | 'Not Watched'
});

const toast = ref({
  show: false,
  message: '',
  type: 'success',
  icon: checkmarkCircle
});

let toastTimeout: any = null;
const triggerToast = (message: string, type: 'success' | 'delete' | 'edit', icon: any) => {
  if (toastTimeout) clearTimeout(toastTimeout);
  toast.value = {
    show: true,
    message,
    type,
    icon
  };
  toastTimeout = setTimeout(() => {
    toast.value.show = false;
  }, 2600);
};

onMounted(() => {
  const moviesRef = dbRef(db, 'movies');
  onValue(moviesRef, (snapshot) => {
    const data = snapshot.val();
    if (data) {
      movies.value = Object.keys(data).map(key => ({
        id: key,
        ...data[key]
      })).reverse();
    } else {
      movies.value = [];
    }
  });
});

const watchedCount = computed(() => movies.value.filter(m => m.status === 'Watched').length);

const filteredMovies = computed(() => {
  return movies.value.filter(movie => {
    const matchesStatus =
      activeFilter.value === 'all' ||
      (activeFilter.value === 'watched' && movie.status === 'Watched') ||
      (activeFilter.value === 'not-watched' && movie.status === 'Not Watched');

    const matchesSearch = movie.title.toLowerCase().includes(searchQuery.value.toLowerCase());
    const matchesGenre = selectedGenre.value === 'All' || movie.genre === selectedGenre.value;

    return matchesStatus && matchesSearch && matchesGenre;
  });
});

const openAddModal = () => {
  isEditing.value = false;
  currentEditId.value = null;
  formData.value = {
    title: '',
    genre: 'Action',
    year: new Date().getFullYear(),
    rating: 5,
    status: 'Not Watched'
  };
  isModalOpen.value = true;
};

const openEditModal = (movie: Movie) => {
  isEditing.value = true;
  currentEditId.value = movie.id;
  formData.value = {
    title: movie.title,
    genre: movie.genre,
    year: movie.year,
    rating: movie.rating,
    status: movie.status
  };
  isModalOpen.value = true;
};

const saveMovie = async () => {
  if (!formData.value.title.trim()) return;

  if (isEditing.value && currentEditId.value !== null) {
    const movieRef = dbRef(db, `movies/${currentEditId.value}`);
    await update(movieRef, {
      title: formData.value.title,
      genre: formData.value.genre,
      year: formData.value.year,
      rating: formData.value.rating,
      status: formData.value.status
    });
    triggerToast(`"${formData.value.title}" updated`, 'edit', createOutline);
  } else {
    const moviesRef = dbRef(db, 'movies');
    await push(moviesRef, {
      title: formData.value.title,
      genre: formData.value.genre,
      year: formData.value.year,
      rating: formData.value.rating,
      status: formData.value.status
    });
    triggerToast(`"${formData.value.title}" added to list`, 'success', addCircle);
  }

  isModalOpen.value = false;
};

const toggleStatus = async (movie: Movie) => {
  const newStatus = movie.status === 'Watched' ? 'Not Watched' : 'Watched';
  const movieRef = dbRef(db, `movies/${movie.id}`);
  await update(movieRef, { status: newStatus });
  triggerToast(
    newStatus === 'Watched' ? `Marked "${movie.title}" as Watched` : `Moved "${movie.title}" to Queue`,
    'success',
    checkmarkCircle
  );
};

const confirmDelete = (id: string) => {
  const target = movies.value.find(m => m.id === id);
  if (target) {
    movieToDeleteId.value = id;
    movieToDeleteTitle.value = target.title;
    isDeleteAlertOpen.value = true;
  }
};

const executeDelete = async () => {
  if (movieToDeleteId.value !== null) {
    const title = movieToDeleteTitle.value;
    const movieRef = dbRef(db, `movies/${movieToDeleteId.value}`);
    await remove(movieRef);
    triggerToast(`"${title}" deleted`, 'delete', trashOutline);
  }
  isDeleteAlertOpen.value = false;
  movieToDeleteId.value = null;
};

const resetFilters = () => {
  searchQuery.value = '';
  selectedGenre.value = 'All';
  activeFilter.value = 'all';
};
</script>

<style scoped>
.ios-app-container {
  --background: #090b10;
  background: #090b10;
  color: #f1f5f9;
  font-family: 'Inter', sans-serif;
}

.brand-group-ios {
  display: flex;
  align-items: center;
  gap: 10px;
  cursor: pointer;
  user-select: none;
}

/* Mas malaki ang icon (1.9rem) kaysa sa app title text (1.35rem) */
.brand-emblem-icon {
  font-size: 1.9rem;
  color: #ef4444;
  filter: drop-shadow(0 0 10px rgba(239, 68, 68, 0.5));
  transition: transform 0.2s ease;
}

.brand-group-ios:active .brand-emblem-icon {
  transform: scale(0.92);
}

.ios-toast-pill {
  position: fixed;
  top: calc(env(safe-area-inset-top, 0px) + 68px);
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 18px;
  background: rgba(16, 21, 36, 0.94);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 24px;
  box-shadow: 0 16px 36px rgba(0, 0, 0, 0.6);
  z-index: 999999;
  max-width: 90%;
}

.toast-ico {
  font-size: 1.25rem;
}

.toast-txt {
  font-family: 'Inter', sans-serif;
  font-size: 0.88rem;
  font-weight: 600;
  color: #ffffff;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.ios-toast-pill.success .toast-ico { color: #4ade80; }
.ios-toast-pill.edit .toast-ico { color: #38bdf8; }
.ios-toast-pill.delete .toast-ico { color: #f87171; }
.ios-toast-pill.delete { border-color: rgba(248, 113, 113, 0.25); }

.toast-slide-enter-active,
.toast-slide-leave-active {
  transition: all 0.28s cubic-bezier(0.16, 1, 0.3, 1);
}
.toast-slide-enter-from {
  opacity: 0;
  transform: translate(-50%, -18px) scale(0.92);
}
.toast-slide-leave-to {
  opacity: 0;
  transform: translate(-50%, -12px) scale(0.96);
}

.ios-ambient-orb {
  position: fixed;
  border-radius: 50%;
  pointer-events: none;
  z-index: 0;
}

.red-orb {
  top: -40px;
  left: -40px;
  width: 280px;
  height: 280px;
  background: rgba(239, 68, 68, 0.12);
  filter: blur(90px);
}

.blue-orb {
  bottom: 8%;
  right: -40px;
  width: 280px;
  height: 280px;
  background: rgba(56, 189, 248, 0.08);
  filter: blur(100px);
}

.ios-glass-toolbar {
  --background: rgba(13, 17, 26, 0.94);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
  padding: calc(env(safe-area-inset-top, 0px) + 10px) 18px 12px 18px;
}

.ios-nav-inner {
  display: flex;
  justify-content: space-between;
  align-items: center;
  max-width: 900px;
  margin: 0 auto;
}

.ios-app-title {
  margin: 0;
  font-family: 'Poppins', sans-serif;
  font-size: 1.35rem;
  font-weight: 800;
  letter-spacing: 0.5px;
}

.flix-red {
  color: #ef4444;
  text-shadow: 0 0 14px rgba(239, 68, 68, 0.55);
}

.list-white {
  color: #ffffff;
}

.ios-text-add-btn {
  background: rgba(239, 68, 68, 0.18);
  border: 1px solid rgba(239, 68, 68, 0.4);
  color: #f87171;
  padding: 8px 16px;
  border-radius: 18px;
  font-family: 'Inter', sans-serif;
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.ios-text-add-btn:active {
  transform: scale(0.96);
}

.ios-content-view {
  --background: transparent;
  --padding-top: 18px;
  --padding-bottom: calc(env(safe-area-inset-bottom, 0px) + 40px);
  --padding-start: 18px;
  --padding-end: 18px;
  position: relative;
}

.ios-wrapper {
  max-width: 900px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  gap: 16px;
  position: relative;
  z-index: 1;
}

.ios-banner-card {
  background: rgba(20, 25, 38, 0.65);
  border: 1px solid rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  border-radius: 18px;
  padding: 18px 20px;
}

.banner-top-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 6px;
}

.ios-badge-tag {
  font-family: 'Inter', sans-serif;
  font-size: 0.7rem;
  font-weight: 700;
  color: #f87171;
  letter-spacing: 1.3px;
}

.banner-ico {
  font-size: 1.25rem;
  color: #64748b;
}

.banner-heading {
  font-family: 'Poppins', sans-serif;
  font-size: 1.3rem;
  font-weight: 700;
  color: #ffffff;
  margin: 0 0 6px 0;
}

.banner-subtext {
  font-size: 0.85rem;
  color: #94a3b8;
  line-height: 1.45;
  margin: 0;
}

.ios-stats-card {
  display: flex;
  justify-content: space-around;
  align-items: center;
  background: rgba(20, 25, 38, 0.7);
  backdrop-filter: blur(14px);
  -webkit-backdrop-filter: blur(14px);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 18px;
  padding: 14px 18px;
}

.stat-cell {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.stat-val {
  font-family: 'Poppins', sans-serif;
  font-size: 1.35rem;
  font-weight: 700;
  color: #ffffff;
}

.stat-val.green { color: #4ade80; }
.stat-val.red { color: #f87171; }

.stat-lbl {
  font-size: 0.72rem;
  font-weight: 600;
  text-transform: uppercase;
  color: #64748b;
  margin-top: 3px;
}

.stat-div {
  width: 1px;
  height: 26px;
  background: rgba(255, 255, 255, 0.08);
}

.ios-search-bar {
  display: flex;
  align-items: center;
  gap: 12px;
  background: rgba(20, 25, 38, 0.7);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 14px;
  padding: 11px 16px;
}

.ios-s-icon {
  font-size: 1.18rem;
  color: #64748b;
}

.ios-s-input {
  flex: 1;
  background: transparent;
  border: none;
  outline: none;
  color: #ffffff;
  font-size: 0.95rem;
}

.ios-s-clear {
  background: transparent;
  border: none;
  color: #64748b;
  cursor: pointer;
  font-size: 0.9rem;
  padding: 4px;
}

.ios-segmented-control {
  display: flex;
  background: rgba(20, 25, 38, 0.7);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 14px;
  padding: 5px;
}

.ios-seg-item {
  flex: 1;
  background: transparent;
  border: none;
  color: #94a3b8;
  font-family: 'Inter', sans-serif;
  font-size: 0.85rem;
  font-weight: 600;
  padding: 10px 14px;
  border-radius: 10px;
  cursor: pointer;
}

.ios-seg-item.selected {
  background: #ef4444;
  color: #ffffff;
  box-shadow: 0 2px 10px rgba(239, 68, 68, 0.35);
}

.ios-genre-chips {
  display: flex;
  gap: 10px;
  overflow-x: auto;
  scrollbar-width: none;
  padding-bottom: 4px;
}
.ios-genre-chips::-webkit-scrollbar { display: none; }

.ios-chip {
  background: rgba(255, 255, 255, 0.06);
  border: 1px solid rgba(255, 255, 255, 0.09);
  color: #94a3b8;
  padding: 8px 16px;
  border-radius: 20px;
  font-size: 0.82rem;
  font-weight: 500;
  white-space: nowrap;
  cursor: pointer;
}

.ios-chip.active {
  background: #ffffff;
  color: #090b10;
  font-weight: 600;
  border-color: #ffffff;
}

.ios-movie-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 16px;
}

@media (min-width: 600px) {
  .ios-movie-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (min-width: 900px) {
  .ios-movie-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

.ios-empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 44px 20px;
  text-align: center;
}

.ios-empty-icon-wrap {
  width: 56px;
  height: 56px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.04);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 26px;
  color: #475569;
  margin-bottom: 12px;
}

.ios-empty-text {
  color: #64748b;
  font-size: 0.9rem;
  margin: 0 0 14px 0;
}

.ios-reset-btn {
  background: rgba(239, 68, 68, 0.16);
  border: 1px solid rgba(239, 68, 68, 0.4);
  color: #f87171;
  font-size: 0.84rem;
  font-weight: 600;
  padding: 8px 16px;
  border-radius: 12px;
  cursor: pointer;
}

.ios-alert-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(6, 9, 15, 0.88);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9999;
  padding: 24px;
  transform: translateZ(0);
}

.ios-alert-card {
  background: #101524;
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 22px;
  padding: 24px;
  width: 100%;
  max-width: 340px;
  text-align: center;
  box-shadow: 0 24px 44px rgba(0, 0, 0, 0.75);
}

.ios-alert-icon-box {
  width: 54px;
  height: 54px;
  border-radius: 50%;
  background: rgba(239, 68, 68, 0.16);
  color: #f87171;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.5rem;
  margin: 0 auto 14px auto;
  border: 1px solid rgba(239, 68, 68, 0.35);
}

.alert-title {
  font-family: 'Poppins', sans-serif;
  font-size: 1.22rem;
  font-weight: 700;
  color: #ffffff;
  margin: 0 0 8px 0;
}

.alert-desc {
  font-family: 'Inter', sans-serif;
  font-size: 0.86rem;
  color: #94a3b8;
  margin: 0 0 22px 0;
  line-height: 1.5;
}

.ios-alert-actions {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

.ios-alert-btn {
  padding: 12px;
  border-radius: 12px;
  font-family: 'Inter', sans-serif;
  font-size: 0.88rem;
  font-weight: 600;
  cursor: pointer;
  border: none;
}

.ios-alert-btn.cancel {
  background: rgba(255, 255, 255, 0.08);
  color: #cbd5e1;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.ios-alert-btn.confirm {
  background: #ef4444;
  color: #ffffff;
  box-shadow: 0 4px 14px rgba(239, 68, 68, 0.45);
}
</style>