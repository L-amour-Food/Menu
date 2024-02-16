<script setup>
import { onMounted, reactive, computed } from 'vue'

const data = reactive({
  closed: false,
  indisponible: false,
  current: null,
  next: null,
  loaded: false,
  titre: '',
  soustitre: '',
});
onMounted(() => {
  data.closed = document.location.hash.includes('closed');
  data.indisponible = document.location.hash.includes('indisponible');
  console.log('mounted!');
  fetch('https://lamourfood.fr/wp-json/custom/v1/menu')
    .then(response => response.json())
    .then(menu => {
      if (data.closed) {
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
    src="https://lamourfood.fr/wp-content/uploads/2023/09/lamourfood-online-video-cutter.com_.mp4">
  </video>

  <div class="menu" :style="style">

    <div v-if="data.loaded">
      <div>
        <figure class="logo"><img src="/logo.png"></figure>

        <template v-if="data.current">
          <div class="titre">
            <span>{{ data.titre }}</span>
            <template v-if="data.soustitre">
              <i style="font-size: 60%;">{{ data.soustitre }}</i>
            </template>
          </div>
          <div style="font-size: 90%;">
            <p class="plats subtitle has-text-black is-7" id="description" v-html="data.current.description"></p>
            <template v-if="data.current && data.next">
              <div style="color:gray;font-size: smaller;">
                <small v-html="'<b>Demain</b>: ' + data.next.description.split('\n')[0]"></small>
              </div>
            </template>
          </div>
        </template>
        <template v-else>
          <p class="plats">À bientôt dans la cantina... </p>
        </template>

      </div>
      <div class="colonne-qr">
        <template v-if="data.current">
          <template v-if="data.current.disponible">
            <p class="texte"><b>Réserver à la cantina<br>ou en ligne sur</b></p>
          </template>
        </template>
        <template v-else>
          <p><small>Plus d'infos sur</small></p>
        </template>
        <div class="qr">
          <template v-if="data.current && (data.indisponible || !data.current?.disponible)">
            <span class="off">Réservation terminée</span>
          </template>
          <img src="/qr.png">
        </div>
        <a href="https://lamourfood.Fr"><u>lamourfood.fr</u></a>
      </div>
    </div>
  </div>
</template>

<style>
.menu>figure>img {
  width: 30vw;
}

.menu>div {
  padding: var(--grande-marge);
  position: relative;
  display: flex;
  gap: 10vw;
  flex: 1;
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

#description p {
  font-size: 80%;
}

.titre {
  display: flex;
  align-items: center;
  font-size: 60%;
  gap: 1.5em;
}</style>
