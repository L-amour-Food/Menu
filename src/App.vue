<script setup>
import { onMounted, reactive, computed } from 'vue'

const data = reactive({
  current: null,
  loaded: false
});
onMounted(() => {
  console.log('mounted!');
  fetch('https://lamourfood.fr/wp-json/custom/v1/menu')
    .then(response => response.json())
    .then(menu => {
      for (let repas of menu) {
        if (!data.current && isToday(repas.time)) {
          data.current = repas
          break;
        }
      }
      data.loaded = true;
    })
})

const style = computed(() => {
  if (!data.loaded) return;
  if (data.current?.illustration) {
    return {
      'background-image': `url(${data.current.illustration})`,
    }
  } else {
    return {
      'background-image': `url(/nappe-rouge.webp)`,
    }
  }
})
</script>

<template>
  <div class="menu" :style="style">
    <figure></figure>

    <div v-if="data.loaded">
      <div>
        <figure><img src="/logo.png"></figure>

        <template v-if="data.current">
          <p class="title has-text-black is-5">
            Menu du jour
          </p>
          <p class="subtitle has-text-black is-7" v-html="data.current.description">
          </p>
        </template>
        <template v-else>
          <p class="title has-text-black">Pas de service aujourd'hui !</p>
        </template>

      </div>
      <div>
        <template v-if="data.current">
          <p class="mb-2"><small>Réserver avant 11h sur</small></p>
          <div class="qr"><img src="/qr.png"></div>
          <a href="https://lamourfood.Fr" class="title is-6"><u>lamourfood.fr</u></a>
        </template>
      </div>
    </div>
  </div>
</template>

<style>
.subtitle {
  display: flex;
  flex-direction: column;
  gap: 0.5vw;
}

.subtitle h3 {
  font-weight: 700;
  font-size: 125%;
  color: #ff406d;
}

.menu {
  background-size: cover;
  background-position: center center;
  height: 100vh;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.menu>figure {
  padding: 2em;
  flex: 1;
  position: relative;
  z-index: 1;
}

.menu>figure>img {
  width: 30vw;
}

.menu>div {
  padding: 2em;
  position: relative;
  display: flex;
  gap: 10vw;
}

.menu>div>* {
  z-index: 1;
}

.menu>div>div:first-of-type {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: end;
}

.menu>div>div:last-of-type {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: end;
}

.menu>div:after {
  content: '';
  position: absolute;
  background: linear-gradient(to bottom, rgba(255, 255, 255, 0) 0%, rgba(255, 255, 255, 0.5) 50%, rgba(255, 255, 255, 1) 100%);
  left: 0;
  bottom: 0;
  width: 100%;
  height: 200%;
  z-index: 0;
}

.qr {
  width: 15vw;
  height: 15vw;
  border: 1em solid white;
}

.qr img {
  display: block;
  width: 100%;
  height: 100%;
}
</style>
