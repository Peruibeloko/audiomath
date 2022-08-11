<template>
  <main>
    <label class="label frequency">Frequency (Hz)</label>
    <input
      placeholder="0"
      id="freq"
      type="number"
      @input="freqHandler"
      v-model="frequency"
    />

    <label class="label note">Note</label>
    <input
      placeholder="A4 0"
      id="note"
      type="text"
      @input="pitchHandler"
      v-model="pitch"
    />

    <label class="label period">Period (ms)</label>
    <input
      placeholder="0"
      id="period"
      type="number"
      @input="periodHandler"
      v-model="period"
    />

    <label class="label wavelength">Wavelength (m)</label>
    <input
      placeholder="0"
      id="wavelength"
      type="number"
      @input="wavelengthHandler"
      v-model="wavelength"
    />

    <label class="label samples">Samples @ 44.1kHz</label>
    <input
      placeholder="0"
      id="samples"
      type="number"
      @input="samplesHandler"
      v-model="samples"
    />
  </main>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const frequency = ref(Infinity);
const pitch = ref('');
const period = ref(Infinity);
const samples = ref(Infinity);
const wavelength = ref(Infinity);

const SPEED_OF_SOUND = 345;
const SAMPLE_RATE = 44100;
const REF_PITCH_A4 = 440;
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
const SCALE_UP = [
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
const SCALE_DOWN = [
  'A',
  'G#',
  'G',
  'F#',
  'F',
  'E',
  'D#',
  'D',
  'C#',
  'C',
  'B',
  'A#'
];

const parsePitch = (pitch: string) => {
  const [noteWithOctave, strCents] = pitch.split(' ');
  const cents = Number(strCents);
  const octave = Number(noteWithOctave.split(/[ABCDEFG]#?/)[1]);
  const note = noteWithOctave.split(String(octave))[0];

  const semisFromA =
    octave >= 4 ? SCALE_UP.indexOf(note) : SCALE_DOWN.indexOf(note);
};

const freqToPitch = (freq: number) => {
  const logFreqRatio = Math.log2(freq / REF_PITCH_A4);

  const unboundCents = Math.round((1200 * logFreqRatio) % 100);
  const hasCentsOverflow = unboundCents < -50 || unboundCents > 50;

  const cents = hasCentsOverflow
    ? 100 + unboundCents * Math.sign(unboundCents) * -1
    : unboundCents;

  const semisFromA4 =
    Math.trunc(12 * logFreqRatio) +
    (hasCentsOverflow ? Math.sign(unboundCents) : 0);

  const semisFromA = semisFromA4 % 12;
  const note = NOTES_FROM_A[semisFromA + 11];
  const hasSemisOverflow = semisFromA < -9 || semisFromA > 2;

  const octavesFromRef = Math.trunc(logFreqRatio);
  const octave =
    4 + octavesFromRef + (hasSemisOverflow ? Math.sign(semisFromA) : 0);

  return [octave, note, cents];
};

const freqHandler = () => {
  const [oct, note, cents] = freqToPitch(frequency.value);
  pitch.value = `${note}${oct} ${cents > 0 ? '+' : ''}${cents}`;
  period.value = Number(((1 / frequency.value) * 1000).toFixed(2));
  wavelength.value = Number((SPEED_OF_SOUND / frequency.value).toFixed(2));
  samples.value = Math.ceil((1 / frequency.value) * SAMPLE_RATE);
};

const pitchHandler = () => {
  parsePitch(pitch.value);
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
