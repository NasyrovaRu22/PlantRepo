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
            <label>Буква культуры (одна буква) *</label>
            <input v-model="newCulture.letter" maxlength="1" placeholder="Например: Т, О, П" style="width: 100px;">
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
              <span class="badge">Буква: {{ culture.letter }}</span>
              <span class="badge">ID: {{ culture.id }}</span>
              <br v-if="culture.description"><small>{{ culture.description }}</small>
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
              <option v-for="c in cultures" :key="c.id" :value="c.id">
                {{ c.name }} ({{ c.letter }})
              </option>
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
              <strong>🌾 {{ sort.name }}</strong>
              <span class="badge">ID: {{ sort.id }}</span><br>
              <small>Культура: {{ getCultureName(sort.cultureId) }} ({{ sort.cultureLetter }})</small><br>
              <small v-if="sort.ripening">Созревание: {{ sort.ripening }} дней</small><br>
              <small v-if="sort.features">Особенности: {{ sort.features }}</small>
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
              <option v-for="c in cultures" :key="c.id" :value="c.id">{{ c.name }} ({{ c.letter }})</option>
            </select>
          </div>
          <div class="form-group">
            <label>Сорт *</label>
            <select v-model="newBed.sortId">
              <option :value="null">Выберите сорт</option>
              <option v-for="s in filteredSorts" :key="s.id" :value="s.id">
                {{ s.name }} ({{ s.id }})
              </option>
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
              <strong>🧑‍🌾 {{ bed.name }}</strong>
              <span class="badge">ID: {{ bed.id }}</span><br>
              <small>Культура: {{ getCultureName(bed.cultureId) }}</small><br>
              <small>Сорт: {{ getSortName(bed.sortId) }} ({{ bed.sortCode }})</small><br>
              <small v-if="bed.quantity">Количество: {{ bed.quantity }} шт.</small><br>
              <small v-if="bed.plantingDate">Дата посадки: {{ bed.plantingDate }}</small>
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
              <strong>💧 {{ care.type }}</strong>
              <span class="badge">ID: {{ care.id }}</span><br>
              <small>Грядка: {{ getBedName(care.bedId) }}</small><br>
              <small>Дата: {{ care.date }}</small>
            </div>
            <div class="item-actions">
              <button class="delete-btn" @click="deleteCare(care.id)">Удалить</button>
            </div>
          </div>
        </div>
      </div>
    </div>
        <!-- Вкладка: Мой огород -->
    <div v-show="activeTab === 'myGarden'" class="tab-content">
      <div class="card">
        <h2>🌳 Мой огород</h2>
        <div class="garden-tree">
          <div v-for="culture in cultures" :key="culture.id" class="tree-culture">
            <div class="culture-header" @click="toggleCulture(culture.id)">
              <span class="toggle-icon">{{ expandedCultures[culture.id] ? '▼' : '▶' }}</span>
              <span class="culture-icon">🌿</span>
              <strong>{{ culture.name }}</strong>
              <span class="badge">{{ culture.letter }}</span>
              <span class="count-badge">{{ getSortsByCulture(culture.id).length }} сортов</span>
            </div>
            
            <div v-show="expandedCultures[culture.id]" class="tree-sorts">
              <div v-for="sort in getSortsByCulture(culture.id)" :key="sort.id" class="tree-sort">
                <div class="sort-header" @click="toggleSort(sort.id)">
                  <span class="toggle-icon">{{ expandedSorts[sort.id] ? '▼' : '▶' }}</span>
                  <span class="sort-icon">🌾</span>
                  <strong>{{ sort.name }}</strong>
                  <span class="badge">{{ sort.id }}</span>
                  <span class="count-badge">{{ getBedsBySort(sort.id).length }} грядок</span>
                </div>
                
                <div v-show="expandedSorts[sort.id]" class="tree-beds">
                  <div v-for="bed in getBedsBySort(sort.id)" :key="bed.id" class="tree-bed">
                    <div class="bed-header" @click="toggleBed(bed.id)">
                      <span class="toggle-icon">{{ expandedBeds[bed.id] ? '▼' : '▶' }}</span>
                      <span class="bed-icon">🧑‍🌾</span>
                      <strong>{{ bed.name }}</strong>
                      <span class="badge">ID: {{ bed.id }}</span>
                      <span class="info-text" v-if="bed.quantity">🌱 {{ bed.quantity }} шт.</span>
                      <span class="info-text" v-if="bed.plantingDate">📅 {{ bed.plantingDate }}</span>
                      <span class="count-badge">{{ getCaresByBed(bed.id).length }} записей</span>
                    </div>
                    
                    <div v-show="expandedBeds[bed.id]" class="tree-cares">
                      <div v-for="care in getCaresByBed(bed.id)" :key="care.id" class="tree-care">
                        <span class="care-icon">💧</span>
                        <strong>{{ care.type }}</strong>
                        <span class="date-text">📅 {{ care.date }}</span>
                        <span class="badge">ID: {{ care.id }}</span>
                      </div>
                      <div v-if="getCaresByBed(bed.id).length === 0" class="empty-care">
                        Нет записей об уходе
                      </div>
                    </div>
                  </div>
                  <div v-if="getBedsBySort(sort.id).length === 0" class="empty-bed">
                    Нет грядок с этим сортом
                  </div>
                </div>
              </div>
              <div v-if="getSortsByCulture(culture.id).length === 0" class="empty-sort">
                Нет сортов для этой культуры
              </div>
            </div>
          </div>
          <div v-if="cultures.length === 0" class="empty-garden">
            🌱 Добавьте культуры, сорта и грядки, чтобы увидеть свой огород
          </div>
        </div>
      </div>
      
      <div class="card stats-card">
        <h2>📊 Статистика огорода</h2>
        <div class="stats-grid">
          <div class="stat-item">
            <span class="stat-number">{{ cultures.length }}</span>
            <span class="stat-label">Культур</span>
          </div>
          <div class="stat-item">
            <span class="stat-number">{{ sorts.length }}</span>
            <span class="stat-label">Сортов</span>
          </div>
          <div class="stat-item">
            <span class="stat-number">{{ beds.length }}</span>
            <span class="stat-label">Грядок</span>
          </div>
          <div class="stat-item">
            <span class="stat-number">{{ cares.length }}</span>
            <span class="stat-label">Записей ухода</span>
          </div>
          <div class="stat-item">
            <span class="stat-number">{{ totalPlants }}</span>
            <span class="stat-label">Всего растений</span>
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
  { id: 'care', name: 'Уход', icon: '💧' },
  { id: 'myGarden', name: 'Мой огород', icon: '🌳' }
]

