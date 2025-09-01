<script lang="ts" setup>
import { ref, onMounted, onBeforeUnmount, computed } from 'vue';

const props = defineProps({
  videoSrc: {
    type: String,
    required: true,
  },
  posterUrl: {
    type: String,
    default:
      'data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMzIwIiBoZWlnaHQ9IjI0MCIgdmlld0JveD0iMCAwIDMyMCAyNDAiIGZpbGw9Im5vbmUiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+CjxyZWN0IHdpZHRoPSIzMjAiIGhlaWdodD0iMjQwIiBmaWxsPSJ1cmwoI3BhaW50MF9saW5lYXJfMF8xKSIvPgo8ZGVmcz4KPGxpbmVhckdyYWRpZW50IGlkPSJwYWludDBfbGluZWFyXzBfMSIgeDE9IjAiIHkxPSIwIiB4Mj0iMzIwIiB5Mj0iMjQwIiBncmFkaWVudFVuaXRzPSJ1c2VyU3BhY2VPblVzZSI+CjxzdG9wIHN0b3AtY29sb3I9IiM2NjdlZWEiLz4KPHN0b3Agb2Zmc2V0PSIxIiBzdG9wLWNvbG9yPSIjNzY0YmEyIi8+CjwvbGluZWFyR3JhZGllbnQ+CjwvZGVmcz4KPC9zdmc+Cg==',
  },
});

const isVideoLoaded = ref(false);
const showPlaceholder = ref(true);
const showLoading = ref(true);
const heroVideoRef = ref<HTMLVideoElement | null>(null);
const intersectionObserver = ref<IntersectionObserver | null>(null);
const isLoadingError = ref(false);

const videoClasses = computed(() => ({
  'hero-video': true,
  loaded: isVideoLoaded.value,
}));

const loadingClasses = computed(() => ({
  'loading-indicator': true,
  hidden: !showLoading.value,
}));

// Gère le chargement de la vidéo et les transitions
const onVideoLoaded = () => {
  console.log('✅ Vidéo chargée et prête à jouer.');
  isVideoLoaded.value = true;
  showLoading.value = false;
  // Utilisez un petit délai pour une transition fluide
  setTimeout(() => {
    showPlaceholder.value = false;
  }, 800);
};

// Gère les erreurs de chargement
const onVideoError = (e: Event) => {
  console.error('❌ Erreur de chargement vidéo:', e);
  isLoadingError.value = true;
  showLoading.value = false;
  showPlaceholder.value = true; // S'assure que le placeholder reste visible
  // Afficher un message d'erreur ou d'un bouton de rechargement peut être ajouté ici
};

// Tente de lancer l'autoplay
const attemptAutoplay = () => {
  if (!heroVideoRef.value) return;
  const playPromise = heroVideoRef.value.play();
  if (playPromise !== undefined) {
    playPromise
      .then(() => {
        console.log('🎬 Autoplay réussi.');
      })
      .catch((error: Error) => {
        console.log('🚫 Autoplay bloqué par le navigateur:', error);
        // Si l'autoplay est bloqué, on peut laisser le placeholder
        // ou ajouter un bouton "play"
      });
  }
};

onMounted(() => {
  const videoElement = heroVideoRef.value;
  if (!videoElement) {
    console.warn('🎥 Élément vidéo non trouvé. Utilisation du fallback CSS.');
    showPlaceholder.value = true;
    showLoading.value = false;
    return;
  }

  // Événements pour gérer le chargement
  videoElement.addEventListener('loadeddata', onVideoLoaded);
  videoElement.addEventListener('canplay', attemptAutoplay);
  videoElement.addEventListener('error', onVideoError);

  // Intersection Observer pour optimiser la lecture
  if ('IntersectionObserver' in window) {
    intersectionObserver.value = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            if (videoElement.paused) {
              videoElement.play().catch(() => {});
            }
          } else {
            if (!videoElement.paused) {
              videoElement.pause();
            }
          }
        });
      },
      { threshold: 0.5 }
    );
    intersectionObserver.value.observe(videoElement);
  }
});

onBeforeUnmount(() => {
  // Nettoyage des événements et de l'observer
  const videoElement = heroVideoRef.value;
  if (videoElement) {
    videoElement.removeEventListener('loadeddata', onVideoLoaded);
    videoElement.removeEventListener('canplay', attemptAutoplay);
    videoElement.removeEventListener('error', onVideoError);
  }
  if (intersectionObserver.value) {
    intersectionObserver.value.disconnect();
  }
});
</script>

<template>
  <section class="hero-section">
    <div :class="loadingClasses">
      <div class="spinner"></div>
    </div>

    <div v-if="showPlaceholder || isLoadingError" class="video-placeholder"></div>

    <video
      v-if="!isLoadingError"
      ref="heroVideoRef"
      :class="videoClasses"
      playsinline
      muted
      loop
      autoplay
      preload="auto"
      crossorigin="anonymous"
      :poster="props.posterUrl"
    >
      <source :src="props.videoSrc" type="video/mp4" />
      <p>Votre navigateur ne supporte pas la lecture vidéo HTML5. Considérez une mise à jour pour une expérience moderne.</p>
    </video>

    <div class="hero-overlay"></div>

    <div class="hero-content">
      <h1 class="hero-title">
        <slot name="title"> Titre par défaut </slot>
      </h1>
      <p class="hero-subtitle">
        <slot name="subtitle">
          Démo vidéo hero pour montrer une intégration propre dans une application Vue 3.
        </slot>
      </p>
      <a href="#" class="hero-cta">
        <slot name="cta">
          Appel à l'action
        </slot>
      </a>
    </div>
  </section>
