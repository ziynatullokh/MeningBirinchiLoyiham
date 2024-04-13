<script setup>
import { watch, ref} from "vue"
import axios from "axios"

const renderData = ref({
  country: "Davlat",
  localtime: "Mahalliy vaqt",
  region: "Shahar",
  feelslike_c: "Temperatura Selsiy",
  feelslike_f: "Temperatura Farengeyt"
})
const cityInput = ref("")
const weatherData = ref({})
const fetchValid = ref(false)
const isLoading = ref(false)
const errorMessage = ref('')



async function startFetch(){
  if( cityInput.value < 3 ) return alert("Shahar nomini kiriting")
  isLoading.value = true
  
  await new Promise( res => setTimeout( res, 1000) )
  const options = {
    method: 'GET',
    url: 'https://weatherapi-com.p.rapidapi.com/current.json',
    params: { q: cityInput.value },
    headers: {
      'X-RapidAPI-Key': 'e89297d4e7msh8203a8ee57ed537p10087ajsne7a82c519ff2',
      'X-RapidAPI-Host': 'weatherapi-com.p.rapidapi.com'
    }
  }
  try {
    const response = await axios.request(options);
    cityInput.value = ''
    weatherData.value = response.data
    fetchValid.value = true
    errorMessage.value = ""

  } catch (error) {
    fetchValid.value = false
    errorMessage.value = error.message
  }
  finally{
    isLoading.value = false
  }
}

async function locationSearch(){
  
  
  if (navigator.geolocation) {
    let name = 'geolocation'
    let checkPermission = await navigator.permissions.query({ name })
    
    if( checkPermission.state != 'granted' ) {
      return alert("Brauzeerdan lokatsiyani olishga ruhsat berilmagan.")
    }
      let position = await new Promise( res => {

      navigator.geolocation.getCurrentPosition( position => {
      const lat = position.coords.latitude
      const long = position.coords.longitude
      res(`${lat},${long}`)
    })
    })
    cityInput.value = position
    return startFetch()
  }
}


</script>

<template>
  <h1>Havo harorati haqida ma'lumot olish. Iltimos shahar nomini kiriting yoki Lokatsiyani olishga ruhsat bering.</h1>
  <br/>
  <div class="body">
    
    
    <button  :disabled="isLoading" class="inputButton" @click="locationSearch">Lokatsiya orqali</button>
    <input class="input" type="text" v-model="cityInput"/>
    <button  :disabled="isLoading" class="inputButton" @click="startFetch">Izlash</button>
    <h2 class="errorMessage" v-if="fetchValid && errorMessage">Kutilmagan xatolik aniqlandi: {{ errorMessage }}</h2>
  </div>
  <div>
    <h2 v-show="isLoading" >Ma'lumot yuklanmoqda</h2>
      <ul v-if="fetchValid">
        <li v-for="(data, key) in renderData" style="list-style: none;">
          {{ `${data}: ${weatherData.location[key] || weatherData.current[key]}` }}
        </li>
      </ul>
    </div>

</template>

<style>

.errorMessage {
  color: red;
}

li {
  font-size: xx-large;
  color: green
}

.body {
  display: flex;
  align-items: center;
  justify-content: center;
}
.input {
  font-size: xx-large;
}

button {
  font-size: xx-large;
  margin-left: 2%;
  margin-right: 2%;
}

</style>