// Культуры
const cultures = ref([])
const newCulture = ref({ name: '', letter: '', description: '' })

// Сорта
const sorts = ref([])
const newSort = ref({ name: '', cultureId: null, ripening: '', features: '' })

// Грядки
const beds = ref([])
const newBed = ref({ name: '', cultureId: null, sortId: null, quantity: '', plantingDate: '' })

// Уход
const cares = ref([])
const newCare = ref({ bedId: null, date: '', type: '' })

// Счётчики ID
let nextId = { culture: 1, bed: 1, care: 1 }

// Счётчики сортов для каждой культуры { cultureId: количество }
const sortCounters = ref({})
const expandedCultures = ref({})
const expandedSorts = ref({})
const expandedBeds = ref({})

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

const getCultureLetter = (id) => {
  const culture = cultures.value.find(c => c.id === id)
  return culture ? culture.letter : '?'
}

const getSortName = (id) => {
  const sort = sorts.value.find(s => s.id === id)
  return sort ? sort.name : 'Неизвестно'
}

const getBedName = (id) => {
  const bed = beds.value.find(b => b.id === id)
  return bed ? bed.name : 'Неизвестно'
}
const toggleCulture = (id) => { expandedCultures.value[id] = !expandedCultures.value[id] }
const toggleSort = (id) => { expandedSorts.value[id] = !expandedSorts.value[id] }
const toggleBed = (id) => { expandedBeds.value[id] = !expandedBeds.value[id] }

