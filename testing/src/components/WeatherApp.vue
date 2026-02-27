<script setup lang="ts">
import { ref, computed } from 'vue'

// =============================================================================
// SHADCN-VUE COMPONENTS
// =============================================================================
// Deze componenten komen van shadcn-vue en zijn al gestyled met Tailwind CSS
// Je importeert ze en gebruikt ze als normale Vue componenten
import { Card, CardContent, CardHeader, CardTitle, CardDescription } from '@/components/ui/card'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Badge } from '@/components/ui/badge'
import { Skeleton } from '@/components/ui/skeleton'

// =============================================================================
// TYPESCRIPT INTERFACE
// =============================================================================
interface WeatherData {
  name: string
  sys: { country: string }
  main: {
    temp: number
    humidity: number
    feels_like: number
    temp_min: number
    temp_max: number
  }
  weather: Array<{
    main: string
    description: string
    icon: string
  }>
  wind: { speed: number }
}

// =============================================================================
// WORLD CITIES DATABASE
// =============================================================================
// Een lijst met steden van over de hele wereld voor random suggesties
const worldCities = [
  // Europa
  'Paris', 'Berlin', 'Madrid', 'Rome', 'Vienna', 'Prague', 'Warsaw', 'Budapest',
  'Athens', 'Lisbon', 'Dublin', 'Stockholm', 'Oslo', 'Copenhagen', 'Helsinki',
  'Brussels', 'Zurich', 'Barcelona', 'Milan', 'Munich', 'Amsterdam',
  // Azië
  'Tokyo', 'Beijing', 'Shanghai', 'Seoul', 'Bangkok', 'Singapore', 'Hong Kong',
  'Mumbai', 'Delhi', 'Dubai', 'Istanbul', 'Jakarta', 'Manila', 'Kuala Lumpur',
  'Taipei', 'Osaka', 'Kyoto', 'Ho Chi Minh City', 'Hanoi', 'Bali',
  // Noord-Amerika
  'New York', 'Los Angeles', 'Chicago', 'Miami', 'San Francisco', 'Las Vegas',
  'Toronto', 'Vancouver', 'Montreal', 'Mexico City', 'Seattle', 'Boston',
  'Washington', 'Denver', 'Phoenix', 'Houston', 'Dallas', 'Atlanta',
  // Zuid-Amerika
  'São Paulo', 'Rio de Janeiro', 'Buenos Aires', 'Lima', 'Bogotá', 'Santiago',
  'Caracas', 'Montevideo', 'Quito', 'Medellín',
  // Afrika
  'Cairo', 'Cape Town', 'Johannesburg', 'Nairobi', 'Lagos', 'Casablanca',
  'Marrakech', 'Tunis', 'Accra', 'Addis Ababa',
  // Oceanië
  'Sydney', 'Melbourne', 'Auckland', 'Brisbane', 'Perth', 'Wellington',
  'Fiji', 'Honolulu', 'Christchurch', 'Adelaide',
  // Exotische plekken
  'Reykjavik', 'Havana', 'Kathmandu', 'Bhutan', 'Zanzibar', 'Madagascar',
  'Maldives', 'Seychelles', 'Mauritius', 'Tahiti'
]

// =============================================================================
// REACTIVE STATE
// =============================================================================
const city = ref('')
const weather = ref<WeatherData | null>(null)
const loading = ref(false)
const error = ref('')
const randomSuggestions = ref<string[]>([])  // Houdt de random stad suggesties bij

const API_KEY = '9d0ed650c88bd1351c5e21dc91663227'

// =============================================================================
// RANDOM CITIES FUNCTION
// =============================================================================
// Pakt X aantal random steden uit de lijst (zonder duplicaten)
function getRandomCities(count: number = 5): string[] {
  const shuffled = [...worldCities].sort(() => Math.random() - 0.5)
  return shuffled.slice(0, count)
}

// Vernieuw de random suggesties
function refreshSuggestions() {
  randomSuggestions.value = getRandomCities(5)
}

// Start met random suggesties
refreshSuggestions()

// =============================================================================
// COMPUTED PROPERTIES
// =============================================================================
// Computed = berekende waarde die automatisch update als dependencies veranderen
// Dit bepaalt welke achtergrondkleur de weather card krijgt op basis van temperatuur
const weatherGradient = computed(() => {
  if (!weather.value) return 'from-violet-600 to-indigo-600'
  const temp = weather.value.main.temp
  if (temp < 0) return 'from-cyan-500 to-blue-600'      // Koud: blauw
  if (temp < 10) return 'from-blue-500 to-indigo-600'   // Fris: indigo
  if (temp < 20) return 'from-indigo-500 to-purple-600' // Mild: paars
  if (temp < 30) return 'from-orange-500 to-pink-600'   // Warm: oranje/roze
  return 'from-red-500 to-orange-600'                   // Heet: rood
})

