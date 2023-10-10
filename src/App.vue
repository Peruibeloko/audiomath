<template>
  <main>
    <label class="label frequency">Frequency (Hz)</label>
    <input placeholder="0" id="freq" type="number" step="0.01" @input="freqHandler" v-model="frequency" />

    <label class="label pitch">Pitch</label>
    <input placeholder="A4 0" id="pitch" type="text" @input="pitchHandler" v-model="pitch" />

    <label class="label period">Period (ms)</label>
    <input placeholder="0" id="period" type="number" step="0.01" @input="periodHandler" v-model="period" />

    <label class="label wavelength">Wavelength (m)</label>
    <input placeholder="0" id="wavelength" type="number" step="0.01" @input="wavelengthHandler" v-model="wavelength" />

    <label class="label samples">Samples&nbsp;@&nbsp;<SampleRate @select="freqHandler" v-model="sampleRate" /></label>
    <input placeholder="0" id="samples" type="number" @input="samplesHandler" v-model="samples" />
  </main>
  <footer>
    <a href="https://github.com/Peruibeloko/audiomath" target="_blank">Source</a>
  </footer>
</template>

<script setup lang="ts">
import { ref } from 'vue';

import SampleRate from './SampleRate.vue';

type Pitch = [octave: number, semis: number, cents: number];

// Reference: https://github.com/audiojs/sample-rate/blob/master/readme.md
type SampleRate = 8000 | 11025 | 16000 | 22050 | 44100 | 48000 | 88200 | 96000 | 176400 | 192000 | 352800 | 384000;

// Constants
const SPEED_OF_SOUND = 345;
const REF_PITCH_A4 = 440;
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

// Reactive data
const frequency = ref(Infinity);
const pitch = ref('');
const period = ref(Infinity);
const samples = ref(Infinity);
const wavelength = ref(Infinity);
const sampleRate = ref<SampleRate>(44100);

// Conversion
const parsePitch = (pitch: string): Pitch => {
  const [noteWithOctave, strCents] = pitch.split(' ');
  const note = noteWithOctave.split(/-?\d+/)[0];
  const octave = Number(noteWithOctave.split(/[ABCDEFG]#?/)[1]);
  const cents = Number.isNaN(Number(strCents)) ? 0 : Number(strCents);

  const semis = -SCALE_DOWN.indexOf(note);

  return [octave, semis, cents];
};

const getNoteFromInterval = (semisFromA: number) => NOTES_FROM_A[semisFromA + 11];

const buildPitchString = (pitchData: Pitch) =>
  `${getNoteFromInterval(pitchData[1])}${pitchData[0]} ${pitchData[2] > 0 ? '+' : ''}${pitchData[2]}`;

const freqToPitch = (freq: number): Pitch => {
  const logFreqRatio = Math.log2(freq / REF_PITCH_A4);
  const absCentsDiff = Math.round(1200 * logFreqRatio);
  const absSemisDiff = Math.trunc(absCentsDiff / 100);

  const relativeCents = absCentsDiff - absSemisDiff * 100;
  const hasCentsOverflow = relativeCents >= 50;

  const cents = hasCentsOverflow ? -100 + relativeCents : relativeCents;

  const semis = (absSemisDiff + (hasCentsOverflow ? 1 : 0)) % 12;
  const hasSemisOverflow = semis < -9 || semis > 2;

  const octavesFromRef = Math.trunc(logFreqRatio);
  const octave = 4 + octavesFromRef + (hasSemisOverflow ? Math.sign(semis) : 0);

  return [octave, semis, cents];
};

const freqToPeriod = (freq: number) => Number(((1 / freq) * 1000).toFixed(2));
const freqToWavelength = (freq: number) => Number((SPEED_OF_SOUND / freq).toFixed(2));
const freqToSamples = (freq: number) => Math.ceil((1 / freq) * sampleRate.value);

const pitchToFreq = (pitchData: Pitch) => {
  const [oct, semis, cents] = pitchData;

  const octDiff = oct - 4;
  const semisDiff = semis + octDiff * 12;
  const centsDiff = cents + semisDiff * 100;

  return Number((Math.pow(2, centsDiff / 1200) * 440).toFixed(2));
};

const reset = () => {
  frequency.value = Infinity;
  pitch.value = '';
  period.value = Infinity;
  wavelength.value = Infinity;
  samples.value = Infinity;
};

// Event Handlers
const freqHandler = () => {
  if (!frequency.value) return reset();

  pitch.value = buildPitchString(freqToPitch(frequency.value));
  period.value = freqToPeriod(frequency.value);
  wavelength.value = freqToWavelength(frequency.value);
  samples.value = freqToSamples(frequency.value);
};

const pitchHandler = () => {
  if (!pitch.value) return reset();

  frequency.value = pitchToFreq(parsePitch(pitch.value));
  period.value = freqToPeriod(frequency.value);
  wavelength.value = freqToWavelength(frequency.value);
  samples.value = freqToSamples(frequency.value);
};

const periodHandler = () => {
  if (!period.value) return reset();

  frequency.value = Math.round(1000 / period.value);
  pitch.value = buildPitchString(freqToPitch(frequency.value));
  wavelength.value = freqToWavelength(frequency.value);
  samples.value = freqToSamples(frequency.value);
};

const wavelengthHandler = () => {
  if (!wavelength.value) return reset();

  frequency.value = Math.round(SPEED_OF_SOUND / wavelength.value);
  pitch.value = buildPitchString(freqToPitch(frequency.value));
  period.value = freqToPeriod(frequency.value);
  samples.value = freqToSamples(frequency.value);
};

const samplesHandler = () => {
  if (!samples.value) return reset();

  frequency.value = Math.round(sampleRate.value / samples.value);
  pitch.value = buildPitchString(freqToPitch(frequency.value));
  period.value = freqToPeriod(frequency.value);
  wavelength.value = freqToWavelength(frequency.value);
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=PT+Serif&display=swap');

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  background-color: #333;
  color: hsl(40, 100%, 92%);
}

html,
body,
#app {
  height: 100%;
}

#app {
  display: flex;
  flex-direction: column;
}

main {
  font-family: 'PT Serif', sans-serif;
  font-size: 2.5rem;

  display: grid;
  position: relative;
  margin: 0 auto;
  margin-top: auto;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
  max-width: 50rem;
}

label.label {
  align-self: center;
  justify-self: flex-end;
  display: flex;
  align-items: center;
}

input {
  width: 15rem;
  margin-right: 1rem;
  align-self: center;
  margin: auto;

  background: none;
  color: hsl(40, 100%, 92%);
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
  appearance: textfield;
}

footer {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 3rem;
  margin-bottom: auto;
}

footer a {
  color: hsl(0, 0%, 40%);
  text-decoration: none;
  border: 2px solid hsl(0, 0%, 40%);
  padding: 0.5rem;
  border-radius: 0.5rem;
  transition-duration: 200ms;
}

footer a:hover {
  color: hsl(40, 100%, 92%);
  border-color: hsl(40, 100%, 92%);
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
