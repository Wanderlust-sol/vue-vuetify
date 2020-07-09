<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <div class="d-flex align-center">
        <v-img
          alt="Vuetify Logo"
          class="shrink mr-2"
          contain
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-logo-dark.png"
          transition="scale-transition"
          width="40"
        />

        <v-img
          alt="Vuetify Name"
          class="shrink mt-1 hidden-sm-and-down"
          contain
          min-width="100"
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-name-dark.png"
          width="100"
        />
      </div>

      <v-spacer></v-spacer>

      <v-btn href="https://github.com/vuetifyjs/vuetify/releases/latest" target="_blank" text>
        <span class="mr-2">Latest Release</span>
        <v-icon>fas fa-external-link-alt</v-icon>
      </v-btn>
    </v-app-bar>

    <v-main>
      <v-text-field label="ID"></v-text-field>
      <div id="audio-uploader">
        <v-btn @click="getAudio">시작</v-btn>
        <!-- <v-btn @click="stopAudio">정지</v-btn> -->
      </div>
    </v-main>
  </v-app>
</template>

<script>
export default {
  name: 'App',

  components: {},

  data: () => ({
    //
  }),
  methods: {
    async getAudio() {
      // AudioContext 객체인데 음원과 관련된 모든 처리가 일어난다.
      const audioContext = await new (window.AudioContext || window.webkitAudioContext)();
      console.log('audioContext', audioContext);
      const res = await fetch('http://localhost:8081/1to20.wav');
      // arrayBuffer()를 이용해 오디오 파일 데이터를 고정 길이의 원시 이진 데이터 버퍼 객체로 나타낸다.
      const bufferData = await res.arrayBuffer();
      // 버퍼 객체를 비동기적으로 디코딩한다(raw한 오디오 데이터일뿐 아직 하나의 노드가 아니여서 사용할 수 없는 상태이다)
      // decodedData 는 AudioBuffer이다.
      const decodedData = await audioContext.decodeAudioData(bufferData);
      // sound source 생성
      const source = audioContext.createBufferSource(); // 생성했던 데이터는 한번 재생이 끝나고 재접근 불가

      // 생성된 source에 어떤 사운드(decodedData)를 플레이할지 알려준다.
      source.buffer = decodedData;

      // 스피커에 source를 연결한다...!
      source.connect(audioContext.destination);
      // source.start();
      await console.log('bufferData', bufferData, 'decodeData', decodedData, 'source', source);
    },
  },
  mounted() {},
};
</script>