// Emoji gebaseerd op weer type
const weatherEmoji = computed(() => {
  if (!weather.value || !weather.value.weather[0]) return '🌤️'
  const main = weather.value.weather[0].main.toLowerCase()
  const emojis: Record<string, string> = {
    clear: '☀️',
    clouds: '☁️',
    rain: '🌧️',
    drizzle: '🌦️',
    thunderstorm: '⛈️',
    snow: '❄️',
    mist: '🌫️',
    fog: '🌫️',
    haze: '🌫️'
  }
  return emojis[main] || '🌤️'
})

// =============================================================================
// API CALL
// =============================================================================
async function fetchWeather() {
  if (!city.value.trim()) {
    error.value = 'Voer een stad in'
    return
  }

  loading.value = true
  error.value = ''
  weather.value = null

  try {
    const response = await fetch(
      `https://api.openweathermap.org/data/2.5/weather?q=${city.value}&appid=${API_KEY}&units=metric`
    )

    if (!response.ok) {
      if (response.status === 404) throw new Error('Stad niet gevonden')
      if (response.status === 401) throw new Error('Ongeldige API key')
      throw new Error('Er ging iets mis')
    }

    const data: WeatherData = await response.json()
    weather.value = data
    refreshSuggestions()  // Nieuwe random steden na elke zoekopdracht!
  } catch (err) {
    error.value = err instanceof Error ? err.message : 'Er ging iets mis'
  } finally {
    loading.value = false
  }
}

// Zoek naar een specifieke stad (voor de knoppen)
function searchCity(cityName: string) {
  city.value = cityName
  fetchWeather()
}

function getWeatherIcon(icon: string): string {
  return `https://openweathermap.org/img/wn/${icon}@4x.png`
}
</script>

