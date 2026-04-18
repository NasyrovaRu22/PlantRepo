<template>
  <div class="app-container">
    <h1>🌱 Садовый учет</h1>

    <!-- Вкладки -->
    <div class="tabs">
      <button 
        v-for="tab in tabs" 
        :key="tab.id"
        :class="['tab-btn', { active: activeTab === tab.id }]"
        @click="activeTab = tab.id"
      >
        {{ tab.icon }} {{ tab.name }}
      </button>
    </div>

    <!-- Вкладка: Культуры -->
    <div v-show="activeTab === 'cultures'" class="tab-content">
      <div class="card">
        <h2>➕ Добавить культуру</h2>
        <div class="grid-2">
          <div class="form-group">
            <label>Название культуры *</label>
            <input v-model="newCulture.name" placeholder="Например: Томат, Огурец, Перец">
          </div>
          <div class="form-group">
            <label>Описание</label>
            <textarea v-model="newCulture.description" placeholder="Краткое описание..."></textarea>
          </div>
        </div>
        <button @click="addCulture">🌿 Добавить культуру</button>
      </div>

      <div class="card">
        <h2>📋 Список культур</h2>
        <div class="items-list">
          <div v-if="cultures.length === 0" class="empty-msg">Нет добавленных культур. Создайте первую!</div>
          <div v-for="culture in cultures" :key="culture.id" class="item-card">
            <div class="item-info">
              <strong>🌿 {{ culture.name }}</strong>
              <br v-if="culture.description"><small>{{ culture.description }}</small>
              <span class="badge">ID: {{ culture.id }}</span>
            </div>
            <div class="item-actions">
              <button class="delete-btn" @click="deleteCulture(culture.id)">Удалить</button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Вкладка: Сорта -->
    <div v-show="activeTab === 'sorts'" class="tab-content">
      <div class="card">
        <h2>➕ Добавить сорт</h2>
        <div class="grid-2">
          <div class="form-group">
            <label>Название сорта *</label>
            <input v-model="newSort.name" placeholder="Например: Бычье сердце">
          </div>
          <div class="form-group">
            <label>Культура *</label>
            <select v-model="newSort.cultureId">
              <option :value="null">Выберите культуру</option>
              <option v-for="c in cultures" :key="c.id" :value="c.id">{{ c.name }}</option>
            </select>
          </div>
          <div class="form-group">
            <label>Срок созревания (дней)</label>
            <input v-model="newSort.ripening" type="number" placeholder="Например: 80">
          </div>
          <div class="form-group">
            <label>Особенности</label>
            <input v-model="newSort.features" placeholder="Устойчивость, урожайность...">
          </div>
        </div>
        <button @click="addSort">🌾 Добавить сорт</button>
      </div>

      <div class="card">
        <h2>📋 Список сортов</h2>
        <div class="items-list">
          <div v-if="sorts.length === 0" class="empty-msg">Нет добавленных сортов. Создайте первый!</div>
          <div v-for="sort in sorts" :key="sort.id" class="item-card">
            <div class="item-info">
              <strong>🌾 {{ sort.name }}</strong><br>
              <small>Культура: {{ getCultureName(sort.cultureId) }}</small><br>
              <small v-if="sort.ripening">Созревание: {{ sort.ripening }} дней</small><br>
              <small v-if="sort.features">Особенности: {{ sort.features }}</small>
              <span class="badge">ID: {{ sort.id }}</span>
            </div>
            <div class="item-actions">
              <button class="delete-btn" @click="deleteSort(sort.id)">Удалить</button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Вкладка: Грядки -->
    <div v-show="activeTab === 'beds'" class="tab-content">
      <div class="card">
        <h2>➕ Добавить грядку</h2>
        <div class="grid-2">
          <div class="form-group">
            <label>Название грядки *</label>
            <input v-model="newBed.name" placeholder="Например: Грядка №1, Теплица">
          </div>
          <div class="form-group">
            <label>Культура *</label>
            <select v-model="newBed.cultureId" @change="onCultureChange">
              <option :value="null">Выберите культуру</option>
              <option v-for="c in cultures" :key="c.id" :value="c.id">{{ c.name }}</option>
            </select>
          </div>
          <div class="form-group">
            <label>Сорт *</label>
            <select v-model="newBed.sortId">
              <option :value="null">Выберите сорт</option>
              <option v-for="s in filteredSorts" :key="s.id" :value="s.id">{{ s.name }}</option>
            </select>
          </div>
          <div class="form-group">
            <label>Количество растений</label>
            <input v-model="newBed.quantity" type="number" placeholder="Количество">
          </div>
          <div class="form-group">
            <label>Дата посадки</label>
            <input v-model="newBed.plantingDate" type="date">
          </div>
        </div>
        <button @click="addBed">🧑‍🌾 Добавить грядку</button>
      </div>

      <div class="card">
        <h2>📋 Список грядок</h2>
        <div class="items-list">
          <div v-if="beds.length === 0" class="empty-msg">Нет добавленных грядок. Создайте первую!</div>
          <div v-for="bed in beds" :key="bed.id" class="item-card">
            <div class="item-info">
              <strong>🧑‍🌾 {{ bed.name }}</strong><br>
              <small>Культура: {{ getCultureName(bed.cultureId) }}</small><br>
              <small>Сорт: {{ getSortName(bed.sortId) }}</small><br>
              <small v-if="bed.quantity">Количество: {{ bed.quantity }} шт.</small><br>
              <small v-if="bed.plantingDate">Дата посадки: {{ bed.plantingDate }}</small>
              <span class="badge">ID: {{ bed.id }}</span>
            </div>
            <div class="item-actions">
              <button class="delete-btn" @click="deleteBed(bed.id)">Удалить</button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Вкладка: Уход -->
    <div v-show="activeTab === 'care'" class="tab-content">
      <div class="card">
        <h2>➕ Добавить запись об уходе</h2>
        <div class="grid-2">
          <div class="form-group">
            <label>Грядка *</label>
            <select v-model="newCare.bedId">
              <option :value="null">Выберите грядку</option>
              <option v-for="b in beds" :key="b.id" :value="b.id">
                {{ b.name }} ({{ getCultureName(b.cultureId) }})
              </option>
            </select>
          </div>
          <div class="form-group">
            <label>Дата *</label>
            <input v-model="newCare.date" type="date">
          </div>
          <div class="form-group">
            <label>Тип ухода *</label>
            <input v-model="newCare.type" placeholder="Полив, удобрение, обрезка, прополка...">
          </div>
        </div>
        <button @click="addCare">💧 Добавить уход</button>
      </div>

      <div class="card">
        <h2>📋 История ухода</h2>
        <div class="items-list">
          <div v-if="cares.length === 0" class="empty-msg">Нет записей об уходе. Добавьте первую!</div>
          <div v-for="care in cares" :key="care.id" class="item-card">
            <div class="item-info">
              <strong>💧 {{ care.type }}</strong><br>
              <small>Грядка: {{ getBedName(care.bedId) }}</small><br>
              <small>Дата: {{ care.date }}</small>
              <span class="badge">ID: {{ care.id }}</span>
            </div>
            <div class="item-actions">
              <button class="delete-btn" @click="deleteCare(care.id)">Удалить</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'

