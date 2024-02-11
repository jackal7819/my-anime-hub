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
			watchedEpisodes: 0,
		});
	};

	localStorage.setItem('myAnime', JSON.stringify(myAnime.value));

	const increaseWatch = (anime) => {
		anime.watchedEpisodes++;
		localStorage.setItem('myAnime', JSON.stringify(myAnime.value));
	};
	const decreaseWatch = (anime) => {
		anime.watchedEpisodes--;
		localStorage.setItem('myAnime', JSON.stringify(myAnime.value));
	};
	
	onMounted(() => {
		myAnime.value = JSON.parse(localStorage.getItem('myAnime')) || [];
	});
</script>

<template>
	<h1 class="text-3xl font-bold underline">Hello world!</h1>
</template>
