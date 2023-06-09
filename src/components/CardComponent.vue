<template>
  <div class="card" :class="{ active: isFavorited }">
    <div class="card__head">
      <FormWeather @change-city="changeCity" />
      <div class="card__actions">
        <button
          v-if="!isFavorited"
          @click.prev="handleFavorited"
          class="card__actions-btn"
          :class="{ active: isClickedHeart }"
        >
          Favorite
        </button>
        <button
          v-if="isFavorited || props.index !== 0"
          @click.prev="handleRemove"
          class="card__actions-btn"
        >
          Delete
        </button>
      </div>
    </div>
    <div class="card__error" v-if="errors">
      {{ errors }}
    </div>
    <SpinnerWeather v-if="isLoading" />
    <div v-if="data" class="card__main-block">
      <div class="card__tab">
        <button
          @click.prev="currentFilter = 'oneDay'"
          class="card__tab-button"
          :class="{ active: currentFilter === 'oneDay' }"
        >
          Today
        </button>
        <button
          @click.prev="currentFilter = 'fiveDay'"
          class="card__tab-button"
          :class="{ active: currentFilter === 'fiveDay' }"
        >
          5 Day
        </button>
      </div>
        <div class="card-content">
            <div class="card__info info-card">
                <div class="info-card__city">{{ city.name }}</div>
                <div v-if="currentFilter === 'oneDay'" class="info-card__one-day">
                    <div class="info-card__head">
                        <div class="info-card__icon-temp"><img :src="iconWeather" alt="" /></div>
                        <div class="info-card__temp">{{ data.current.temp }}°C</div>
                    </div>
                    <div class="info-card__descr">
                        Feels like {{ data.current.feels_like }}°C, {{ data.current.weather[0].description }}
                    </div>
                    <ul class="info-card__list">
                        <li class="info-card__list-item">
                            <span class="info-card__icon"> <img :src="WindIcon" alt="" /> </span>Wind:
                            {{ data.current.wind_speed }}m/s
                        </li>
                        <li class="info-card__list-item">
                            <span class="info-card__icon"> <img :src="PressureIcon" alt="" /> </span>Pressure:
                            {{ data.current.pressure }}hPa
                        </li>
                        <li class="info-card__list-item">
              <span class="info-card__icon">
                <img :src="HumidityIcon" alt="" />
              </span>
                            Humidity: {{ data.current.humidity }}%
                        </li>
                        <li class="info-card__list-item">
              <span class="info-card__icon">
                <img :src="VisibilityIcon" alt="" />
              </span>
                            Visibility: {{ normalizeVisibility }}km
                        </li>
                    </ul>
                </div>
                <div v-if="currentFilter === 'fiveDay'" class="info-card__five-day">
                    <div class="info-card__row">
                        <div v-for="day of normalizeData" class="info-card__item small-info">
                            <div class="small-info__date">
                                {{ day.dt }}
                            </div>
                            <div class="small-info__head">
                                <img
                                        :src="`http://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png`"
                                        alt=""
                                />
                                <div class="small-info__temp">{{ day.temp }}°C</div>
                            </div>
                            <ul class="small-info__list">
                                <li class="small-info__item">Wind:{{ day.wind_speed }}</li>
                                <li class="small-info__item">Hum.:{{ day.humidity }}</li>
                                <li class="small-info__item">Pres.:{{ day.pressure }}</li>
                            </ul>
                        </div>
                    </div>
                </div>
            </div>
            <ChartWeather :data="normalizeData" />
        </div>

    </div>
  </div>
</template>

<script setup>
import VisibilityIcon from '@/assets/images/icons/visibility.svg'
import HumidityIcon from '@/assets/images/icons/humidity.svg'
import PressureIcon from '@/assets/images/icons/pressure.svg'
import WindIcon from '@/assets/images/icons/wind.svg'
import SpinnerWeather from '@/components/SpinnerComponent.vue'
import ChartWeather from '@/components/ChartComponent.vue'
import FormWeather from '@/components/FormComponent.vue'
import { getToday, getAllDays } from '@/helpers/withData.js'
import { getCoordinates } from '@/api/wheather.js'
import { useWheather } from '@/hooks/wheather.js'
import { onMounted, computed, ref, defineEmits } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()
const isFavorited = computed(() => {
  return route.name === 'favorite'
})
const { data, isLoading, errors, getData } = useWheather()
const props = defineProps({
  card: {
    type: Object,
    required: true
  },
  index: {
    type: Number,
    required: true
  }
})
const emit = defineEmits(['deleteItem', 'changeCity', 'addFavorited'])
const handleRemove = () => {
  emit('deleteItem', props.card.id)
}

const isClickedHeart = ref(false)
let timerHeart = null
const animateHeart = () => {
  isClickedHeart.value = false
  clearTimeout(timerHeart)
}
const handleFavorited = () => {
  isClickedHeart.value = true
  timerHeart = setTimeout(animateHeart, 1000)
  emit('addFavorited', props.card.id, props.card.isFavorited)
}

