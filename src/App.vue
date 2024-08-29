<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { get } from '@tonaljs/chord'
const chord = ref('')
const notes = ref('')
const stores = ref({} as Record<string, ReturnType<typeof get>>)

const convertChordAndNotes = () => {
  if (chord.value) {
    const chordInfo = stores.value[chord.value] ? stores.value[chord.value]! : get(chord.value)
    if (!chordInfo.empty) {
      stores.value[chord.value] = chordInfo
      localStorage.setItem('stores', JSON.stringify(stores.value))
      notes.value = chordInfo.notes.join(', ')
    }
  }
}
const convertFromStores = (value: string) => {
  chord.value = value
  convertChordAndNotes()
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
  <div style="display: flex; flex-direction: column; gap: 8px;">
    <div>
      Chord: <input type="text" v-model="chord" />
    </div>
    <div>
      <button @click.prevent="convertChordAndNotes">Get</button>
    </div>
    <div>
      Notes: <input type="text" v-model="notes" />
    </div>
    <div>
      inputs
      <select multiple style="width: 200px;" @input="convertFromStores(($event.target as HTMLSelectElement).value)">
        <option v-for="key in Object.keys(stores)" :key="key">{{ key }}</option>
      </select>
    </div>
  </div>
</template>

<style scoped></style>
