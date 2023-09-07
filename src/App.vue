<script setup>
import { onMounted, reactive, computed } from 'vue'

const data = reactive({
  closed: false,
  current: null,
  next: null,
  loaded: false,
  titre: '',
  soustitre: '',
});
onMounted(() => {
  data.closed = document.location.hash.includes('closed');
  console.log('mounted!');
  fetch('https://lamourfood.fr/wp-json/custom/v1/menu')
    .then(response => response.json())
    .then(menu => {
      if(data.closed) {
        data.loaded = true;
        return;
      }
      
      for (let repas of menu) {
        if (isToday(repas.time)) {
          data.current = repas
          data.titre = 'Menu du jour';
          data.soustitre = 'Pensez à réserver le plus tôt possible, et au plus tard avant 11h30';
          break;
        }
      }
      if (data.current) {
        for (let repas of menu) {
          if (isTomorrow(repas.time)) {
            data.next = repas
            break;
          }
        }
      } else {
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
      // 'background-image': `url(/nappe-rouge.webp)`,
    }
  }
})
</script>

<template>
  <video v-if="data.loaded && !data.current" autoplay loop muted
    src="https://lamourfood.fr/wp-content/uploads/2022/02/lamourfood.mp4?_=1">
</video>

  <div class="menu" :style="style">
    <figure></figure>

    <div v-if="data.loaded">
      <div>
        <figure><img src="/logo.png"></figure>

        <template v-if="data.current">
          <p class="title has-text-black is-5">
            {{ data.titre }}
            <template v-if="data.soustitre">
              <br><small style="font-size: 60%;">{{ data.soustitre }}</small>
            </template>
          </p>
          <p class="subtitle has-text-black is-7" v-html="data.current.description">
          </p>
          <template v-if="data.current && data.next">
            <div style="color:gray;font-size: smaller;">
              <small v-html="'<b>Demain</b>: ' + data.next.description.split('\n')[0]"></small>
            </div>
          </template>
        </template>
        <template v-else>
          <p class="subtitle has-text-black">À bientôt dans la cantina... </p>
        </template>

      </div>
      <div>
        <template v-if="data.current">
          <template v-if="!data.current.disponible">
            <b class="off"><small>Réserveration terminée</small></b>
          </template>
          <template v-if="data.current.disponible">
            <center style="font-size: 70%; line-height: 1;"><b>Réserver à la cantina<br>ou en ligne sur</b></center>
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
video {
  position: fixed;
  width: 100vw;
  height: 100vh;
  object-fit: cover;
  transform: scale(1.1);
}
</style>
