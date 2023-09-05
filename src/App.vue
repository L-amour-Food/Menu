<script setup>
import { onMounted, reactive, computed } from 'vue'

const data = reactive({
  current: false,
  next: false
});
onMounted(() => {
  console.log('mounted!');
  fetch('https://lamourfood.fr/wp-json/custom/v1/menu')
    .then(response => response.json())
    .then(menu => {
      for (let repas of menu) {
        if (isToday(repas.time)) {
          data.current = repas
        } else if (isTomorrow(repas.time)) {
          data.next = repas
        }
        if (data.current && data.next) {
          break
        }
      }
    })
})

const style = computed(() => {
  if (data.current.illustration) {
    return {
      'background-image': `url(${data.current.illustration})`,
    }
  }
})
</script>

<template>
  <div class="menu" :style="style">
    <figure><img src="/logo.png"></figure>
    <div>
      <div>

        <template v-if="data.current">
          <p class="title has-text-black">
            Menu du jour
          </p>
          <p class="subtitle has-text-black is-6">
            {{ data.current.nom }}
          </p>
        </template>
        <template v-else>
          <p class="title has-text-black">Pas de service aujourd'hui !</p>
        </template>

      </div>
      <div>
        <span class="title is-5">Réserver avant 11h sur</span>
        <div class="qr"><img src="/qr.png"></div>
        <a href="https://lamourfood.Fr" class="title is-5"><u>lamourfood.fr</u></a>
      </div>
    </div>
  </div>
</template>

<style scoped>
figure {}

.subtitle {
  white-space: pre-wrap;
}

.menu {
  background-size: cover;
  background-position: center center;
  background-image: url(/nappe-rouge.webp);
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

.menu>div {
  padding: 2em;
  position: relative;
  display: flex;
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
  background: linear-gradient(to bottom, rgba(255, 255, 255, 0) 0%, rgba(255, 255, 255, 0.5) 30%, rgba(255, 255, 255, 1) 100%);
  left: 0;
  bottom: 0;
  width: 100%;
  height: 200%;
  z-index: 0;
}


.hero-body {
  /* W3C, IE10+, FF16+, Chrome26+, Opera12+, Safari7+ */

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
