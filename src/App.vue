<script>
	import axios from 'axios'
	export default {
		data() {
			return {
				city: "Kyiv",
				error: "",
				info: null,
				weatherForecast: [],
				searchQuery: '', // Запит для пошуку
				cities: [],
			}
		},
		computed: {
			cityName() {
				return this.info.name
			},
			showTemp() {
				let temp = this.info.main.temp
				return parseFloat(temp.toFixed(1));
			},
			showWind() {
				return  this.info.wind.speed + " km/h"
			},
			showHumidity() {
				return this.info.main.humidity + "%"
			},
			showPressure() {
				return this.info.main.pressure + " mb"
			},
			getWeatherIcon() {
				if (this.info.weather[0].main == "Clouds") {
					return "../src/assets/images/icons/icon-6.svg"
				} else if (this.info.weather[0].main == "Clear") {
					return "../src/assets/images/icons/icon-2.svg"
				} else if (this.info.weather[0].main == "Rain") {
					return "../src/assets/images/icons/icon-9.svg"
				} else if (this.info.weather[0].main == "Drizzle") {
					return "../src/assets/images/icons/icon-4.svg"
				} else if (this.info.weather[0].main == "Thunderstorm") {
					return "../src/assets/images/icons/icon-12.svg"
				} else if (this.info.weather[0].main == "Snow") {
					return "../src/assets/images/icons/icon-14.svg"
				} else if (this.info.weather[0].main == "Mist") {
					return "../src/assets/images/icons/icon-7.svg"
				} else if (this.info.weather[0].main == "Haze") {
					return "../src/assets/images/icons/icon-7.svg"
				} else if (this.info.weather[0].main == "Fog") {
					return "../src/assets/images/icons/icon-7.svg"
				} else  {
					return "../src/assets/images/icons/icon-8.svg"
				}
			},
			thisDate() {
				const date = new Date();
				return date.toLocaleDateString('en-US', { month: 'long', day: 'numeric', weekday: 'long' })
			},
			backgroundVideo() {
				if (this.info && this.info.weather && this.info.weather[0]) {
						if (this.info.weather[0].main === "Clouds") {
								return "../src/assets/video/Clouds.mp4";
						} else if (this.info.weather[0].main === "Clear") {
								return "../src/assets/video/Clear.mp4";
						} else if (this.info.weather[0].main === "Rain") {
								return "../src/assets/video/Rain.mp4";
						} else if (this.info.weather[0].main === "Drizzle") {
								return "../src/assets/video/Drizzle.mp4";
						} else if (this.info.weather[0].main === "Thunderstorm") {
								return "../src/assets/video/Thunderstorm.mp4";
						} else if (this.info.weather[0].main === "Snow") {
								return "../src/assets/video/Snow.mp4";
						} else if (this.info.weather[0].main === "Mist") {
								return "../src/assets/video/Fog.mp4";
						} else if (this.info.weather[0].main === "Haze") {
								return "../src/assets/video/Haze.mp4";
						} else if (this.info.weather[0].main === "Fog") {
								return "../src/assets/video/Fog.mp4";
						} else {
								return "../src/assets/video/Fog.mp4";
						}
				} else {
						return "";
				}
			}
		},
		mounted() {
			this.searchQuery = 'London';
    	this.getWeather();
		},
		methods: {
			getWeather() {
				if (this.searchQuery.trim().length < 2) {
						this.error = 'Such a city does not exist';
						return false;
				}
				this.error = '';
				axios.get(`https://api.openweathermap.org/data/2.5/weather?q=${this.searchQuery}&units=metric&appid=8e173c68274d8c38e4bf725d2c34ca58`)
				.then(res => {
						this.info = res.data;
				})
				.catch(error => {
						if (error.response && error.response.status === 404) {
								this.error = 'City not found';
						} else {
								this.error = 'An error occurred while fetching data';
						}
				});

				axios.get(`https://api.openweathermap.org/data/2.5/forecast?q=${this.searchQuery}&units=metric&appid=8e173c68274d8c38e4bf725d2c34ca58`)
				.then(res => {
						this.weatherForecast = this.groupByDay(res.data.list);
						this.weekDay();
				})
				.catch(error => {
						if (error.response && error.response.status === 404) {
								this.error = 'City not found';
						} else {
								this.error = 'An error occurred while fetching data';
						}
				});
				const container = this.$refs.container;
					const existingList = container.querySelector('.list');
					if (existingList) {
						container.removeChild(existingList); // Видалення попереднього списку
					}
			},
			weekDay() {
				let ul = this.$refs.ul;
				ul.innerHTML = '';

				this.weatherForecast.forEach(day => {
					const forecastDate = new Date(day[0].dt * 1000); // Дата прогнозу

					// Перевірка, чи поточна дата не співпадає з датою прогнозу
					const today = new Date(); // Поточна дата
					if (forecastDate.getDate() === today.getDate()) {
							return; // Пропускаємо поточний день
					}
					// Створюємо елемент div для прогнозу
					let forecastDiv = document.createElement('div');
					forecastDiv.className = 'forecast';

					// Створюємо блок заголовку прогнозу з днем тижня
					let headerDiv = document.createElement('div');
					headerDiv.className = 'forecast-header';
					let dayDiv = document.createElement('div');
					dayDiv.className = 'day';

					dayDiv.textContent = forecastDate.toLocaleDateString('en-US', { day: 'numeric', month: 'long' });

					headerDiv.appendChild(dayDiv);
					forecastDiv.appendChild(headerDiv);

					// Створюємо блок контенту прогнозу з іконкою погоди, температурою та найвищою та найнижчою температурами
					let contentDiv = document.createElement('div');
					contentDiv.className = 'forecast-content';
					let iconDiv = document.createElement('div');
					iconDiv.className = 'forecast-icon';
					let weatherText = day[0].weather[0].main; // Текстовий опис погоди

					if (weatherText == "Clouds") {
						iconDiv.innerHTML = "<img src='../src/assets/images/icons/icon-6.svg' alt=''>"
					} else if (weatherText == "Clear") {
						iconDiv.innerHTML = "<img src='../src/assets/images/icons/icon-2.svg' alt=''>"
					} else if (weatherText == "Rain") {
						iconDiv.innerHTML = "<img src='../src/assets/images/icons/icon-9.svg' alt=''>"
					} else if (weatherText == "Drizzle") {
						iconDiv.innerHTML = "<img src='../src/assets/images/icons/icon-4.svg' alt=''>"
					} else if (weatherText == "Thunderstorm") {
						iconDiv.innerHTML = "<img src='../src/assets/images/icons/icon-12.svg' alt=''>"
					} else if (weatherText == "Snow") {
						iconDiv.innerHTML = "<img src='../src/assets/images/icons/icon-14.svg' alt=''>"
					} else if (weatherText == "Mist") {
						iconDiv.innerHTML = "<img src='../src/assets/images/icons/icon-7.svg' alt=''>"
					} else if (weatherText == "Haze") {
						iconDiv.innerHTML = "<img src='../src/assets/images/icons/icon-7.svg' alt=''>"
					} else if (weatherText == "Fog") {
						iconDiv.innerHTML = "<img src='../src/assets/images/icons/icon-7.svg' alt=''>"
					} else  {
						iconDiv.innerHTML = "<img src='../src/assets/images/icons/icon-8.svg' alt=''>"
					}
					// iconDiv.textContent = weatherText;
					contentDiv.appendChild(iconDiv);

					// Визначаємо найвищу та найнижчу температури для поточного дня
					let highestTemp = -Infinity;
					let lowestTemp = Infinity;
					day.forEach(item => {
						const temperature = item.main.temp;
						if (temperature > highestTemp) {
							highestTemp = temperature;
						}
						if (temperature < lowestTemp) {
							lowestTemp = temperature;
						}
					});

					// Додаємо блок з температурою та найвищою та найнижчою температурами до контенту прогнозу
					let degreeDiv = document.createElement('div');
					degreeDiv.className = 'degree';
					degreeDiv.innerHTML = `${highestTemp}<sup>o</sup>C`;
					contentDiv.appendChild(degreeDiv);
					let smallDiv = document.createElement('small');
					smallDiv.innerHTML = `${lowestTemp}<sup>o</sup>C`;
					contentDiv.appendChild(smallDiv);

					// Додаємо блок контенту до прогнозу
					forecastDiv.appendChild(contentDiv);

					// Додаємо прогноз до списку
					ul.appendChild(forecastDiv);
				});
			},
			groupByDay(forecastList) {
				const grouped = {};
				forecastList.forEach(item => {
					const date = new Date(item.dt * 1000);
					const day = date.toDateString(); // Групуємо за датою (без часу)
					if (!grouped[day]) {
						grouped[day] = [];
					}
					grouped[day].push(item);
				});
				return Object.values(grouped); // Повертаємо масив масивів, де кожен масив містить прогноз погоди для певного дня
			},
			searchCities() {
				axios.get('https://secure.geonames.org/searchJSON', {
						params: {
								name_startsWith: this.searchQuery,
								maxRows: 5,
								username: 'rydolif'
						}
				})
				.then(response => {
					const allCities = response.data.geonames;
					this.cities = [];

					allCities.forEach(city => {
							if (!this.cities.some(item => item.name === city.name)) {
									this.cities.push(city);
							}
					});

					// Видалення попереднього списку
					const container = this.$refs.container;
					const existingList = container.querySelector('.list');
					if (existingList) {
						container.removeChild(existingList); 
					}

					const ul = document.createElement('ul');
					ul.classList.add('list');

					this.cities.forEach(city => {
						const li = document.createElement('li');
						li.classList.add('list__item');
						li.textContent = city.name;

						// Додавання обробника подій кліку для встановлення значення searchQuery
						li.addEventListener('click', () => {
							this.searchQuery = city.name; // Встановлення значення searchQuery
							container.removeChild(ul); // Видалення списку ul
							this.getWeather();
						});

						ul.appendChild(li); // Додавання li до ul
					});

					container.appendChild(ul); 
				})
				.catch(error => {
						console.error('Помилка при отриманні даних:', error);
				});
			},
			selectCity(city) {
					this.searchQuery = city.name;
					this.getWeather();
			}
		},

	}