// ========== Данные ==========
const activeTab = ref('cultures')

const tabs = [
  { id: 'cultures', name: 'Культуры', icon: '🌿' },
  { id: 'sorts', name: 'Сорта', icon: '🌾' },
  { id: 'beds', name: 'Грядки', icon: '🧑‍🌾' },
  { id: 'care', name: 'Уход', icon: '💧' }
]

// Культуры
const cultures = ref([])
const newCulture = ref({ name: '', description: '' })

// Сорта
const sorts = ref([])
const newSort = ref({ name: '', cultureId: null, ripening: '', features: '' })

// Грядки
const beds = ref([])
const newBed = ref({ name: '', cultureId: null, sortId: null, quantity: '', plantingDate: '' })

// Уход
const cares = ref([])
const newCare = ref({ bedId: null, date: '', type: '' })

// ID счетчики
let nextId = { culture: 1, sort: 1, bed: 1, care: 1 }

// ========== Вычисляемые свойства ==========
const filteredSorts = computed(() => {
  if (!newBed.value.cultureId) return sorts.value
  return sorts.value.filter(s => s.cultureId === newBed.value.cultureId)
})

// ========== Вспомогательные функции ==========
const getCultureName = (id) => {
  const culture = cultures.value.find(c => c.id === id)
  return culture ? culture.name : 'Неизвестно'
}

