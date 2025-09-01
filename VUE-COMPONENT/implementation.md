# Implementation exemple

```vue
<template>
  <div>
    <HeroVideo
      videoSrc="https://cdn.midjourney.com/video/a92a6ff2-1d26-4194-a59c-4cc2d464cde8/0.mp4"
      posterUrl="/images/mon-poster-hero.jpg"
    >
      <template #title>Mon Super Titre</template>
      <template #subtitle
        >Une phrase d'accroche incroyable et impactante.</template
      >
      <template #cta>Cliquez ici !</template>
    </HeroVideo>
  </div>
</template>

<script setup>
import HeroVideo from "./components/HeroVideo.vue";
</script>
```