const getSortsByCulture = (cultureId) => sorts.value.filter(s => s.cultureId === cultureId)
const getBedsBySort = (sortId) => beds.value.filter(b => b.sortId === sortId)
const getCaresByBed = (bedId) => cares.value.filter(c => c.bedId === bedId)

const totalPlants = computed(() => {
  return beds.value.reduce((sum, bed) => sum + (parseInt(bed.quantity) || 0), 0)
})

// ========== CRUD: Культуры ==========
const addCulture = () => {
  if (!newCulture.value.name || !newCulture.value.letter) {
    alert('Введите название культуры и букву!')
    return
  }
  
  const letter = newCulture.value.letter.toUpperCase()
  
  // Проверяем, что буква не занята
  if (cultures.value.some(c => c.letter === letter)) {
    alert(`Буква "${letter}" уже используется!`)
    return
  }
  
  cultures.value.push({
    id: letter,  // ← ID = БУКВА
    name: newCulture.value.name,
    letter: letter,
    description: newCulture.value.description
  })
  
  newCulture.value = { name: '', letter: '', description: '' }
}

const deleteCulture = (id) => {
  if (confirm('Удалить культуру? Все связанные сорта и грядки также будут удалены!')) {
    cultures.value = cultures.value.filter(c => c.id !== id)
    sorts.value = sorts.value.filter(s => s.cultureId !== id)
    
    // Удаляем счётчик для этой культуры
    delete sortCounters.value[id]
    
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
  
  // Находим культуру, чтобы узнать её букву
  const culture = cultures.value.find(c => c.id === newSort.value.cultureId)
  if (!culture) return
  
  // Получаем следующий номер для этой культуры
  const nextNumber = (sortCounters.value[culture.id] || 0) + 1
  
  // Формируем ID: буква + номер (например "Т1", "О2")
  const sortId = `${culture.letter}${nextNumber}`
  
  sorts.value.push({
    id: sortId,
    name: newSort.value.name,
    cultureId: newSort.value.cultureId,
    cultureLetter: culture.letter,
    ripening: newSort.value.ripening || null,
    features: newSort.value.features || null
  })
  
  // Обновляем счётчик для этой культуры
  sortCounters.value[culture.id] = nextNumber
  
  newSort.value = { name: '', cultureId: null, ripening: '', features: '' }
}

const deleteSort = (id) => {
  if (confirm('Удалить сорт?')) {
    // Находим сорт перед удалением
    const sortToDelete = sorts.value.find(s => s.id === id)
    
    sorts.value = sorts.value.filter(s => s.id !== id)
    
    const bedsToDelete = beds.value.filter(b => b.sortId === id)
    beds.value = beds.value.filter(b => b.sortId !== id)
    cares.value = cares.value.filter(c => !bedsToDelete.some(b => b.id === c.bedId))
    
    // Обновляем счётчики для культуры (перенумеровываем оставшиеся сорта)
    if (sortToDelete) {
      const cultureId = sortToDelete.cultureId
      const remainingSorts = sorts.value.filter(s => s.cultureId === cultureId)
      
      // Перенумеровываем сорта
      remainingSorts.forEach((sort, index) => {
        const newId = `${sortToDelete.cultureLetter}${index + 1}`
        if (sort.id !== newId) {
          sort.id = newId
        }
      })
      
      // Обновляем счётчик
      sortCounters.value[cultureId] = remainingSorts.length
    }
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
  
  // Находим сорт, чтобы получить его ID
  const selectedSort = sorts.value.find(s => s.id === newBed.value.sortId)
  
  beds.value.push({
    id: nextId.bed++,
    name: newBed.value.name,
    cultureId: newBed.value.cultureId,
    sortId: newBed.value.sortId,
    sortCode: selectedSort ? selectedSort.id : null,
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
  localStorage.setItem('garden_sortCounters', JSON.stringify(sortCounters.value))
}

const loadFromLocalStorage = () => {
  const savedCultures = localStorage.getItem('garden_cultures')
  const savedSorts = localStorage.getItem('garden_sorts')
  const savedBeds = localStorage.getItem('garden_beds')
  const savedCares = localStorage.getItem('garden_cares')
  const savedIds = localStorage.getItem('garden_ids')
  const savedCounters = localStorage.getItem('garden_sortCounters')
  
  if (savedCultures) cultures.value = JSON.parse(savedCultures)
  if (savedSorts) sorts.value = JSON.parse(savedSorts)
  if (savedBeds) beds.value = JSON.parse(savedBeds)
  if (savedCares) cares.value = JSON.parse(savedCares)
  if (savedIds) nextId = JSON.parse(savedIds)
  if (savedCounters) sortCounters.value = JSON.parse(savedCounters)
}

// ========== Установка сегодняшней даты ==========
const setTodayDate = () => {
  const today = new Date().toISOString().split('T')[0]
  newCare.value.date = today
}

// ========== Watch для автосохранения ==========
watch([cultures, sorts, beds, cares, sortCounters], () => {
  saveToLocalStorage()
}, { deep: true })

// ========== Монтирование ==========
onMounted(() => {
  loadFromLocalStorage()
  setTodayDate()
})
</script>

<style scoped>
body {
  margin: 0;
  padding: 0;
  min-height: 100vh;
}
.app-container {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #e8f5e9 0%, #c8e6c9 100%);
  min-height: 100vh;
  padding: 20px;
  width: 100%;
  margin: 0;
}

h1 {
  text-align: center;
  color: #2e7d32;
  margin-bottom: 30px;
  font-size: 2.2em;
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
/* ========== Стили для вкладки "Мой огород" ========== */
.garden-tree {
  padding: 10px 0;
}

.tree-culture {
  margin-bottom: 15px;
  border-left: 3px solid #2e7d32;
  padding-left: 15px;
}

.culture-header, .sort-header, .bed-header {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px;
  background: #e8f5e9;
  border-radius: 10px;
  cursor: pointer;
  transition: all 0.2s;
  flex-wrap: wrap;
}

.culture-header:hover, .sort-header:hover, .bed-header:hover {
  background: #c8e6c9;
  transform: translateX(5px);
}

.tree-sorts, .tree-beds, .tree-cares {
  margin-left: 30px;
  margin-top: 10px;
}

.tree-sort, .tree-bed {
  margin-bottom: 12px;
  border-left: 2px solid #81c784;
  padding-left: 12px;
}

.sort-header {
  background: #f1f8e9;
}

.bed-header {
  background: #fff3e0;
}

.tree-care {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 6px 10px;
  margin: 5px 0;
  background: #e3f2fd;
  border-radius: 6px;
  flex-wrap: wrap;
}

.toggle-icon {
  font-size: 12px;
  color: #2e7d32;
  width: 20px;
}

.culture-icon, .sort-icon, .bed-icon, .care-icon {
  font-size: 18px;
}

.count-badge, .info-text, .date-text {
  background: #e0e0e0;
  padding: 2px 8px;
  border-radius: 12px;
  font-size: 11px;
  color: #555;
}

.info-text, .date-text {
  background: #fff3e0;
}

.empty-sort, .empty-bed, .empty-care, .empty-garden {
  text-align: center;
  color: #999;
  padding: 20px;
  font-style: italic;
}

.stats-card {
  background: linear-gradient(135deg, #2e7d32 0%, #1b5e20 100%);
  color: white;
}

.stats-card h2 {
  color: white;
  border-bottom-color: rgba(255,255,255,0.3);
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 15px;
  text-align: center;
}

.stat-item {
  background: rgba(255,255,255,0.2);
  border-radius: 12px;
  padding: 12px;
}

.stat-number {
  display: block;
  font-size: 28px;
  font-weight: bold;
}

.stat-label {
  font-size: 12px;
  opacity: 0.9;
}
</style>