const getSortName = (id) => {
  const sort = sorts.value.find(s => s.id === id)
  return sort ? sort.name : 'Неизвестно'
}

const getBedName = (id) => {
  const bed = beds.value.find(b => b.id === id)
  return bed ? bed.name : 'Неизвестно'
}

// ========== CRUD: Культуры ==========
const addCulture = () => {
  if (!newCulture.value.name) {
    alert('Введите название культуры!')
    return
  }
  
  cultures.value.push({
    id: nextId.culture++,
    name: newCulture.value.name,
    description: newCulture.value.description
  })
  
  newCulture.value = { name: '', description: '' }
}

const deleteCulture = (id) => {
  if (confirm('Удалить культуру? Все связанные сорта и грядки также будут удалены!')) {
    cultures.value = cultures.value.filter(c => c.id !== id)
    sorts.value = sorts.value.filter(s => s.cultureId !== id)
    
    const bedsToDelete = beds.value.filter(b => b.cultureId === id)
    beds.value = beds.value.filter(b => b.cultureId !== id)
    cares.value = cares.value.filter(c => !bedsToDelete.some(b => b.id === c.bedId))
  }
}

// ========== CRUD: Сорта ==========
const addSort = () => {
  if (!newSort.value.name || !newSort.value.cultureId) {
    alert('Заполните название сорта и выберите культуру!')
    return
  }
  
  sorts.value.push({
    id: nextId.sort++,
    name: newSort.value.name,
    cultureId: newSort.value.cultureId,
    ripening: newSort.value.ripening || null,
    features: newSort.value.features || null
  })
  
  newSort.value = { name: '', cultureId: null, ripening: '', features: '' }
}

const deleteSort = (id) => {
  if (confirm('Удалить сорт?')) {
    sorts.value = sorts.value.filter(s => s.id !== id)
    
    const bedsToDelete = beds.value.filter(b => b.sortId === id)
    beds.value = beds.value.filter(b => b.sortId !== id)
    cares.value = cares.value.filter(c => !bedsToDelete.some(b => b.id === c.bedId))
  }
}

// ========== CRUD: Грядки ==========
const onCultureChange = () => {
  newBed.value.sortId = null
}

const addBed = () => {
  if (!newBed.value.name || !newBed.value.cultureId || !newBed.value.sortId) {
    alert('Заполните название грядки, выберите культуру и сорт!')
    return
  }
  
  beds.value.push({
    id: nextId.bed++,
    name: newBed.value.name,
    cultureId: newBed.value.cultureId,
    sortId: newBed.value.sortId,
    quantity: newBed.value.quantity || null,
    plantingDate: newBed.value.plantingDate || null
  })
  
  newBed.value = { name: '', cultureId: null, sortId: null, quantity: '', plantingDate: '' }
}

const deleteBed = (id) => {
  if (confirm('Удалить грядку? Все записи об уходе за ней также будут удалены!')) {
    beds.value = beds.value.filter(b => b.id !== id)
    cares.value = cares.value.filter(c => c.bedId !== id)
  }
}

// ========== CRUD: Уход ==========
const addCare = () => {
  if (!newCare.value.bedId || !newCare.value.date || !newCare.value.type) {
    alert('Заполните все поля!')
    return
  }
  
  cares.value.push({
    id: nextId.care++,
    bedId: newCare.value.bedId,
    date: newCare.value.date,
    type: newCare.value.type
  })
  
  newCare.value = { bedId: null, date: '', type: '' }
}

const deleteCare = (id) => {
  if (confirm('Удалить запись об уходе?')) {
    cares.value = cares.value.filter(c => c.id !== id)
  }
}

// ========== LocalStorage ==========
const saveToLocalStorage = () => {
  localStorage.setItem('garden_cultures', JSON.stringify(cultures.value))
  localStorage.setItem('garden_sorts', JSON.stringify(sorts.value))
  localStorage.setItem('garden_beds', JSON.stringify(beds.value))
  localStorage.setItem('garden_cares', JSON.stringify(cares.value))
  localStorage.setItem('garden_ids', JSON.stringify(nextId))
}