</script>

<template>
	<video :src="backgroundVideo" autoplay loop muted></video>

	<div class="hero">
		<div class="container">
			<form action="#" class="find-location">
				<input v-model="searchQuery" @input="searchCities" @keydown.enter.prevent="getWeather" type="text" placeholder="Find your location..." value="London">

				<div ref="container"></div>

				<button v-if="city != ''" @click="getWeather()" type="button">Find</button>
				<button disabled v-else>Enter the name of the city</button>
			</form>
			<p class="error" v-if="error">{{ error }}</p>
		</div>
	</div>

	<div v-if="info != null" class="forecast-table">
		<div class="container">
			<div class="forecast-container"  >

				<div class="today forecast">
					<div class="forecast-header">
						<div class="date">{{ thisDate }}</div>
					</div>
					<div class="forecast-content">
						<div class="location">{{ cityName }}</div>
						<div class="degree">
							<div class="num">{{ showTemp }}<sup>o</sup>C</div>
							<div class="forecast-icon">
								<img :src="getWeatherIcon" alt="" width=90>
							</div>	
						</div>
						<span><img src="../src/assets/images/icon-umberella.png" alt="">{{ showHumidity }}</span>
						<span><img src="../src/assets/images/icon-wind.png" alt="">{{ showWind }}</span>
						<span><img src="../src/assets/images/icon-compass.png" alt="">{{ showPressure }}</span>
					</div>
				</div>

				<div ref="ul" class="wrapper"></div>

			</div>
		</div>
	</div>

</template>

<style scoped>

</style>
