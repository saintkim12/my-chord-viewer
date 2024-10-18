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
const removeStores = () => {
  stores.value = {}
  localStorage.removeItem('stores')
}
const isTargetKey = (v1: string, v2: string) => {
  return v1 && v2 && midi(v1) === midi(v2)
}
const hasTargetKey = (v: string) => {
  return pressedKeys.value.some((key) => isTargetKey(key, v))
}

const updateChordText = () => {
  const [note, suffix, chordType] = chordSelector.value
  let newNote = note ? `${note}${suffix ?? ''}` : chord.value.match(new RegExp(`(${dict.value.notes.concat(dict.value.notes.map((s) => s.toLowerCase())).join('|')})(${['#', 'b', ''].join('|')})`))?.[0]
  if (!newNote) return
  const newChord = `${newNote}${chordType}`
  chord.value = newChord
}

onMounted(() => {
  dict.value.chordTypes = all()
    .flatMap((o) => o.aliases)
    .filter((v) => v)
  dict.value.notes = names()
  try {
    stores.value = JSON.parse(localStorage.getItem('stores') ?? '{}')
  } catch (e: any) {
    window.alert(e.message)
  }
})
</script>
<template>
  <div style="height: -webkit-fill-available; overflow: hidden; display: flex; padding: 20px; gap: 16px">
    <div style="flex: 1 1 auto; height: 100%; outline: 1px solid red; display: flex; flex-direction: column; gap: 20px; overflow: hidden">
      <div style="flex: 0 0 135px; outline: 1px solid green">
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
      </div>
      <div style="flex: 0 0 73px; outline: 1px solid gray; display: flex; flex-direction: column; justify-content: center; gap: 8px">
        <div style="display: flex; align-items: center; justify-content: center">
          <span style="display: inline-flex; align-items: center">{{ notes.join(', ') }}</span>
        </div>
        <div style="display: flex; align-items: center; justify-content: center; position: relative">
          <span style="display: inline-flex; align-items: center" @click.prevent="convertChordAndNotes">{{ chord || '&nbsp;' }}</span>
          <span v-show="!!chord" style="display: inline-flex; align-items: center; cursor: pointer; position: absolute; right: 50px; top: 0" @click.prevent=";[(chord = ''), (notes = []), (pressedKeys = []), (chordSelector = ['', '', ''])]"
            ><svg width="20" height="20" viewBox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path
                d="M3.59236 0L0 3.59236L1.8344 5.42675L6.36943 10.0382L1.8344 14.5732L0 16.3312L3.59236 20L5.42675 18.1656L10.0382 13.5541L14.5732 18.1656L16.3312 20L20 16.3312L18.1656 14.5732L13.5541 10.0382L18.1656 5.42675L20 3.59236L16.3312 0L14.5732 1.8344L10.0382 6.36943L5.42675 1.8344L3.59236 0Z"
                fill="#888"
              />
            </svg>
          </span>
        </div>
      </div>
      <div style="flex: 1 1 auto; outline: 1px solid purple; display: flex; align-items: center; justify-content: center; overflow: hidden">
        <div style="width: 26px; height: 100%; display: flex; flex-direction: column; justify-content: safe center; align-items: center; overflow: hidden auto; scrollbar-width: thin">
          <div
            v-for="item in dict.notes"
            :key="item"
            style="padding: 10px 0; width: 100%; text-align: center"
            :style="item === chordSelector[0] && { 'background-color': '#eee' }"
            @click=";[(chordSelector[0] = item === chordSelector[0] ? '' : item), updateChordText()]"
          >
            {{ item }}
          </div>
        </div>
        <div style="width: 47px; height: 100%; display: flex; flex-direction: column; justify-content: safe center; align-items: center; overflow: hidden auto; scrollbar-width: thin">
          <div
            v-for="item in ['', '#', 'b']"
            :key="item"
            style="padding: 10px 0; width: 100%; text-align: center"
            :style="item === chordSelector[1] && { 'background-color': '#eee' }"
            @click=";[(chordSelector[1] = item === chordSelector[1] ? '' : item), updateChordText()]"
          >
            {{ item || '(X)' }}
          </div>
        </div>
        <div style="height: 100%; display: flex; flex-direction: column; justify-content: safe center; align-items: center; overflow: hidden auto; scrollbar-width: thin">
          <div
            v-for="item in dict.chordTypes"
            :key="item"
            style="padding: 10px 0; width: 100%; text-align: center"
            :style="item === chordSelector[2] && { 'background-color': '#eee' }"
            @click=";[(chordSelector[2] = item === chordSelector[2] ? '' : item), updateChordText()]"
          >
            {{ item }}
          </div>
        </div>
      </div>
    </div>
    <div style="flex: 0 0 48px; height: 100%; outline: 1px solid blue; display: flex; flex-direction: column; justify-content: flex-start; align-items: center; overflow: hidden auto; scrollbar-width: thin">
      <div @click.prevent="removeStores()">
        <svg width="20" height="20" viewBox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path
            d="M3.59236 0L0 3.59236L1.8344 5.42675L6.36943 10.0382L1.8344 14.5732L0 16.3312L3.59236 20L5.42675 18.1656L10.0382 13.5541L14.5732 18.1656L16.3312 20L20 16.3312L18.1656 14.5732L13.5541 10.0382L18.1656 5.42675L20 3.59236L16.3312 0L14.5732 1.8344L10.0382 6.36943L5.42675 1.8344L3.59236 0Z"
            fill="#888"
          />
        </svg>
      </div>
      <div>
        <svg width="16" height="8" viewBox="0 0 16 8" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M8 0L15.7942 7.5H0.205771L8 0Z" fill="#D9D9D9" />
        </svg>
      </div>
      <div v-for="key in Object.keys(stores)" :key="key" style="padding: 10px 0" @click="convertFromStores(key)">{{ key }}</div>
      <div>
        <svg width="16" height="8" viewBox="0 0 16 8" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M8 8L15.7942 0.5H0.205771L8 8Z" fill="#D9D9D9" />
        </svg>
      </div>
    </div>
  </div>
  <!-- <div style="display: flex; flex-direction: column; height: 100%; gap: 8px; text-align: center"> -->
    <!-- <div> -->
      <!-- <div class="piano-container"> -->
        <!-- <ul class="piano-keys-list"> -->
          
          <!-- <li class="piano-keys white-key" :class="[hasTargetKey('C' + '' + '4') && 'pressed']"></li> -->
          <!-- <li class="piano-keys black-key" :class="[hasTargetKey('C' + '#' + '4') && 'pressed']"></li> -->
          <!-- <li class="piano-keys white-key" :class="[hasTargetKey('D' + '' + '4') && 'pressed']"></li> -->
          <!-- <li class="piano-keys black-key" :class="[hasTargetKey('D' + '#' + '4') && 'pressed']"></li> -->
          <!-- <li class="piano-keys white-key" :class="[hasTargetKey('E' + '' + '4') && 'pressed']"></li> -->
          
          <!-- <li class="piano-keys white-key" :class="[hasTargetKey('F' + '' + '4') && 'pressed']"></li> -->
          <!-- <li class="piano-keys black-key" :class="[hasTargetKey('F' + '#' + '4') && 'pressed']"></li> -->
          <!-- <li class="piano-keys white-key" :class="[hasTargetKey('G' + '' + '4') && 'pressed']"></li> -->
          <!-- <li class="piano-keys black-key" :class="[hasTargetKey('G' + '#' + '4') && 'pressed']"></li> -->
          <!-- <li class="piano-keys white-key" :class="[hasTargetKey('A' + '' + '4') && 'pressed']"></li> -->
          <!-- <li class="piano-keys black-key" :class="[hasTargetKey('A' + '#' + '4') && 'pressed']"></li> -->
          <!-- <li class="piano-keys white-key" :class="[hasTargetKey('B' + '' + '4') && 'pressed']"></li> -->
          
          <!-- <li class="piano-keys white-key" :class="[hasTargetKey('C' + '' + '5') && 'pressed']"></li> -->
          <!-- <li class="piano-keys black-key" :class="[hasTargetKey('C' + '#' + '5') && 'pressed']"></li> -->
          <!-- <li class="piano-keys white-key" :class="[hasTargetKey('D' + '' + '5') && 'pressed']"></li> -->
          <!-- <li class="piano-keys black-key" :class="[hasTargetKey('D' + '#' + '5') && 'pressed']"></li> -->
          <!-- <li class="piano-keys white-key" :class="[hasTargetKey('E' + '' + '5') && 'pressed']"></li> -->
          
          <!-- <li class="piano-keys white-key" :class="[hasTargetKey('F' + '' + '5') && 'pressed']"></li> -->
          <!-- <li class="piano-keys black-key" :class="[hasTargetKey('F' + '#' + '5') && 'pressed']"></li> -->
          <!-- <li class="piano-keys white-key" :class="[hasTargetKey('G' + '' + '5') && 'pressed']"></li> -->
          <!-- <li class="piano-keys black-key" :class="[hasTargetKey('G' + '#' + '5') && 'pressed']"></li> -->
          <!-- <li class="piano-keys white-key" :class="[hasTargetKey('A' + '' + '5') && 'pressed']"></li> -->
          <!-- <li class="piano-keys black-key" :class="[hasTargetKey('A' + '#' + '5') && 'pressed']"></li> -->
          <!-- <li class="piano-keys white-key" :class="[hasTargetKey('B' + '' + '5') && 'pressed']"></li> -->
        <!-- </ul> -->
      <!-- </div> -->
      <!-- <div>Notes: <input type="text" :value="notes.join(', ')" /></div> -->
      <!-- <hr /> -->
    <!-- </div> -->
    <!-- <div> -->
      <!-- <div>Chord: <input type="text" v-model="chord" /> <button @click.prevent="convertChordAndNotes">Get</button></div> -->
      <!-- <div> -->
        <!-- <select v-model="chordSelector[0]" @change="updateChordText()"> -->
          <!-- <option></option> -->
          <!-- <option v-for="note in dict.notes" :key="note" :value="note">{{ note }}</option> -->
        <!-- </select> -->
        <!-- <select v-model="chordSelector[1]" @change="updateChordText()"> -->
          <!-- <option></option> -->
          <!-- <option v-for="item in ['#', 'b']" :key="item" :value="item">{{ item }}</option> -->
        <!-- </select> -->
        <!-- <select v-model="chordSelector[2]" @change="updateChordText()"> -->
          <!-- <option></option> -->
          <!-- <option v-for="item in dict.chordTypes" :key="item" :value="item">{{ item }}</option> -->
        <!-- </select> -->
        <!-- <button @click.prevent="convertChordAndNotes">Get</button> -->
      <!-- </div> -->
      <!-- <hr /> -->
    <!-- </div> -->
    <!-- <div> -->
      <!-- inputs -->
      <!-- <select multiple style="width: 200px" @input="convertFromStores(($event.target as HTMLSelectElement).value)"> -->
        <!-- <option v-for="key in Object.keys(stores)" :key="key">{{ key }}</option> -->
      <!-- </select> -->
    <!-- </div> -->
  <!-- </div> -->
</template>

<style scoped lang="scss">
ul {
  padding: 0;
}

// $key-width: calc(100% / 14);
// $key-width: 16px;
$key-width: 11px;
$key-white-height: 200px;
$key-black-height: 120px;

.piano-container {
  display: flex;
  align-items: center;
  justify-content: safe center;
  // justify-content: flex-start;

  width: 100%;
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
  border-bottom-right-radius: 0;
  border-bottom-left-radius: 0;
  // &:nth-of-type(12n + 1),
  // &:nth-of-type(12n + 6) {
  //   margin-left: -1px;
  // }
  &:nth-of-type(12n + 1) {
    border-bottom-left-radius: 10px;
  }
  &:nth-of-type(12n) {
    border-bottom-right-radius: 10px;
  }
  &:nth-of-type(12n + 5) {
    border-top-right-radius: 0;
  }
  &:nth-of-type(12n + 6) {
    border-top-left-radius: 0;
  }
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