</template>

<style scoped>
/* Scoped pour s'assurer que les styles n'affectent que ce composant */

/*
==========================================
HERO SECTION - CONTENEUR PRINCIPAL
==========================================
*/

.hero-section {
  position: relative;
  width: 100%;
  height: 100vh;
  min-height: 600px;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #000;
}

/*
==========================================
PLACEHOLDER VIDÉO
==========================================
*/

.video-placeholder {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1;
  background: linear-gradient(45deg, #667eea 0%, #764ba2 25%, #f093fb 50%, #f5576c 75%, #4facfe 100%);
  background-size: 400% 400%;
  animation: gradientShift 8s ease infinite;
  opacity: 1;
  transition: opacity 0.8s ease-in-out;
}

@keyframes gradientShift {
  0% {
    background-position: 0% 50%;
  }

  50% {
    background-position: 100% 50%;
  }

  100% {
    background-position: 0% 50%;
  }
}

/*
==========================================
ÉLÉMENT VIDÉO RÉEL
==========================================
*/

.hero-video {
  position: absolute;
  width: 100%;
  height: 100%;
  object-fit: cover;
  z-index: 2;
  opacity: 0;
  transition: opacity 0.8s ease-in-out;
}

.hero-video.loaded {
  opacity: 1;
}

/*
==========================================
OVERLAY
==========================================
*/

.hero-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, rgba(0, 0, 0, 0.5) 0%, rgba(0, 0, 0, 0.2) 100%);
  z-index: 3;
}

/*
==========================================
CONTENU TEXTE DU HERO
==========================================
*/

.hero-content {
  position: relative;
  z-index: 4;
  text-align: center;
  color: white;
  max-width: 1200px;
  padding: 0 2rem;
  animation: fadeInUp 1.2s ease-out;
}

@keyframes fadeInUp {
  0% {
    opacity: 0;
    transform: translateY(30px);
  }

  100% {
    opacity: 1;
    transform: translateY(0);
  }
}

.hero-title {
  font-size: clamp(2.5rem, 8vw, 6rem);
  font-weight: 700;
  margin-bottom: 1.5rem;
  background: linear-gradient(135deg, #fff 0%, #e2e8f0 100%);
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  text-shadow: 0 0 30px rgba(255, 255, 255, 0.3);
}

.hero-subtitle {
  font-size: clamp(1.25rem, 3vw, 1.8rem);
  margin-bottom: 2.5rem;
  opacity: 0.95;
  font-weight: 300;
  max-width: 600px;
  margin-left: auto;
  margin-right: auto;
}

/*
==========================================
BOUTON CALL-TO-ACTION
==========================================
*/

.hero-cta {
  display: inline-block;
  padding: 1rem 2.5rem;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  text-decoration: none;
  border-radius: 50px;
  font-weight: 600;
  font-size: 1.1rem;
  transition: all 0.3s ease;
  box-shadow: 0 10px 30px rgba(102, 126, 234, 0.3);
  user-select: none;
}

.hero-cta:hover {
  transform: translateY(-2px);
  box-shadow: 0 15px 40px rgba(102, 126, 234, 0.4);
}

.hero-cta:active {
  transform: translateY(0);
}

/*
==========================================
INDICATEUR DE CHARGEMENT
==========================================
*/

.loading-indicator {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 5;
  opacity: 1;
  transition: opacity 0.5s ease;
  pointer-events: none;
}

.loading-indicator.hidden {
  opacity: 0;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 3px solid rgba(255, 255, 255, 0.3);
  border-top: 3px solid white;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }

  100% {
    transform: rotate(360deg);
  }
}

/*
==========================================
RESPONSIVE DESIGN
==========================================
*/

@media (max-width: 1024px) {
  .hero-section {
    height: 80vh;
  }
}

@media (max-width: 768px) {
  .hero-section {
    height: 70vh;
    min-height: 500px;
  }

  .hero-content {
    padding: 0 1rem;
  }

  .hero-overlay {
    background: linear-gradient(135deg, rgba(0, 0, 0, 0.6) 0%, rgba(0, 0, 0, 0.3) 100%);
  }
}

@media (max-width: 480px) {
  .hero-subtitle {
    font-size: 1.1rem;
  }

  .hero-cta {
    padding: 0.8rem 2rem;
    font-size: 1rem;
  }
}

/*
==========================================
PRÉFÉRENCES UTILISATEUR
==========================================
*/

@media (prefers-reduced-motion: reduce) {
  .video-placeholder {
    animation: none;
  }

  .hero-content {
    animation: none;
  }

  * {
    transition-duration: 0.1s !important;
  }
}
</style>