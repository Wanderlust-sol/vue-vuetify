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
        <v-btn @click="draw">시작</v-btn>
        <!-- <v-btn @click="stopAudio">정지</v-btn> -->
      </div>
      <svg id="waveform" preserveAspectRatio="none">
        <g id="waveform-path-group"></g>
      </svg>
    </v-main>
  </v-app>
</template>

<script>
export default {
  name: 'App',

  components: {},

  data() {
    return {
      audioBuffer: null,
      sampleRate: 0,
      peaks: [],
    };
  },
  methods: {
    getPeaks() {
      const sampleSize = this.audioBuffer.length / this.sampleRate; // 20
      const sampleStep = Math.floor(sampleSize / 10) || 1; // 2
      const channels = this.audioBuffer.numberOfChannels;
      const mergedPeaks = [];

      for (let i = 0; i < channels; i++) {
        const peaks = this.audioBuffer.getChannelData(i);

        Array(this.sampleRate)
          .fill()
          .forEach((v, newPeakIndex) => {
            const start = Math.floor(newPeakIndex * sampleSize);
            const end = Math.floor(start + sampleSize);

            let min = peaks[0];
            let max = peaks[0];

            for (let sampleIndex = start; sampleIndex < end; sampleIndex += sampleStep) {
              const idx = peaks[sampleIndex];
              if (idx > max) {
                max = idx;
              } else if (idx < min) {
                min = idx;
              }
            }

            if (i === 0 || max > mergedPeaks[2 * newPeakIndex]) {
              // console.log(this.peaks[2 * newPeakIndex]);
              mergedPeaks[2 * newPeakIndex] = max;
            }
            if (i === 0 || min < mergedPeaks[2 * newPeakIndex + 1]) {
              // console.log(this.peaks[2 * newPeakIndex + 1]);
              mergedPeaks[2 * newPeakIndex + 1] = min;
            }
          });
      }

      return mergedPeaks;
    },
    async getAudio() {
      // AudioContext 객체인데 음원과 관련된 모든 처리가 일어난다.
      const audioContext = await new (window.AudioContext || window.webkitAudioContext)();

      const res = await fetch('http://localhost:8081/1to20.wav');
      // arrayBuffer()를 이용해 오디오 파일 데이터를 고정 길이의 원시 이진 데이터 버퍼 객체로 나타낸다.
      const bufferData = await res.arrayBuffer();
      // 버퍼 객체를 비동기적으로 디코딩한다(raw한 오디오 데이터일뿐 아직 하나의 노드가 아니여서 사용할 수 없는 상태이다)
      // decodedData 는 AudioBuffer이다.
      const audioBuffer = await audioContext.decodeAudioData(bufferData);

      this.audioBuffer = audioBuffer;
      this.sampleRate = audioBuffer.sampleRate;

      // // sound source 생성
      // const source = audioContext.createBufferSource(); // 생성했던 데이터는 한번 재생이 끝나고 재접근 불가

      // // 생성된 source에 어떤 사운드(decodedData)를 플레이할지 알려준다.
      // source.buffer = audioBuffer;

      // // 스피커에 source를 연결한다...!
      // source.connect(audioContext.destination);
      // // source.start();
      // await console.log('bufferData', bufferData, 'decodeData', audioBuffer, 'source', source);
    },

    draw() {
      const waveFormBox = document.getElementById('waveform');
      waveFormBox.setAttribute('viewBox', `0 -1 ${this.sampleRate} 2`);

      const peaks = this.getPeaks();
      console.log(peaks);
      if (this.audioBuffer) {
        const totalPeaks = peaks.length;

        let d = '';
        for (let peakNumber = 0; peakNumber < totalPeaks; peakNumber++) {
          if (peakNumber % 2 === 0) {
            d += ` M${Math.floor(peakNumber / 2)}, ${peaks.shift()}`;
          } else {
            d += ` L${Math.floor(peakNumber / 2)}, ${peaks.shift()}`;
          }
        }
        console.log('d', d);
        const path = document.createElementNS('http://www.w3.org/2000/svg', 'path');
        path.setAttributeNS(null, 'd', d);

        const waveFormPathGroup = document.getElementById('waveform-path-group');

        waveFormPathGroup.appendChild(path);
      }
    },
  },
  mounted() {
    this.getAudio();
  },
};
</script>

<style scoped>
#waveform {
  stroke: black;
  width: 100%;
  height: 100%;
  background-color: #00ced1;
}
</style>