<template>
  <div class="w-full max-w-md space-y-6">
    
    <!-- =================================================================== -->
    <!-- HEADER MET ANIMATIES                                                -->
    <!-- =================================================================== -->
    <!-- 
      Tailwind classes uitgelegd:
      - text-4xl: font-size 2.25rem
      - font-bold: font-weight 700
      - text-center: text-align center
      - mb-2: margin-bottom 0.5rem
      - glow-text: custom class uit style.css voor glow effect
      - float: custom class voor zweef animatie
    -->
    <div class="text-center">
      <h1 class="text-4xl font-bold mb-2 glow-text float">
        {{ weatherEmoji }} Weather App
      </h1>
      <p class="text-muted-foreground text-sm">
        Zoek het weer in elke stad ter wereld
      </p>
    </div>

    <!-- =================================================================== -->
    <!-- SEARCH CARD MET GLASSMORPHISM                                       -->
    <!-- =================================================================== -->
    <!-- 
      Card is een shadcn component
      We voegen extra classes toe:
      - glass: custom glassmorphism effect
      - glow: custom glow shadow
      - transition-all duration-300: smooth animatie bij hover
      - hover:scale-[1.02]: wordt iets groter bij hover
    -->
    <Card class="glass glow transition-all duration-300 hover:scale-[1.02]">
      <CardHeader>
        <CardTitle class="text-lg">🔍 Zoek een stad</CardTitle>
        <CardDescription>
          Typ een stadsnaam en klik op zoeken
        </CardDescription>
      </CardHeader>
      <CardContent>
        <!-- 
          @submit.prevent voorkomt page refresh
          flex gap-2: flexbox met 0.5rem gap tussen items
        -->
        <form @submit.prevent="fetchWeather" class="flex gap-2">
          <!-- 
            Input is een shadcn component
            v-model linkt de waarde aan onze 'city' ref
            Extra styling met Tailwind classes
          -->
          <Input
            v-model="city"
            type="text"
            placeholder="Amsterdam, London, Tokyo..."
            class="flex-1 bg-white/5 border-white/10 focus:border-primary 
                   transition-all duration-300 focus:ring-2 focus:ring-primary/50"
          />
          <!-- 
            Button is een shadcn component
            :disabled maakt de button inactive tijdens loading
            class binding voor conditional styling
          -->
          <Button 
            type="submit" 
            :disabled="loading"
            class="bg-gradient-to-r from-violet-600 to-indigo-600 
                   hover:from-violet-500 hover:to-indigo-500
                   transition-all duration-300 hover:scale-105 hover:shadow-lg
                   disabled:opacity-50 disabled:cursor-not-allowed"
          >
            <!-- Conditional rendering: toon spinner of tekst -->
            <span v-if="loading" class="flex items-center gap-2">
              <!-- Animated spinner met Tailwind -->
              <svg class="animate-spin h-4 w-4" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none"/>
                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z"/>
              </svg>
              Laden...
            </span>
            <span v-else>Zoeken</span>
          </Button>
        </form>
      </CardContent>
    </Card>

    <!-- =================================================================== -->
    <!-- ERROR MESSAGE                                                       -->
    <!-- =================================================================== -->
    <!-- 
      v-if="error" toont dit alleen als er een error is
      animate-in fade-in slide-in-from-top: shadcn animatie classes
    -->
    <Card v-if="error" class="border-destructive/50 bg-destructive/10 animate-in fade-in slide-in-from-top-2">
      <CardContent class="pt-6">
        <p class="text-destructive text-center font-medium">
          ❌ {{ error }}
        </p>
      </CardContent>
    </Card>

    <!-- =================================================================== -->
    <!-- LOADING SKELETON                                                    -->
    <!-- =================================================================== -->
    <!-- 
      Skeleton is een shadcn component die een "loading placeholder" toont
      Dit geeft gebruikers feedback dat er iets aan het laden is
    -->
    <Card v-if="loading" class="glass overflow-hidden">
      <CardContent class="pt-6 space-y-4">
        <div class="flex items-center justify-between">
          <Skeleton class="h-8 w-32 bg-white/10" />
          <Skeleton class="h-16 w-16 rounded-full bg-white/10" />
        </div>
        <Skeleton class="h-16 w-40 bg-white/10" />
        <Skeleton class="h-4 w-48 bg-white/10" />
        <div class="grid grid-cols-3 gap-4 pt-4">
          <Skeleton class="h-16 bg-white/10" />
          <Skeleton class="h-16 bg-white/10" />
          <Skeleton class="h-16 bg-white/10" />
        </div>
      </CardContent>
    </Card>

    <!-- =================================================================== -->
    <!-- WEATHER RESULT CARD                                                 -->
    <!-- =================================================================== -->
    <!-- 
      Dit is de main weather card met dynamische gradient
      :class bindt de computed weatherGradient
      animate-in fade-in slide-in-from-bottom: entry animatie
    -->
    <Card 
      v-if="weather && !loading" 
      :class="[
        'overflow-hidden transition-all duration-500',
        'animate-in fade-in slide-in-from-bottom-4',
        'bg-gradient-to-br', weatherGradient,
        'border-0 shadow-2xl pulse-glow'
      ]"
    >
      <CardContent class="pt-6 text-white">
        <!-- Header met stad en icon -->
        <div class="flex items-start justify-between mb-4">
          <div>
            <!-- Badge is een shadcn component voor labels/tags -->
            <Badge variant="secondary" class="mb-2 bg-white/20 hover:bg-white/30">
              {{ weather.sys.country }}
            </Badge>
            <h2 class="text-3xl font-bold">{{ weather.name }}</h2>
            <p class="text-white/80 capitalize">{{ weather.weather[0]?.description }}</p>
          </div>
          <!-- Weather icon met float animatie -->
          <img
            :src="getWeatherIcon(weather.weather[0]?.icon ?? '01d')"
            :alt="weather.weather[0]?.description ?? 'weather'"
            class="w-24 h-24 float drop-shadow-2xl"
          />
        </div>

        <!-- Grote temperatuur display -->
        <div class="text-7xl font-bold mb-6 tracking-tight">
          {{ Math.round(weather.main.temp) }}°
          <span class="text-2xl font-normal text-white/70">C</span>
        </div>

        <!-- Min/Max temperatuur -->
        <div class="flex gap-4 mb-6 text-sm">
          <span class="flex items-center gap-1">
            <span class="text-white/60">↑</span>
            {{ Math.round(weather.main.temp_max) }}°
          </span>
          <span class="flex items-center gap-1">
            <span class="text-white/60">↓</span>
            {{ Math.round(weather.main.temp_min) }}°
          </span>
        </div>

        <!-- Detail grid met glassmorphism cards -->
        <div class="grid grid-cols-3 gap-3">
          <!-- 
            Elk detail item is een mini-card met:
            - bg-white/10: semi-transparante witte achtergrond
            - backdrop-blur-sm: blur effect
            - rounded-xl: afgeronde hoeken
            - p-3: padding
            - text-center: gecentreerde tekst
            - transition hover:bg-white/20: hover effect
          -->
          <div class="bg-white/10 backdrop-blur-sm rounded-xl p-3 text-center 
                      transition-all duration-300 hover:bg-white/20 hover:scale-105">
            <div class="text-2xl mb-1">🌡️</div>
            <div class="text-xs text-white/60 mb-1">Voelt als</div>
            <div class="font-semibold">{{ Math.round(weather.main.feels_like) }}°C</div>
          </div>
          
          <div class="bg-white/10 backdrop-blur-sm rounded-xl p-3 text-center
                      transition-all duration-300 hover:bg-white/20 hover:scale-105">
            <div class="text-2xl mb-1">💧</div>
            <div class="text-xs text-white/60 mb-1">Vochtigheid</div>
            <div class="font-semibold">{{ weather.main.humidity }}%</div>
          </div>
          
          <div class="bg-white/10 backdrop-blur-sm rounded-xl p-3 text-center
                      transition-all duration-300 hover:bg-white/20 hover:scale-105">
            <div class="text-2xl mb-1">💨</div>
            <div class="text-xs text-white/60 mb-1">Wind</div>
            <div class="font-semibold">{{ weather.wind.speed }} m/s</div>
          </div>
        </div>
      </CardContent>
    </Card>

    <!-- =================================================================== -->
    <!-- RANDOM CITY SUGGESTIONS (na zoekresultaat)                          -->
    <!-- =================================================================== -->
    <!-- 
      Dit verschijnt alleen als er weer data is (na een zoekopdracht)
      Elke keer krijg je 5 nieuwe random steden van over de wereld
    -->
    <Card v-if="weather && !loading" class="glass animate-in fade-in slide-in-from-bottom-4 delay-150">
      <CardContent class="pt-6">
        <div class="flex items-center justify-between mb-4">
          <div>
            <h3 class="font-semibold text-lg">🌍 Ontdek meer steden</h3>
            <p class="text-sm text-muted-foreground">Random steden van over de wereld</p>
          </div>
          <!-- Refresh knop voor nieuwe random steden -->
          <Button 
            variant="ghost" 
            size="sm"
            @click="refreshSuggestions"
            class="hover:bg-white/10 transition-all duration-300 hover:rotate-180"
          >
            🔄
          </Button>
        </div>
        
        <!-- Grid met random stad buttons -->
        <div class="flex flex-wrap gap-2">
          <Button
            v-for="(suggestion, index) in randomSuggestions"
            :key="suggestion"
            variant="outline"
            size="sm"
            @click="searchCity(suggestion)"
            :class="[
              'bg-white/5 border-white/10 hover:bg-white/15',
              'transition-all duration-300 hover:scale-105 hover:border-primary/50',
              'animate-in fade-in zoom-in',
            ]"
            :style="{ animationDelay: `${index * 50}ms` }"
          >
            {{ suggestion }}
          </Button>
        </div>
      </CardContent>
    </Card>

    <!-- =================================================================== -->
    <!-- INITIAL STATE / INSTRUCTIONS                                        -->
    <!-- =================================================================== -->
    <Card v-if="!weather && !loading && !error" class="glass text-center">
      <CardContent class="pt-6 space-y-4">
        <div class="text-6xl float">🌍</div>
        <p class="text-muted-foreground">
          Voer een stad in om het weer te bekijken
        </p>
        
        <!-- Random stad suggesties -->
        <div class="space-y-3">
          <div class="flex items-center justify-center gap-2 text-sm text-muted-foreground">
            <span>Of kies een random stad</span>
            <Button 
              variant="ghost" 
              size="sm"
              @click="refreshSuggestions"
              class="h-6 w-6 p-0 hover:bg-white/10 transition-all duration-300 hover:rotate-180"
            >
              🔄
            </Button>
          </div>
          
          <div class="flex flex-wrap justify-center gap-2">
            <Button 
              v-for="(suggestion, index) in randomSuggestions"
              :key="suggestion"
              variant="outline"
              size="sm"
              @click="searchCity(suggestion)"
              :class="[
                'bg-white/5 border-white/10 hover:bg-white/15',
                'transition-all duration-300 hover:scale-105 hover:border-primary/50',
                'animate-in fade-in zoom-in'
              ]"
              :style="{ animationDelay: `${index * 50}ms` }"
            >
              {{ suggestion }}
            </Button>
          </div>
        </div>
      </CardContent>
    </Card>

  </div>
</template>