const loadFromLocalStorage = () => {
  const savedCultures = localStorage.getItem('garden_cultures')
  const savedSorts = localStorage.getItem('garden_sorts')
  const savedBeds = localStorage.getItem('garden_beds')
  const savedCares = localStorage.getItem('garden_cares')
  const savedIds = localStorage.getItem('garden_ids')
  
  if (savedCultures) cultures.value = JSON.parse(savedCultures)
  if (savedSorts) sorts.value = JSON.parse(savedSorts)
  if (savedBeds) beds.value = JSON.parse(savedBeds)
  if (savedCares) cares.value = JSON.parse(savedCares)
  if (savedIds) nextId = JSON.parse(savedIds)
}

// ========== Установка сегодняшней даты ==========
const setTodayDate = () => {
  const today = new Date().toISOString().split('T')[0]
  newCare.value.date = today
}

// ========== Watch для автосохранения ==========
watch([cultures, sorts, beds, cares], () => {
  saveToLocalStorage()
}, { deep: true })

// ========== Монтирование ==========
onMounted(() => {
  loadFromLocalStorage()
  setTodayDate()
})
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  margin: 0;
  padding: 0;
}

.app-container {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #e8f5e9 0%, #c8e6c9 100%);
  min-height: 100vh;
  padding: 20px;
  width: 100vw;
  margin: 0;
  position: absolute;
  top: 0;
  left: 0;
}

h1 {
  text-align: center;
  color: #2e7d32;
  margin-bottom: 30px;
  font-size: 2.2em;
}

.tabs {
  display: flex;
  gap: 10px;
  margin-bottom: 25px;
  flex-wrap: wrap;
}

.tab-btn {
  background: white;
  border: none;
  padding: 12px 25px;
  font-size: 16px;
  font-weight: bold;
  border-radius: 30px;
  cursor: pointer;
  transition: all 0.3s;
  color: #2e7d32;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.tab-btn.active {
  background: #2e7d32;
  color: white;
}

.tab-btn:hover:not(.active) {
  background: #a5d6a7;
}

.tab-content {
  display: block;
}

.card {
  background: white;
  border-radius: 15px;
  padding: 20px;
  margin-bottom: 20px;
  box-shadow: 0 5px 15px rgba(0,0,0,0.1);
  width: 100%;
}

.card h2 {
  color: #2e7d32;
  margin-bottom: 15px;
  border-bottom: 2px solid #c8e6c9;
  padding-bottom: 8px;
}

.form-group {
  margin-bottom: 15px;
}

label {
  display: block;
  font-weight: 600;
  margin-bottom: 5px;
  color: #333;
}

input, select, textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 14px;
}

textarea {
  resize: vertical;
  min-height: 60px;
}

button {
  background: #2e7d32;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 14px;
  font-weight: bold;
  transition: background 0.3s;
}

button:hover {
  background: #1b5e20;
}

.delete-btn {
  background: #e53935;
}

.delete-btn:hover {
  background: #c62828;
}

.items-list {
  margin-top: 20px;
  max-height: 400px;
  overflow-y: auto;
  width: 100%;
}

.item-card {
  background: #f5f5f5;
  border-radius: 10px;
  padding: 12px;
  margin-bottom: 10px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 10px;
  width: 100%;
}

.item-info {
  flex: 1;
}

.item-info strong {
  color: #2e7d32;
}

.item-actions {
  display: flex;
  gap: 8px;
}

.item-actions button {
  padding: 5px 12px;
  font-size: 12px;
}

.empty-msg {
  text-align: center;
  color: #999;
  padding: 30px;
  font-style: italic;
}

.badge {
  display: inline-block;
  background: #c8e6c9;
  color: #2e7d32;
  padding: 3px 10px;
  border-radius: 15px;
  font-size: 12px;
  margin-right: 8px;
  margin-top: 5px;
}

.grid-2 {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

@media (max-width: 768px) {
  .grid-2 {
    grid-template-columns: 1fr;
  }
  .item-card {
    flex-direction: column;
    align-items: flex-start;
  }
}
</style>