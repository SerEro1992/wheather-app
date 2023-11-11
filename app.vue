<template>
	<div v-if="city" class="h-screen relative overflow-hidden">
		<img :src="background" class="h-full w-full" />
		<div class="absolute w-full h-full top-0 overlay" />
		<div class="absolute w-full h-full top-0 p-48">
			<div class="flex justify-between">
				<div class="">
					<h1 class="text-white text-7xl">{{ city.name }}</h1>
					<p class="font-extralight text-2xl mt-2 text-white">
						{{ today }}
					</p>
					<img
						class="w-56 icon"
						:src="`https://openweathermap.org/img/wn/${city.weather[0].icon}@4x.png`"
						alt=""
					/>
				</div>
				<div class="">
					<p class="text-9xl text-white font-extralight">
						{{ city.main.temp }}°
					</p>
				</div>
			</div>
			<div class="mt-20 flex items-center">
				<input
					v-model="input"
					type="text"
					class="w-1/2 h-10 p-2"
					placeholder="Введите город..."
				/>
				<button class="bg-sky-400 w-20 h-10 text-white" @click="handleClick">
					Поиск
				</button>
			</div>
		</div>
	</div>
	<div v-else class="p-10">
		<h1 class="text-7xl">Не можем найти город</h1>
		<button class="mt-5 bg-sky-400 px-10 py-5 text-white" @click="goBack">
			Назад
		</button>
	</div>
</template>

<script setup>
const cookie = useCookie('city');

const key = process.env.WEATHER_KEY_SECRET;
console.log(key);

if (!cookie.value) {
	cookie.value = 'Moscow';
}
const search = ref(cookie.value);
const input = ref('');
const background = ref('');

// const { data: city, error } = useFetch(
// 	()=>`https://api.openweathermap.org/data/2.5/weather?q=${search.value}&units=metric&appid=d6c56d799a818462e55956ff2b49724a`
// );

const {
	data: city,
	error,
	refresh,
} = useAsyncData(
	'city',
	async () => {
		let response;
		try {
			response = await $fetch(
				`https://api.openweathermap.org/data/2.5/weather?q=${search.value}&appid=${process.env.WEATHER_KEY_SECRET}`,
				{
					params: {
						units: 'metric',
					},
				}
			);

			cookie.value = search.value;

			const temp = response.main.temp;
			if (temp <= -10) {
				background.value =
					'https://img2.akspic.ru/previews/9/2/3/8/98329/98329-les-derevo-svet-sneg-zamorazhivanie-500x.jpg';
			} else if (temp > -10 && temp <= 0) {
				background.value =
					'https://img3.akspic.ru/previews/3/3/0/1/7/171033/171033-osennij_voshod-osen-zima-priroda-derevo-500x.jpg';
			} else if (temp > 0 && temp < 10) {
				background.value =
					'https://img2.akspic.ru/previews/7/7/5/6/96577/96577-peyzash-list-roshha-osen-priroda-500x.jpg';
			} else {
				background.value =
					'https://img3.akspic.ru/previews/5/9/7/2/0/102795/102795-rastitelnost-leto-bank-nebo-peyzash-500x.jpg';
			}
		} catch (error) {}
		return response;
	},
	{
		watch: [search],
	}
);

let today = new Date().toLocaleDateString('ru-Ru', {
	weekday: 'long',
	year: 'numeric',
	month: 'long',
	day: 'numeric',
});

const handleClick = () => {
	const formatedSearch = input.value.trim().split(' ').join('+');
	search.value = formatedSearch;
	input.value = '';
	// refresh()
};

const goBack = () => {
	search.value = cookie.value;
};
</script>

<style scoped>
.overlay {
	background-color: rgba(0, 0, 0, 0.5);
}
.icon {
	margin-left: -2.75rem;
	margin-top: -2.75rem;
}
</style>
