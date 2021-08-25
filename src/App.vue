<template>
  <main>
    <div class="input-group">
      <input id="freq" type="number" @input="parseFreq" :value="frequency" />
      Hz
    </div>
    <div class="input-group">
      <input id="note" type="text" @input="parsePitch" :value="pitch" />
      cents
    </div>
    <div class="input-group">
      <input id="period" type="text" @input="parsePeriod" :value="period" />
      ms
    </div>
    <div class="input-group">
      <input
        id="wavelength"
        type="number"
        @input="parseWavelength"
        :value="wavelength"
      />
      m
    </div>
    <div class="input-group">
      <input
        id="samples"
        type="number"
        @input="parseSamples"
        :value="samples"
      />
      samples @ 44.1kHz
    </div>
  </main>
</template>

<script>
export default {
  data() {
    return {
      frequency: 0,
      pitch: '',
      period: 0,
      samples: 0,
      wavelength: 0,
      LOG_HALF_STEP: Math.log(Math.pow(2, 1 / 12)),
      NOTES_TO_A4: {
        [-11]: 'A#',
        [-10]: 'B',
        [-9]: 'C',
        [-8]: 'C#',
        [-7]: 'D',
        [-6]: 'D#',
        [-5]: 'E',
        [-4]: 'F',
        [-3]: 'F#',
        [-2]: 'G',
        [-1]: 'G#',
        0: 'A',
        1: 'A#',
        2: 'B',
        3: 'C',
        4: 'C#',
        5: 'D',
        6: 'D#',
        7: 'E',
        8: 'F',
        9: 'F#',
        10: 'G',
        11: 'G#'
      },
      SPEED_OF_SOUND: 345,
      SAMPLE_RATE: 44100
    };
  },
  methods: {
    parseFreq(e) {
      const REF_PITCH_A4 = 440;

      this.frequency = e.target.value;

      const freqRatio = this.frequency / REF_PITCH_A4;
      const logFreqRatio = Math.log(freqRatio);
      const stepCount = logFreqRatio / this.LOG_HALF_STEP;
      const roundedStepCount = Math.round(stepCount);

      let octCount =
        roundedStepCount > 0
          ? Math.floor(roundedStepCount / 12)
          : Math.ceil(roundedStepCount / 12);
      if (roundedStepCount % 12 >= 3) octCount += 1;
      else if (roundedStepCount % 12 < -9) octCount -= 1;

      const centsRaw = 1200 * Math.log2(freqRatio);
      const fullCents =
        centsRaw > 0 ? Math.floor(centsRaw) : Math.ceil(centsRaw);
      let centAbsDiff = Math.abs(roundedStepCount * 100 - fullCents);
      let cents =
        roundedStepCount * 100 > fullCents ? -centAbsDiff : centAbsDiff;

      const note = this.NOTES_TO_A4[roundedStepCount % 12];
      const octave = 4 + octCount;
      this.pitch = `${note}${octave} ${cents > 0 ? '+' : ''}${cents}`;

      this.period = ((1 / this.frequency) * 1000).toFixed(2);
      this.wavelength = (this.SPEED_OF_SOUND / this.frequency).toFixed(2);
      this.samples = Math.ceil((1 / this.frequency) * this.SAMPLE_RATE);
    },
    parsePitch(e) {
      console.log(e);
    },
    parsePeriod(e) {
      console.log(e);
    },
    parseWavelength(e) {
      console.log(e);
    },
    parseSamples(e) {
      console.log(e);
    }
  }
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=PT+Serif&display=swap');

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html,
body,
#app,
main {
  height: 100%;
}

main {
  background-color: #333;
  color: rgb(255, 242, 215);
  font-family: 'PT Serif', sans-serif;
  font-size: 2.5rem;

  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

div.input-group {
  display: flex;
  align-items: center;
  justify-content: flex-start;
}

input {
  width: 15rem;
  margin-right: 1rem;

  background: none;
  color: rgb(255, 242, 215);
  font-family: 'PT Serif', sans-serif;
  font-size: 2.5rem;
  text-align: right;

  border: none;
  border-bottom: 3px solid rgb(255, 242, 215);
  outline: none;
}

input[type='number']::-webkit-inner-spin-button,
input[type='number']::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

input[type='number'] {
  -moz-appearance: textfield;
}
</style>
