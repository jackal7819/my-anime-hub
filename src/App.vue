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

		myAnime.value.push({
			id: anime.mal_id,
			title: anime.title,
			image: anime.images.jpg.image_url,
			total_episodes: anime.episodes,
			watched_episodes: 0,
		});

		localStorage.setItem('myAnime', JSON.stringify(myAnime.value));
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
		<div
			class="container flex flex-col gap-5 p-10 text-center border rounded-lg border-slate-400"
		>
			<h1 class="text-3xl font-bold underline underline-offset-4">
				My Anime Hub
			</h1>
			<form @submit.prevent="searchAnime">
				<div class="flex justify-center">
					<input
						type="text"
						placeholder="Search anime..."
						v-model="query"
						@input="handleInput"
						class="p-2 text-lg border-2 rounded-l-lg outline-none focus:border-sky-500"
					/>
					<button
						type="submit"
						class="px-4 text-lg text-white duration-300 bg-pink-500 rounded-r-lg hover:bg-pink-600 active:bg-pink-700 disabled:bg-sky-950"
					>
						Search
					</button>
				</div>
			</form>
			<div v-if="searchResults.length">
				<div v-for="anime in searchResults" :key="anime.mal_id">
					<img :src="anime.images.jpg.image_url" alt="anime" />
					<div>
						<h2>{{ anime.title }}</h2>
						<p :title="anime.synopsis" v-if="anime.synopsis">
							{{ anime.synopsis.slice(0, 200) }}...
						</p>
						<span class="flex-1"></span>
						<button @click="addAnime(anime)">Add to my hub</button>
					</div>
				</div>
			</div>
			<div v-if="myAnime.length">
				<h2 class="text-2xl font-bold underline underline-offset-4">
					My Anime
				</h2>
				<div v-for="anime in myAnimeAsc" :key="anime.mal_id">
					<img :src="anime.image" alt="anime" />
					<h3>{{ anime.title }}</h3>
					<span class="flex-1"></span>
					<div>
						{{ anime.watched_episodes }} /
						{{ anime.total_episodes }}
					</div>
					<button
						v-if="anime.watched_episodes !== anime.total_episodes"
						@click="increaseWatch(anime)"
					>
						+
					</button>
					<button
						v-if="anime.watched_episodes > 0"
						@click="decreaseWatch(anime)"
					>
						-
					</button>
				</div>
			</div>
		</div>
	</main>
</template>
