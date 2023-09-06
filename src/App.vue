<script setup>
import { onMounted, reactive, computed } from 'vue'

const data = reactive({
  current: null,
  loaded: false,
  titre: ''
});
onMounted(() => {
  console.log('mounted!');
  fetch('https://lamourfood.fr/wp-json/custom/v1/menu')
    .then(response => response.json())
    .then(menu => {
      for (let repas of menu) {
        if (isToday(repas.time)) {
          console.log(repas)
          data.current = repas
          data.titre = 'Menu du jour';
          break;
        }
      }
      if (!data.current) {
        for (let repas of menu) {
          if (isTomorrow(repas.time)) {
            data.current = repas
            data.titre = 'Menu de demain';
            break;
          }
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
            {{ data.titre }}
          </p>
          <p class="subtitle has-text-black is-7" v-html="data.current.description">
          </p>
        </template>
        <template v-else>
          <p class="subtitle has-text-black">Menu en cours de préparation ... </p>
        </template>

      </div>
      <div>
        <template v-if="data.current">
          <template v-if="!data.current.disponible">
            <b class="off"><small>Réserveration terminée</small></b>
          </template>
          <template v-if="data.current.disponible">
            <p><small>Réserver avant 11h30 sur</small></p>
          </template>
        </template>
        <template v-else>
          <p><small>Plus d'infos sur</small></p>
        </template>
        <div class="qr mt-2"><img src="/qr.png"></div>
        <a href="https://lamourfood.Fr" class="title is-6"><u>lamourfood.fr</u></a>
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

.off {
  text-align: center;
  position: absolute;
  bottom: 0;
  left: 50%;
  color: red;
  text-transform: uppercase;
  text-shadow: 2px 2px white, -2px -2px white;
  transform: translate(-50%, -75%) rotate(-45deg);
}

.menu>div>div:last-of-type {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: end;
  position: relative;
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
