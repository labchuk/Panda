<template>
  <form @submit.prev="handleSubmit" action="#" class="card__form form-card">
    <div class="form-card__item">
      <input
        :disabled="isFavorited"
        autocomplete="off"
        type="text"
        name="form[]"
        data-error="Ошибка"
        placeholder="Type name city"
        class="text-field"
        v-model="inputValue"
      />
      <div v-show="openDropCities" class="form-card__cities">
        <div v-if="isLoading" class="form-card__text">...Loading</div>
        <div v-if="data && data.length === 0" class="form-card__text">No results</div>
        <div v-if="errors" class="form-card__text">{{ errors }}</div>
        <ul v-if="data" class="form-card__cities-list">
          <li
            @click="handleChangeCity({ name: item.name, lat: item.lat, lon: item.lon })"
            v-for="item of data"
            :key="item.id"
            class="form-card__city"
          >
            {{ item.name }},{{ item.country }}
          </li>
        </ul>
      </div>
      <button :disabled="isFavorited" class="button form-card__button">Search</button>
    </div>
  </form>
</template>

<script setup>
import { useCities } from '@/hooks/cities'
import { ref, watch, defineEmits, computed } from 'vue'
import { useRoute } from 'vue-router'

const { data, errors, isLoading, getCities } = useCities()
const openDropCities = ref(false)
const inputValue = ref('')
const cityInfo = ref({})
const emit = defineEmits(['changeCity'])

const route = useRoute()
const isFavorited = computed(() => {
  return route.name === 'favorite'
})

let timerId = null

watch(inputValue, (oldValue, newValue) => {
  if (oldValue.length > 2) {
    openDropCities.value = true
    clearTimeout(timerId)
    isLoading.value = true
    data.value = null
    timerId = setTimeout(searchCities, 1000)
  } else {
    openDropCities.value = false
  }
})

const handleSubmit = () => {
  if (inputValue.value.trim().length > 0) {
    cityInfo.value = {
      name: inputValue.value,
      lat: '',
      lon: ''
    }
    emit('changeCity', cityInfo.value)
    resetCity()
  }
}

const handleChangeCity = (city) => {
  cityInfo.value = city
  emit('changeCity', cityInfo.value)
  resetCity()
}

const resetCity = () => {
  cityInfo.value = {}
  inputValue.value = ''
  openDropCities.value = false
}

const searchCities = () => {
  getCities({ query: inputValue.value }).then((res) => {
    isLoading.value = false
  })
}
</script>

<style lang="scss" scoped>
.form-card {
  @media (max-width: $mobileSmall) {
    width: 100%;
  }
  &__text {
    color: #000;
    width: 100%;
  }
  &__cities-list {
    display: grid;
    gap: 10px;
    width: 100%;
    color: #000;
    text-align: left;
    align-self: flex-start;
  }

  &__cities {
    min-height: 100px;
    position: absolute;
    max-height: 200px;
    border-radius: 5px;
    z-index: 4;
    background: #ffffff;
    width: 100%;
    padding: 20px;
    overflow: auto;

    top: calc(100% + 3px);
    left: 0;
  }

  &__city {
    color: #000;
    cursor: pointer;
    align-items: center;
    transition: all 0.3s cubic-bezier(0.5, 0.02, 0.13, 0.5);
  }
  &__item {
    display: flex;
    position: relative;
    gap: 10px;
    .text-field{
      width: 100%;
      max-width: 400px;
      height: 40px;
      border-radius: 5px;
      border: none;
      padding-left: 10px;

    }
    .button{
      border-radius: 5px;
      cursor: pointer;
      background-color: #e96e50;
      border: none;
      color: #fff;
    }
    @media (max-width: $mobileSmall) {
      display: grid;
      width: 100%;
      grid-template-columns: repeat(1, 1fr);
    }
  }
}
</style>
