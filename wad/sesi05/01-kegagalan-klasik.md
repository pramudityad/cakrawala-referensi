# Nomor 1 — lima kegagalan klasik port Vue

Pilih **satu** yang benar-benar kamu alami. Tiap potongan berisi: kode salah → gejala → akar → versi benar.

---

## (a) Lupa `.value` di `<script setup>`

**Salah**
```js
// ItemGrid.vue
const items = ref([])
items.push({ id: 1, nama: 'Kopi Susu' })   // !!
console.log(items.length)                  // !!
```

**Gejala** — `Uncaught TypeError: items.push is not a function` di console; `console.log` mencetak
`undefined`; template `{{ items.length }}` malah benar sehingga terlihat "setengah jalan".

**Akar** — `ref()` membungkus nilainya jadi objek `{ value }`. Di `<script setup>` kamu menyentuh
pembungkusnya, di `<template>` Vue membuka bungkusnya otomatis (unwrapping).

**Benar**
```js
items.value.push({ id: 1, nama: 'Kopi Susu' })
console.log(items.value.length)
```
```vue
<template>
  <span>{{ items.length }}</span>   <!-- tanpa .value di template -->
</template>
```

---

## (b) Mengubah `props` langsung, atau destructure `reactive()`

**Salah**
```js
// FilterBar.vue
const props = defineProps({ filter: String })
props.filter = 'kopi'            // !!

const { items } = reactive({ items: [], loading: false })   // !!
items.push({ id: 1 })            // tidak memicu render
```

**Gejala** — console: `Set operation on key "filter" failed: target is readonly.`; atau UI diam saja
padahal datanya sudah berubah (bug yang paling menjengkelkan: tidak ada error sama sekali).

**Akar** — props itu satu arah dan read-only; anak tidak boleh menulisnya. Dan `destructure` melepas
proxy reaktivitas: variabel hasil destructure adalah nilai biasa, bukan lagi yang dipantau Vue.

**Benar**
```js
// FilterBar.vue — anak minta induk yang mengubah
const props = defineProps({ filter: String })
const emit = defineEmits(['update:filter'])
function onInput(e) { emit('update:filter', e.target.value) }
```
```js
// jangan destructure reactive → pakai ref (disarankan di mata kuliah ini)
const items = ref([])
```
```vue
<!-- App.vue -->
<FilterBar :filter="filter" @update:filter="filter = $event" />
```

---

## (c) `defineEmits` belum dideklarasikan

**Salah**
```js
// FilterBar.vue
const props = defineProps({ filter: String })
// tidak ada defineEmits
function onInput(e) { emit('update:filter', e.target.value) }   // !!
```

**Gejala** — `ReferenceError: emit is not defined` saat mengetik di kotak filter. Kadang lebih sunyi:
`defineEmits` ada tapi induk tidak mendengarkan, jadi filter tidak pernah berubah dan grid tidak
pernah tersaring — tidak ada pesan error sama sekali.

**Akar** — di `<script setup>`, `emit` bukan variabel ajaib; ia harus diambil dari `defineEmits`.
Dan komunikasi ke atas butuh dua sisi: anak mengirim, induk mendengarkan.

**Benar**
```js
const emit = defineEmits(['update:filter'])
function onInput(e) { emit('update:filter', e.target.value) }
```
```vue
<FilterBar :filter="filter" @update:filter="filter = $event" />
```

---

## (d) `watch` tanpa cleanup — race condition Sesi 4 kembali

**Salah**
```js
watch(filter, async (newVal) => {
  items.value = await fetchItems(newVal)      // !!
})
```

**Gejala** — filter cepat diketik → daftar berkedip dan berakhir menampilkan hasil filter **lama**.
Tidak ada error; hasilnya kadang benar, kadang salah. Ini bug yang hilang sendiri saat di-demo.

**Akar** — dua request berjalan bersamaan; yang **lambat** menang, bukan yang terakhir dipanggil.
Di Sesi 4 kamu menyelesaikan ini dengan `AbortController`; di Vue ia punya rumah resmi di cleanup
`watch` (slide 17).

**Benar**
```js
watch(filter, async (newVal, _oldVal, onCleanup) => {
  const ctrl = new AbortController()
  onCleanup(() => ctrl.abort())
  try {
    items.value = await fetchItems(newVal, ctrl.signal)
  } catch (err) {
    if (err.name !== 'AbortError') { error.value = err.message }
  }
})
```

> Catatan: dobel fetch juga muncul kalau `onMounted(load)` **dan** `watch(filter, load, { immediate: true })`
> dipasang bersama — dua-duanya jalan saat komponen muncul. Pilih satu.

---

## (e) Kebiasaan `innerHTML` Sesi 3 terbawa → `v-html` untuk data user

**Salah**
```vue
<!-- DetailPanel.vue -->
<div v-html="item.catatan"></div>   <!-- !!
```

**Gejala** — tidak ada error di console. Layar menampilkan tulisan aneh, atau — kalau kolom itu diisi
data dari luar — `<img src=x onerror="...">` benar-benar **jalan**. Ini "berhasil" secara tampilan,
yang membuatnya berbahaya.

**Akar** — `{{ }}` meng-escape otomatis (slide 11). `v-html` melewatkan escaping dan mem-parse teks
sebagai HTML — persis `innerHTML` yang kamu pelajari sebagai berbahaya di Sesi 3.

**Benar**
```vue
<div>{{ item.catatan }}</div>
```
Pakai `v-html` hanya untuk HTML yang **kamu sendiri** yang menyusunnya, tidak pernah untuk data dari
luar.

---

## Setelah memilih

Kalau kamu memilih salah satu di atas, jawab dengan **bukti milikmu**: pesan console asli (tempel apa
adanya), 3–5 baris kode dari berkasmu, dan nama berkas + nama fungsi. Kalau kejadianmu berbeda dari
kelima ini, tulis kejadianmu — tidak harus cocok dengan daftar.
