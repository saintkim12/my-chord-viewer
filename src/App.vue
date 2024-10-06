<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { midi, names } from '@tonaljs/note'
import { all } from '@tonaljs/chord-type'
import { get, degrees } from '@tonaljs/chord'
const chord = ref('')
const notes = ref([] as string[])
const stores = ref({} as Record<string, ReturnType<typeof get>>)
const pressedKeys = ref([] as string[])
const chordSelector = ref(['', '', ''])
const dict = ref({
  notes: new Array<string>(),
  chordTypes: new Array<string>(),
})

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

const updateChordText = (() => {
  const [note, suffix, chordType] = chordSelector.value
  let newNote = note ? `${note}${suffix ?? ''}` : chord.value.match(
    new RegExp(`(${dict.value.notes.concat(dict.value.notes.map(s => s.toLowerCase())).join('|')})(${['#', 'b', ''].join('|')})`)
  )?.[0]
  if (!newNote) return;
  const newChord = `${newNote}${chordType}`
  chord.value = newChord;
})

onMounted(() => {
  dict.value.chordTypes = all().flatMap(o => o.aliases).filter(v => v);
  dict.value.notes = names();
  try {
    stores.value = JSON.parse(localStorage.getItem('stores') ?? '{}')
  } catch (e: any) {
    window.alert(e.message)
  }
})
</script>
<template>
  <div style="display: flex; flex-direction: column; height: 100%; gap: 8px; text-align: center;">
    <div>
      <div class="piano-container">
        <ul class="piano-keys-list">
          <!--  -->
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
      <div>
        Notes: <input type="text" :value="notes.join(', ')" />
      </div>
      <hr />
    </div>
    <div>
      <div>
        Chord: <input type="text" v-model="chord" /> <button @click.prevent="convertChordAndNotes">Get</button>
      </div>
      <div>
        <select v-model="chordSelector[0]" @change="updateChordText()">
          <option></option>
          <option v-for="note in dict.notes" :key="note" :value="note">{{ note }}</option>
        </select>
        <select v-model="chordSelector[1]" @change="updateChordText()">
          <option></option>
          <option v-for="item in ['#', 'b']" :key="item" :value="item">{{ item }}</option>
        </select>
        <select v-model="chordSelector[2]" @change="updateChordText()">
          <option></option>
          <option v-for="item in dict.chordTypes" :key="item" :value="item">{{ item }}</option>
        </select>
        <button @click.prevent="convertChordAndNotes">Get</button>
      </div>
      <hr />
    </div>
    <div>
      inputs
      <select multiple style="width: 200px;" @input="convertFromStores(($event.target as HTMLSelectElement).value)">
        <option v-for="key in Object.keys(stores)" :key="key">{{ key }}</option>
      </select>
    </div>
  </div>
</template>

<style scoped lang="scss">
ul {
  padding: 0;
}

// $key-width: calc(100% / 14);
$key-width: 16px;
$key-white-height: 200px;
$key-black-height: 120px;

.piano-container {
  display: flex;
  align-items: center;
  justify-content: safe center;
  // justify-content: flex-start;
  
  width: 100vw;
  overflow: auto;

  .piano-keys-list {
    // width: 100%;
  }
}

.piano-keys-list {
  list-style: none;
  display: flex;
  justify-content: center;
}

.piano-keys {
  // width: 5rem;
  width: $key-width * 1.5;
  cursor: pointer;
  // position: relative;
  height: 20rem;
  border-radius: 10px;
  border: 1px solid #000;
}

.white-key {
  height: $key-white-height;
  background-color: #fff;
  & + .white-key {
    margin-left: -1px;
  }
  // &:nth-of-type(12n + 1), &:nth-of-type(12n + 6) {
  //     margin-left: -1px;
  // }
}

.black-key {
  // width: 2rem;
  // height: 13rem;
  width: $key-width;
  height: $key-black-height;
  
  border-radius: 5px;
  border-top-left-radius: 0;
  border-top-right-radius: 0;
  background-color: #000;
  z-index: 2;
  // margin: 0 -20px 0 -20px;
  margin: 0 calc(-#{$key-width} / 2 - 0.1rem);
}

.white-key.pressed,
.black-key.pressed {
  background-color: lightblue;
}

// @media screen and (min-width: 821px) and (max-width: 1024px) {
//   .piano-keys {
//     width: 4rem;
//     cursor: pointer;
//     position: relative;
//     height: 20rem;
//     border: 1px solid #000;
//   }

//   .black-key {
//     width: 2rem;
//     height: 13rem;
//     border-radius: 5px;
//     border-top-left-radius: 0;
//     border-top-right-radius: 0;
//     background-color: #000;
//     z-index: 2;
//     margin: 0 -20px 0 -20px;
//   }
// }

// @media screen and (min-width: 768px) and (max-width: 820px) {
//   .piano-keys {
//     width: 3.5rem;
//     cursor: pointer;
//     position: relative;
//     height: 16rem;
//     border: 1px solid #000;
//   }

//   .black-key {
//     width: 1.8rem;
//     height: 10rem;
//     border-radius: 5px;
//     border-top-left-radius: 0;
//     border-top-right-radius: 0;
//     background-color: #000;
//     z-index: 2;
//     margin: 0 -20px 0 -20px;
//   }
// }

// @media screen and (max-width: 576px) {
//   $key-width: 16px;
//   .piano-container {
//     width: 100vw;
//     display: flex;
//     align-items: center;
//     // justify-content: flex-start;
//     overflow: auto;

//     .piano-keys-list{
//       width: 100%;
//     }
//   }

//   .piano-keys {
//     width: $key-width * 1.5;
//     cursor: pointer;
//     height: 16rem;
//     border: 1px solid #000;
//   }

//   .black-key {
//     width: $key-width;
//     height: 10rem;
//     background-color: #000;
//     z-index: 2;
//     // margin: 0 calc(-1rem / 2 - 0.1rem);
//     margin: 0 calc(-#{$key-width} / 2 - 0.1rem);
//   }
// }
</style>