const currentFilter = ref('oneDay')
const normalizeData = computed(() => {
  if (currentFilter.value === 'oneDay' && data.value) {
    return getToday(data.value.hourly)
  } else if (currentFilter.value === 'fiveDay' && data.value) {
    return getAllDays(data.value.daily)
  }
})

const city = ref({})

const getDataWeather = (cityInfo) => {
  if (cityInfo.lat && cityInfo.lon) {
    city.value = cityInfo
    getData({ city: city.value }).then((res) => {})
  } else {
    getCoordinates(cityInfo.name).then((res) => {
      res
        ? (city.value = { lat: res.lat, lon: res.lon, name: res.name })
        : (city.value = { lat: null, lon: null, name: '' })

      getData({ city: city.value })
    })
  }
}

const changeCity = (newCity) => {
  emit('changeCity', newCity, props.card.id)
  getDataWeather(newCity)
}

onMounted(() => {
  getDataWeather(props.card.city)
})

const normalizeVisibility = computed(() => {
  if (data.value) {
    return parseFloat(data.value.current.visibility / 1000).toFixed(2)
  }
})

const iconWeather = computed(() => {
  if (data.value) {
    return `http://openweathermap.org/img/wn/${data.value.current.weather[0].icon}@2x.png`
  }
})
</script>

<style lang="scss" scoped>
.card {
  width: 100%;
  padding: 30px;
  border-radius: 5px;
  box-shadow: rgba(0, 0, 0, 0.35) 0px 5px 15px;
  background: #91a3b4;
  transition: all 0.3s cubic-bezier(0.5, 0.02, 0.13, 0.5);
  border: 3px solid transparent;
  .card-content{
    display: grid;
    grid-template-columns: 1fr 1fr;
    @media (max-width: $tablet) {
      grid-template-columns: 1fr ;
    }
  }
  @media (max-width: $tablet) {
    width: 100%;
  }
  @media (max-width: $mobile) {
    padding: 10px;
  }
  &.active {
    border: 3px solid var(--greenColor);
  }

  &__head {
    display: flex;
    align-items: center;
    justify-content: space-between;
    flex-wrap: wrap;
    gap: 10px;
    &:not(:last-child) {
      margin-bottom: 20px;
    }
  }

  &__actions {
    display: flex;
    align-items: center;
    gap: 20px;
    @media (max-width: $mobile) {
      gap: 10px;
    }
  }
  &__actions-btn {
    display: inline-flex;
    height: 40px;
    cursor: pointer;
    border: none;
    border-radius: 5px;
    align-items: center;
    justify-content: center;
    transition: all 0.3s cubic-bezier(0.5, 0.02, 0.13, 0.5);
    &.active {
      transform: scale(1.5);
      pointer-events: none;
    }
  }

  &__tab {
    display: flex;
    align-items: center;
    gap: 20px;
    font-size: 20px;
    margin: 15px 0;
  }

  &__tab-button {
    transition: all 0.3s cubic-bezier(0.5, 0.02, 0.13, 0.5);
    border: none;
    border-radius: 5px;
    cursor: pointer;
    &.active {
      text-decoration: underline;
    }
  }
}

.card__error {
  color: red;
  margin-bottom: 20px;
}
.info-card {
  &__icon-temp {
  }

  &__city {
    font-size: 40px;
    font-weight: 700;
    margin-bottom: rem(20);
    @media (max-width: $mobile) {
      font-size: 30px;
    }
  }

  &__head {
    display: flex;
    align-items: center;
    gap: 15px;
  }

  &__icon {
    width: 25px;
    height: 25px;
  }

  &__temp {
    font-size: 40px;
    @media (max-width: $mobile) {
      font-size: 30px;
    }
  }

  &__descr {
    font-size: 22px;
    margin: 20px 0;
    @media (max-width: $mobile) {
      font-size: 16px;
    }
  }

  &__list {
    margin-bottom: 30px;
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 10px;
    @media (max-width: $mobile) {
      margin-bottom: 20px;
    }
    @media (max-width: $mobileSmall) {
      grid-template-columns: repeat(1, 1fr);
    }
  }

  &__list-item {
    font-size: 20px;
    display: flex;
    align-items: center;
    gap: 15px;
    @media (max-width: $mobile) {
      font-size: 16px;
    }
  }
}
.info-card {
  &__row {
    display: grid;
    gap: 10px;
    grid-template-columns: repeat(auto-fit, minmax(80px, 1fr));
    margin: 20px 0px;
    font-size: 14px;
  }

  &__item {
  }
}

.small-info {
  text-align: center;
  &__head {
    display: flex;
    align-items: center;
    gap: 5px;
    img {
      width: 30px;
      height: 30px;
    }
  }

  &__date {
    font-weight: 700;
    margin-bottom: 5px;
    text-align: center;
  }
}
</style>
