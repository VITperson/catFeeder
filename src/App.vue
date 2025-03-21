<template>
  <div class="container mt-4">
    <h1>Отслеживание кормления кота</h1>
    
    <div class="card mb-4">
      <div class="card-body">
        <h2 class="card-title">Добавить кормление</h2>
        
        <div class="mb-3">
          <label for="feedingTime" class="form-label">Время:</label>
          <input 
            type="datetime-local" 
            class="form-control" 
            id="feedingTime" 
            v-model="feedingTime"
          >
        </div>
        
        <div class="mb-3">
          <label for="foodType" class="form-label">Тип корма:</label>
          <select class="form-select" id="foodType" v-model="foodType">
            <option value="dry">Сухой</option>
            <option value="wet">Влажный</option>
            <option value="treat">Лакомство</option>
          </select>
        </div>
        
        <div class="mb-3" v-if="foodType === 'wet'">
          <label for="volume" class="form-label">Объем:</label>
          <select class="form-select" id="volume" v-model="volume">
            <option value="full packet">Полная упаковка</option>
            <option value="half">Половина</option>
            <option value="1/3">1/3</option>
            <option value="2/3">2/3</option>
          </select>
        </div>
        
        <button @click="addFeeding" class="btn btn-primary">Добавить запись</button>
      </div>
    </div>
    
    <button @click="toggleStats" class="btn btn-secondary mb-3">
      {{ showStats ? 'Скрыть статистику' : 'Показать статистику' }}
    </button>
    
    <div v-if="showStats">
      <h2>Статистика кормлений</h2>
      <table class="table table-striped">
        <thead>
          <tr>
            <th>Время</th>
            <th>Тип корма</th>
            <th>Объем</th>
            <th>Действия</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(feeding, index) in feedings" :key="index">
            <td>{{ formatDateTime(feeding.time) }}</td>
            <td>{{ translateFoodType(feeding.foodType) }}</td>
            <td>{{ translateVolume(feeding.volume) }}</td>
            <td>
              <button @click="deleteFeeding(index)" class="btn btn-sm btn-danger">
                Удалить
              </button>
            </td>
          </tr>
        </tbody>
      </table>
      
      <div v-if="feedings.length === 0" class="alert alert-info">
        Записей о кормлении пока нет.
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      feedingTime: this.getCurrentDateTime(),
      foodType: 'dry',
      volume: 'full packet',
      feedings: [],
      showStats: false
    }
  },
  created() {
    // Загрузка данных из локального хранилища при инициализации
    this.loadFeedingsFromStorage();
  },
  methods: {
    getCurrentDateTime() {
      const now = new Date();
      const year = now.getFullYear();
      const month = String(now.getMonth() + 1).padStart(2, '0');
      const day = String(now.getDate()).padStart(2, '0');
      const hours = String(now.getHours()).padStart(2, '0');
      const minutes = String(now.getMinutes()).padStart(2, '0');
      
      return `${year}-${month}-${day}T${hours}:${minutes}`;
    },
    
    addFeeding() {
      const feeding = {
        time: this.feedingTime,
        foodType: this.foodType,
        volume: this.foodType === 'wet' ? this.volume : '-',
        id: Date.now() // Уникальный идентификатор для записи
      };
      
      this.feedings.push(feeding);
      
      // Сохранение данных в локальное хранилище
      this.saveFeedingsToStorage();
      
      // Сброс формы
      this.feedingTime = this.getCurrentDateTime();
      this.foodType = 'dry';
      this.volume = 'full packet';
    },
    
    deleteFeeding(index) {
      if (confirm('Вы уверены, что хотите удалить эту запись?')) {
        this.feedings.splice(index, 1);
        // Сохранение обновленного списка в локальное хранилище
        this.saveFeedingsToStorage();
      }
    },
    
    toggleStats() {
      this.showStats = !this.showStats;
    },
    
    formatDateTime(dateTimeString) {
      const date = new Date(dateTimeString);
      return new Intl.DateTimeFormat('ru-RU', {
        day: '2-digit',
        month: '2-digit',
        year: 'numeric',
        hour: '2-digit',
        minute: '2-digit'
      }).format(date);
    },
    
    translateFoodType(type) {
      const types = {
        'dry': 'Сухой',
        'wet': 'Влажный',
        'treat': 'Лакомство'
      };
      return types[type] || type;
    },
    
    translateVolume(volume) {
      const volumes = {
        'full packet': 'Полная упаковка',
        'half': 'Половина',
        '1/3': '1/3',
        '2/3': '2/3',
        '-': '-'
      };
      return volumes[volume] || volume;
    },
    
    // Методы для работы с локальным хранилищем
    saveFeedingsToStorage() {
      localStorage.setItem('catFeedings', JSON.stringify(this.feedings));
    },
    
    loadFeedingsFromStorage() {
      const stored = localStorage.getItem('catFeedings');
      if (stored) {
        try {
          this.feedings = JSON.parse(stored);
        } catch (e) {
          console.error('Ошибка при загрузке данных из хранилища:', e);
          this.feedings = [];
        }
      }
    }
  }
}
</script>

<style scoped>
.container {
  max-width: 800px;
}
</style> 