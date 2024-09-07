<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { midi } from '@tonaljs/note'
import { get, degrees } from '@tonaljs/chord'
const chord = ref('')
const notes = ref([] as string[])
const stores = ref({} as Record<string, ReturnType<typeof get>>)
const pressedKeys = ref([] as string[])

const convertChordAndNotes = () => {
  pressedKeys.value = []
  if (chord.value) {
    const chordInfo = stores.value[chord.value] ? stores.value[chord.value]! : get(chord.value)
    if (!chordInfo.empty) {
      stores.value[chord.value] = chordInfo
      localStorage.setItem('stores', JSON.stringify(stores.value))
      notes.value = chordInfo.notes

      const pressedData = [...Array(chordInfo.notes.length).keys()].map((_, i) => i + 1).map(degrees(chordInfo.aliases[0], `${chordInfo.notes[0]}4`))
      pressedKeys.value = pressedData
    }
  }
}
const convertFromStores = (value: string) => {
  chord.value = value
  convertChordAndNotes()
}
const isTargetKey = (v1: string, v2: string) => {
  return v1 && v2 && midi(v1) === midi(v2)
}
const hasTargetKey = (v: string) => {
  return pressedKeys.value.some(key => isTargetKey(key, v))
}

onMounted(() => {
  try {
    stores.value = JSON.parse(localStorage.getItem('stores') ?? '{}')
  } catch (e: any) {
    window.alert(e.message)
  }
})
</script>
<template>
  <div style="display: flex; flex-direction: column; justify-content: center; height: 100%; gap: 8px; text-align: center;">
    <div>
      Chord: <input type="text" v-model="chord" />
    </div>
    <div>
      <button @click.prevent="convertChordAndNotes">Get</button>
    </div>
    <div>
      inputs
      <select multiple style="width: 200px;" @input="convertFromStores(($event.target as HTMLSelectElement).value)">
        <option v-for="key in Object.keys(stores)" :key="key">{{ key }}</option>
      </select>
    </div>
    <div>
      <div>
        Notes: <input type="text" :value="notes.join(', ')" />
      </div>
      <div class="piano-container">
        <ul class="piano-keys-list">
          <li class="piano-keys white-key" :class="[hasTargetKey('C' + '' + '4') && 'pressed']"></li>
          <li class="piano-keys black-key" :class="[hasTargetKey('C' + '#' + '4') && 'pressed']"></li>
          <li class="piano-keys white-key" :class="[hasTargetKey('D' + '' + '4') && 'pressed']"></li>
          <li class="piano-keys black-key" :class="[hasTargetKey('D' + '#' + '4') && 'pressed']"></li>
          <li class="piano-keys white-key" :class="[hasTargetKey('E' + '' + '4') && 'pressed']"></li>
          <!--  -->
          <li class="piano-keys white-key" :class="[hasTargetKey('F' + '' + '4') && 'pressed']"></li>
          <li class="piano-keys black-key" :class="[hasTargetKey('F' + '#' + '4') && 'pressed']"></li>
          <li class="piano-keys white-key" :class="[hasTargetKey('G' + '' + '4') && 'pressed']"></li>
          <li class="piano-keys black-key" :class="[hasTargetKey('G' + '#' + '4') && 'pressed']"></li>
          <li class="piano-keys white-key" :class="[hasTargetKey('A' + '' + '4') && 'pressed']"></li>
          <li class="piano-keys black-key" :class="[hasTargetKey('A' + '#' + '4') && 'pressed']"></li>
          <li class="piano-keys white-key" :class="[hasTargetKey('B' + '' + '4') && 'pressed']"></li>
          <!--  -->
          <li class="piano-keys white-key" :class="[hasTargetKey('C' + '' + '5') && 'pressed']"></li>
          <li class="piano-keys black-key" :class="[hasTargetKey('C' + '#' + '5') && 'pressed']"></li>
          <li class="piano-keys white-key" :class="[hasTargetKey('D' + '' + '5') && 'pressed']"></li>
          <li class="piano-keys black-key" :class="[hasTargetKey('D' + '#' + '5') && 'pressed']"></li>
          <li class="piano-keys white-key" :class="[hasTargetKey('E' + '' + '5') && 'pressed']"></li>
          <!--  -->
          <li class="piano-keys white-key" :class="[hasTargetKey('F' + '' + '5') && 'pressed']"></li>
          <li class="piano-keys black-key" :class="[hasTargetKey('F' + '#' + '5') && 'pressed']"></li>
          <li class="piano-keys white-key" :class="[hasTargetKey('G' + '' + '5') && 'pressed']"></li>
          <li class="piano-keys black-key" :class="[hasTargetKey('G' + '#' + '5') && 'pressed']"></li>
          <li class="piano-keys white-key" :class="[hasTargetKey('A' + '' + '5') && 'pressed']"></li>
          <li class="piano-keys black-key" :class="[hasTargetKey('A' + '#' + '5') && 'pressed']"></li>
          <li class="piano-keys white-key" :class="[hasTargetKey('B' + '' + '5') && 'pressed']"></li>
        </ul>
      </div>
    </div>
  </div>
</template>

<style scoped>
ul {
  padding: 0;
}

.piano-container {
  display: flex;
  align-items: center;
  justify-content: center;
}

.piano-keys-list {
  list-style: none;
  display: flex;
  justify-content: center;
}

.piano-keys {
  width: 5rem;
  cursor: pointer;
  position: relative;
  height: 20rem;
  border-radius: 10px;
  border: 1px solid #000;
}

.white-key {
  background-color: #fff;
}

.black-key {
  width: 2rem;
  height: 13rem;
  border-radius: 5px;
  border-top-left-radius: 0;
  border-top-right-radius: 0;
  background-color: #000;
  z-index: 2;
  margin: 0 -20px 0 -20px;
}

.white-key.pressed,
.black-key.pressed {
  background-color: lightblue;
}

@media screen and (min-width: 821px) and (max-width: 1024px) {
  .piano-keys {
    width: 4rem;
    cursor: pointer;
    position: relative;
    height: 20rem;
    border-radius: 10px;
    border: 1px solid #000;
  }

  .black-key {
    width: 2rem;
    height: 13rem;
    border-radius: 5px;
    border-top-left-radius: 0;
    border-top-right-radius: 0;
    background-color: #000;
    z-index: 2;
    margin: 0 -20px 0 -20px;
  }
}

@media screen and (min-width: 768px) and (max-width: 820px) {
  .piano-keys {
    width: 3.5rem;
    cursor: pointer;
    position: relative;
    height: 16rem;
    border-radius: 10px;
    border: 1px solid #000;
  }

  .black-key {
    width: 1.8rem;
    height: 10rem;
    border-radius: 5px;
    border-top-left-radius: 0;
    border-top-right-radius: 0;
    background-color: #000;
    z-index: 2;
    margin: 0 -20px 0 -20px;
  }
}

@media screen and (max-width: 576px) {
  .piano-container {
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: flex-start;
    overflow: auto;
  }

  .piano-keys {
    width: 3.5rem;
    cursor: pointer;
    position: relative;
    height: 16rem;
    border-radius: 10px;
    border: 1px solid #000;
  }

  .black-key {
    width: 1.8rem;
    height: 10rem;
    border-radius: 5px;
    border-top-left-radius: 0;
    border-top-right-radius: 0;
    background-color: #000;
    z-index: 2;
    margin: 0 -20px 0 -20px;
  }
}
</style>
