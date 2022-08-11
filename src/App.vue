<template>
  <main>
    <label class="label frequency">Frequency (Hz)</label>
    <input placeholder="0" id="freq" type="number" @input="freqHandler" v-model="frequency" />

    <label class="label note">Note</label>
    <input placeholder="A4 0" id="note" type="text" @input="pitchHandler" v-model="pitch" />

    <label class="label period">Period (ms)</label>
    <input placeholder="0" id="period" type="number" @input="periodHandler" v-model="period" />

    <label class="label wavelength">Wavelength (m)</label>
    <input placeholder="0" id="wavelength" type="number" @input="wavelengthHandler" v-model="wavelength" />

    <label class="label samples">Samples @ 44.1kHz</label>
    <input placeholder="0" id="samples" type="number" @input="samplesHandler" v-model="samples" />
  </main>
</template>

<script setup lang="ts">
import { ref } from 'vue';

// Constants
const SPEED_OF_SOUND = 345;
const SAMPLE_RATE = 44100;
const REF_PITCH_A4 = 440;
const SCALE_UP = ['A', 'A#', 'B', 'C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#'];
const SCALE_DOWN = ['A', 'G#', 'G', 'F#', 'F', 'E', 'D#', 'D', 'C#', 'C', 'B', 'A#'];
const NOTES_FROM_A = [
  'A#',
  'B',
  'C',
  'C#',
  'D',
  'D#',
  'E',
  'F',
  'F#',
  'G',
  'G#',
  'A',
  'A#',
  'B',
  'C',
  'C#',
  'D',
  'D#',
  'E',
  'F',
  'F#',
  'G',
  'G#'
];
const getNoteFromInterval = (semisFromA: number) => NOTES_FROM_A[semisFromA + 11];

// Reactive data
const frequency = ref(Infinity);
const pitch = ref('');
const period = ref(Infinity);
const samples = ref(Infinity);
const wavelength = ref(Infinity);

// Conversion
const parsePitch = (pitch: string) => {
  const [noteWithOctave, strCents] = pitch.split(' ');
  const cents = Number(strCents);
  const octave = Number(noteWithOctave.split(/[ABCDEFG]#?/)[1]);
  const note = noteWithOctave.split(String(octave))[0];

  const semis = octave > 4 ? SCALE_UP.indexOf(note) : -SCALE_DOWN.indexOf(note);

  return [octave, semis, cents];
};

const freqToPitch = (freq: number) => {
  const logFreqRatio = Math.log2(freq / REF_PITCH_A4);

  const absCentsDiff = Math.round((1200 * logFreqRatio) % 100);
  const hasCentsOverflow = absCentsDiff < -50 || absCentsDiff > 50;

  const cents = hasCentsOverflow ? 100 + absCentsDiff * Math.sign(absCentsDiff) * -1 : absCentsDiff;

  const absSemisDiff = Math.trunc(12 * logFreqRatio) + (hasCentsOverflow ? Math.sign(absCentsDiff) : 0);

  const semis = absSemisDiff % 12;
  const hasSemisOverflow = semis < -9 || semis > 2;

  const octavesFromRef = Math.trunc(logFreqRatio);
  const octave = 4 + octavesFromRef + (hasSemisOverflow ? Math.sign(semis) : 0);

  return [octave, semis, cents];
};
const freqToPeriod = (freq: number) => Number(((1 / freq) * 1000).toFixed(2));
const freqToWavelength = (freq: number) => Number((SPEED_OF_SOUND / freq).toFixed(2));
const freqToSamples = (freq: number) => Math.ceil((1 / freq) * SAMPLE_RATE);

const pitchToFreq = (pitchData: number[]) => {
  const [oct, semis, cents] = pitchData;

  const octDiff = oct - 4;
  const semisDiff = semis + octDiff * 12;
  const centsDiff = cents + semisDiff * 100;

  return Math.round(Math.pow(2, centsDiff / 1200) * 440);
};

// Event Handlers
const freqHandler = () => {
  const [oct, semis, cents] = freqToPitch(frequency.value);
  pitch.value = `${getNoteFromInterval(semis)}${oct} ${cents > 0 ? '+' : ''}${cents}`;
  period.value = freqToPeriod(frequency.value);
  wavelength.value = freqToWavelength(frequency.value);
  samples.value = freqToSamples(frequency.value);
};

const pitchHandler = () => {
  const [oct, semis, cents] = parsePitch(pitch.value);
  frequency.value = pitchToFreq([oct, semis, cents]);
  period.value = freqToPeriod(frequency.value);
  wavelength.value = freqToWavelength(frequency.value);
  samples.value = freqToSamples(frequency.value);
};

const periodHandler = () => {};

const wavelengthHandler = () => {};

const samplesHandler = () => {};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=PT+Serif&display=swap');

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  background-color: #333;
  color: rgb(255, 242, 215);
}

html,
body,
#app {
  height: 100%;
}

#app {
  display: flex;
}

main {
  font-family: 'PT Serif', sans-serif;
  font-size: 2.5rem;

  display: grid;
  position: relative;
  margin: auto;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
  max-width: 50rem;
}

label.label {
  align-self: center;
  justify-self: flex-end;
}

input {
  width: 15rem;
  margin-right: 1rem;
  align-self: center;
  margin: auto;

  background: none;
  color: rgb(255, 242, 215);
  font-family: 'PT Serif', sans-serif;
  font-size: 2.5rem;
  text-align: left;

  border: none;
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

@media (max-width: 480px) {
  main {
    width: 100%;
  }

  input {
    width: 5rem;
    font-size: 1.5rem;
  }

  label {
    font-size: 1.5rem;
  }
}
</style>
