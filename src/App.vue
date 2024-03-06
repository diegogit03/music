<script setup>
  import { ref, onMounted, computed } from 'vue';

  const term = ref('');
  const musicProgress = ref(0);
  const musicDuration = ref(0);
  const results = ref([]);

  function createNewPlayer(videoId) {
    // Destrua o player existente, se houver
    if (player) {
        player.destroy();
    }

    // Crie um novo player com o ID do vídeo especificado
    player = new YT.Player('player', {
        height: '0',
        width: '0',
        videoId: videoId,
        events: {
            'onReady': onPlayerReady,
            'onStateChange': onPlayerStateChange
        }
    });
}

  function onPlayerStateChange(event) {
      // Verifique o novo estado do player
      switch (event.data) {
          case YT.PlayerState.PLAYING:
              startProgressTimer();
              console.log('O vídeo começou a ser reproduzido');
              break;
          case YT.PlayerState.PAUSED:
            stopProgressTimer();
              console.log('O vídeo foi pausado');
              break;
          // case YT.PlayerState.ENDED:
          //     // O vídeo chegou ao fim
          //     console.log('O vídeo chegou ao fim');
          //     break;
          // Outros estados do player podem ser tratados aqui
      }
  }
  function onPlayerReady(event) {
    musicDuration.value = player.getDuration();
    event.target.playVideo();
    // startProgressTimer();
  }

  const progress = computed(() => ((musicProgress.value / musicDuration.value) * 100) + '%')

  async function play (videoId) {
    musicProgress.value = 0
    createNewPlayer(videoId);
  }

  function startProgressTimer() {
    progessTimer = setInterval(() => {
      musicProgress.value++;
      console.log(musicDuration.value)
      console.log(musicProgress.value)
      console.log()
    }, 1000);
  }

  function stopProgressTimer() {
    clearInterval(progessTimer);
  }

  function toggleMusic () {
    if (player.getPlayerState() === YT.PlayerState.PAUSED) {
      
      player.playVideo();
    } else {
      
      player.pauseVideo();
    }
  }

  async function search () {
    const response = await fetch('https://www.googleapis.com/youtube/v3/search?' + new URLSearchParams({
      part: 'snippet',
      q: term.value,
      key: 'AIzaSyALnTt_IK290oqwOlVFDN2CFBLn4_nhStU',
      maxResults: 50
    }));
    const data = await response.json();

    results.value = data.items
  }

  onMounted(() => {
    window.player = null;
    window.progessTimer = null;
  });
</script>

<template>
  <div id="player" style="display: none;"></div>
  <nav class="navbar bg-body-secondary">
    <div class="container-fluid">
      <h1>Diego<span class="text-success">fy</span></h1>
      <form class="d-flex" role="search" @submit.prevent="search">
        <div class="input-group">
          <input v-model="term" class="form-control me-2" type="search" placeholder="Encontrar Hits.." aria-label="Encontrar Hits..">
          <button class="btn btn-success input-group-text" type="submit"><i class="bi bi-search"></i></button>
        </div>
      </form>
    </div>
  </nav>

  <div class="player bg-body-secondary p-4">
    <div class="w-100">
      <div class="d-flex w-100" style="justify-content: center;">
        <button class="btn"></button>
        <button @click="toggleMusic" class="btn"><i class="bi bi-pause-circle-fill"></i></button>
        <button class="btn"></button>
      </div>
      <div class="progress bg-secondary" role="progressbar" aria-label="Basic example">
        <div class="progress-bar bg-success" :style="{ width: progress }"></div>
      </div>
    </div>
  </div>

  <div class="container mt-2">
    <ul class="list-group">
      <template v-for="result in results">
        <li class="list-group-item">
          <div class="d-flex align-items-center">
            <img class="mr-2" :src="result.snippet.thumbnails.default.url" alt="">
            <button @click="play(result.id.videoId)" class="btn btn-link"><h2>{{ result.snippet.title }}</h2></button>
          </div>
        </li>
      </template>
    </ul>
  </div>
</template>

<style scoped>
  .player {
    position: fixed;
    bottom: 0;
    width: 100%;
    z-index: 100;
    display: flex;
    align-items: center;
  }
</style>
