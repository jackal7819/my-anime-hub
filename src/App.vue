<script setup>
	import { ref, computed, onMounted } from 'vue';

	const query = ref('');
	const myAnime = ref([]);
	const searchResults = ref([]);

	const myAnimeAsc = computed(() => {
		return myAnime.value.sort((a, b) => {
			return a.title.localeCompare(b.title);
		});
	});

	const searchAnime = async () => {
		try {
			const response = await fetch(
				`https://api.jikan.moe/v4/anime?q=${query.value}`
			);
			const { data } = await response.json();
			searchResults.value = data;
		} catch (error) {
			console.log(error);
		}
	};

	const handleInput = (event) => {
		if (!event.target.value) {
			searchResults.value = [];
		}
	};

	const addAnime = (anime) => {
		searchResults.value = [];
		query.value = '';

		const animeExists = myAnime.value.some(
			(item) => item.id === anime.mal_id
		);

		if (!animeExists) {
			myAnime.value.push({
				id: anime.mal_id,
				title: anime.title,
				image: anime.images.jpg.image_url,
				total_episodes: anime.episodes,
				watched_episodes: 0,
			});

			localStorage.setItem('myAnime', JSON.stringify(myAnime.value));
		}
	};

	const increaseWatch = (anime) => {
		anime.watched_episodes++;
		localStorage.setItem('myAnime', JSON.stringify(myAnime.value));
	};
	const decreaseWatch = (anime) => {
		anime.watched_episodes--;
		localStorage.setItem('myAnime', JSON.stringify(myAnime.value));
	};

	onMounted(() => {
		myAnime.value = JSON.parse(localStorage.getItem('myAnime')) || [];
	});
</script>

<template>
	<main
		class="flex items-center justify-center min-h-screen antialiased text-slate-400 bg-slate-900"
	>
		<div class="container flex flex-col gap-10 p-10">
			<h1
				class="text-3xl font-bold text-center underline underline-offset-4"
			>
				My Anime Hub
			</h1>
			<form @submit.prevent="searchAnime">
				<div class="flex justify-center">
					<input
						type="text"
						placeholder="Search anime..."
						v-model="query"
						@input="handleInput"
						class="p-2 text-lg border-2 rounded-l-lg outline-none focus:border-sky-500 w-52 sm:w-72"
					/>
					<button
						type="submit"
						class="px-4 text-lg text-white duration-300 bg-pink-500 rounded-r-lg hover:bg-pink-600 active:bg-pink-700 disabled:bg-sky-950"
					>
						Search
					</button>
				</div>
			</form>
			<div
				v-if="searchResults.length"
				class="grid gap-10 lg:grid-cols-2 2xl:grid-cols-3"
			>
				<div
					v-for="anime in searchResults"
					:key="anime.mal_id"
					class="flex flex-col items-center w-full gap-5 sm:flex-row sm:items-stretch"
				>
					<img
						:src="anime.images.jpg.image_url"
						alt="anime"
						class="rounded w-[225px] h-[318px] object-cover"
					/>
					<div class="flex flex-col justify-end gap-5">
						<h2 class="text-xl">{{ anime.title.slice(0, 44) }}</h2>
						<p :title="anime.synopsis">
							{{
								anime.synopsis
									? anime.synopsis.slice(0, 120)
									: 'It is a famous Japanese anime watched by thousands of viewers around the world'
							}}...
						</p>
						<span class="flex-1"></span>
						<button
							@click="addAnime(anime)"
							class="py-2 text-white duration-300 rounded bg-sky-500 hover:bg-sky-600"
						>
							Add to my hub
						</button>
					</div>
				</div>
			</div>
			<h2
				v-if="myAnime.length"
				class="text-2xl font-bold text-center underline underline-offset-4"
			>
				My Anime
			</h2>
			<div
				v-if="myAnime.length"
				class="grid gap-10 lg:grid-cols-3 2xl:grid-cols-5 xl:grid-cols-4 md:grid-cols-2 place-items-center"
			>
				<div
					v-for="anime in myAnimeAsc"
					:key="anime.mal_id"
					class="flex flex-col justify-end gap-5 text-center"
				>
					<img
						:src="anime.image"
						alt="anime"
						class="rounded w-[225px] h-[318px] object-cover"
					/>
					<span class="flex-1"></span>
					<h3 class="text-lg">{{ anime.title.slice(0, 24) }}</h3>
					<div class="flex items-center justify-center">
						<div class="w-20 mr-5 text-lg">
							{{ anime.watched_episodes }} /
							{{ anime.total_episodes }}
						</div>
						<button
							:disabled="
								anime.watched_episodes === anime.total_episodes
							"
							@click="increaseWatch(anime)"
							class="px-4 text-xl text-white duration-300 rounded-l-lg bg-sky-500 hover:bg-sky-600 disabled:bg-slate-600"
						>
							+
						</button>
						<button
							:disabled="anime.watched_episodes === 0"
							@click="decreaseWatch(anime)"
							class="px-4 text-xl text-white duration-300 bg-pink-500 rounded-r-lg hover:bg-pink-600 disabled:bg-slate-600"
						>
							-
						</button>
					</div>
				</div>
			</div>
		</div>
	</main>
</template>
