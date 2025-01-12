<script setup lang="ts">
import { onMounted, ref } from "vue";
import { invoke } from "@tauri-apps/api";
import SessionIdModal from "./components/SessionIdModal.vue";

if (
	localStorage.theme === "dark" ||
	(!("theme" in localStorage) && window.matchMedia("(prefers-color-scheme: dark)").matches)
) {
	document.documentElement.classList.add("dark");
} else {
	document.documentElement.classList.remove("dark");
}

// const toggleDarkMode = ref(document.documentElement.className === "dark");
// const changeDarkMode = () => {
// 	toggleDarkMode.value = document.documentElement.classList.toggle("dark");
// 	toggleDarkMode.value ? (localStorage.theme = "dark") : (localStorage.theme = "light");
// };

const periods = [
	{ name: "Dakika", value: "every minute" },
	{ name: "Yarım saat", value: "every half hour" },
	{ name: "Saat", value: "hourly" },
	{ name: "12 saat", value: "half day" },
	{ name: "Gün", value: "daily" },
	{ name: "Hafta", value: "weekly" },
	{ name: "Ay", value: "monthly" },
];

const settings = ref<Settings>({
	movies: true,
	tv: true,
	list_type: "watchlist",
	fetch_period: "daily",
	filter_photos_with_text: false,
	language_of_photos: "en",
	width: 1280,
	height: 720,
});

const response = ref("");
const sessionIdModal = ref(false);

const onSubmit = async () => {
	const res = await invoke("save_settings", {
		settings: JSON.stringify(settings.value),
	});

	response.value = String(res);
	//TODO: show a dialog to the user that they need to restart the app
	if (!settings.value.session_id) {
		sessionIdModal.value = true;
	}
};

onMounted(async () => {
	const res = await invoke("get_settings");
	settings.value = JSON.parse(String(res));

	if (!settings.value.session_id && settings.value.tmdb_api_key) {
		sessionIdModal.value = true;
	}
});
</script>

<template>
	<div class="">
		<SessionIdModal v-if="sessionIdModal" @close="sessionIdModal = false" />
		<h1 class="text-center text-white text-2xl font-semibold mt-4">Uygulama ayarları</h1>
		<form class="mt-5" @submit.prevent="onSubmit">
			<div class="ml-12">
				<label class="block">
					<span class="text-white">TMDb API Key</span>
					<input
						type="password"
						v-model="settings.tmdb_api_key"
						class="bg-gray-500 px-2 py-2 border border-gray-700 text-white rounded text-sm h-7 w-[400px] mt-2 block focus:outline-none focus:border-sky-500 focus:ring-1 focus:ring-sky-500" />
				</label>
				<div class="mt-2">
					<label class="block">
						<span class="text-white">Yenileme sıklığı</span>
						<select
							class="ml-3 bg-gray-500 border border-gray-700 text-gray-900 text-sm rounded-lg focus:ring-blue-500 mb-2">
							<option v-for="period in periods" :value="period.value">
								{{ period.name }}
							</option>
						</select>
					</label>
				</div>
				<div class="mt-2">
					<label class="block">
						<span class="text-white">Hangi türden wallpaper çekmek istiyorsunuz?</span>

						<div class="flex justify-self-center items-center mt-3">
							<input type="checkbox" v-model="settings.movies" class="w-4 h-4" />
							<label class="ml-2 text-sm font-medium text-gray-900 dark:text-gray-300">Filmler</label>

							<input
								type="checkbox"
								v-model="settings.tv"
								class="w-4 h-4 ml-4 text-blue-600 bg-gray-100 rounded border-gray-300 focus:ring-blue-500 dark:focus:ring-blue-600 dark:ring-offset-gray-800 focus:ring-2 dark:bg-gray-700 dark:border-gray-600" />
							<label class="ml-2 text-sm font-medium text-gray-900 dark:text-gray-300">Diziler</label>
						</div>
					</label>
				</div>
				<div class="mt-2">
					<label class="block">
						<span class="text-white">Fotoğraflar hangi listeden alınsın?</span>

						<div class="flex justify-self-center items-center mt-3">
							<input
								type="radio"
								value="watchlist"
								v-model="settings.list_type"
								class="w-4 h-4 text-blue-600 bg-gray-100 rounded border-gray-300 focus:ring-blue-500 dark:focus:ring-blue-600 dark:ring-offset-gray-800 focus:ring-2 dark:bg-gray-700 dark:border-gray-600" />
							<label class="ml-2 text-sm font-medium text-gray-900 dark:text-gray-300">İzleme listesi</label>

							<input
								type="radio"
								value="favorites"
								v-model="settings.list_type"
								class="w-4 h-4 ml-4 text-blue-600 bg-gray-100 rounded border-gray-300 focus:ring-blue-500 dark:focus:ring-blue-600 dark:ring-offset-gray-800 focus:ring-2 dark:bg-gray-700 dark:border-gray-600" />
							<label class="ml-2 text-sm font-medium text-gray-900 dark:text-gray-300">Favoriler</label>
						</div>
					</label>
				</div>

				<div class="mt-2">
					<label class="block">
						<span class="text-white">Üzerinde yazı olan fotoğraflar filtrelensin mi</span>

						<div class="flex justify-self-center items-center mt-3">
							<input
								type="checkbox"
								v-model="settings.filter_photos_with_text"
								class="w-4 h-4 text-blue-600 bg-gray-100 rounded border-gray-300 focus:ring-blue-500 dark:focus:ring-blue-600 dark:ring-offset-gray-800 focus:ring-2 dark:bg-gray-700 dark:border-gray-600" />
							<label class="ml-2 text-sm font-medium text-gray-900 dark:text-gray-300">Filtrele</label>

							<div v-if="!settings.filter_photos_with_text">
								<label class="ml-2 text-sm font-medium text-gray-900 dark:text-gray-300">Yazı Dili</label>
								<input
									v-model="settings.language_of_photos"
									class="ml-4 text-blue-600 bg-gray-100 rounded border-gray-300 focus:ring-blue-500 dark:focus:ring-blue-600 dark:ring-offset-gray-800 focus:ring-2 dark:bg-gray-700 dark:border-gray-600" />
							</div>
						</div>
					</label>
				</div>

				<div class="mt-2">
					<label class="block">
						<span class="text-white">İstenen fotoğraf çözünürlüğü.</span>

						<div class="items-center">
							<label class="text-sm font-medium text-gray-900 dark:text-gray-300">Genişlik</label>
							<input
								type="number"
								v-model="settings.width"
								class="bg-gray-500 px-2 py-2 border border-gray-700 text-white rounded text-sm h-7 w-64 mt-2 block focus:outline-none focus:border-sky-500 focus:ring-1 focus:ring-sky-500" />
							<label class="text-sm font-medium text-gray-900 dark:text-gray-300">Yükseklik</label>

							<input
								type="number"
								v-model="settings.height"
								class="bg-gray-500 px-2 py-2 border border-gray-700 text-white rounded text-sm h-7 w-64 mt-2 block focus:outline-none focus:border-sky-500 focus:ring-1 focus:ring-sky-500" />
						</div>
					</label>
				</div>
				<div class="mt-2">
					<button
						type="submit"
						class="text-white bg-gray-800 hover:bg-gray-900 focus:outline-none focus:ring-4 focus:ring-gray-300 font-medium rounded-lg text-sm px-5 py-2.5 mr-2 mb-2 dark:bg-gray-800 dark:hover:bg-gray-700 dark:focus:ring-gray-700 dark:border-gray-700">
						Kaydet
					</button>
				</div>
			</div>
		</form>
	</div>
</template>
