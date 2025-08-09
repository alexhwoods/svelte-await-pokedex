<script context="module" lang="ts">
	interface EvolutionChain {
		species: {
			name: string;
		};
		evolves_to: EvolutionChain[];
	}

	interface PokemonStat {
		base_stat: number;
		stat: {
			name: string;
		};
	}

	function getTypeColor(type: string) {
		const colors: Record<string, string> = {
			normal: '#A8A878',
			fighting: '#C03028',
			flying: '#A890F0',
			poison: '#A040A0',
			ground: '#E0C068',
			rock: '#B8A038',
			bug: '#A8B820',
			ghost: '#705898',
			steel: '#B8B8D0',
			fire: '#F08030',
			water: '#6890F0',
			grass: '#78C850',
			electric: '#F8D030',
			psychic: '#F85888',
			ice: '#98D8D8',
			dragon: '#7038F8',
			dark: '#705848',
			fairy: '#EE99AC'
		};
		return colors[type] || '#68A090';
	}

	function parseEvolutionChain(chain: EvolutionChain) {
		const result = [];
		let current = chain;

		while (current) {
			result.push(current.species.name);
			current = current.evolves_to[0];
		}

		return result;
	}
</script>

<script lang="ts">
	// Fetch Pokémon data from the API
	async function fetchPokemon(id: number) {
		const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${id}`);
		if (!response.ok) throw new Error('Pokémon not found');
		return response.json();
	}

	// Fetch Pokémon species data for additional info
	async function fetchPokemonSpecies(id: number) {
		const response = await fetch(`https://pokeapi.co/api/v2/pokemon-species/${id}`);
		if (!response.ok) throw new Error('Species data not found');
		return response.json();
	}

	// Fetch evolution chain
	async function fetchEvolutionChain(url: string) {
		const response = await fetch(url);
		if (!response.ok) throw new Error('Evolution data not found');
		return response.json();
	}

	// Calculate total stats
	async function calculateStats(pokemonId: number) {
		const pokemon = await fetchPokemon(pokemonId);
		const total = pokemon.stats.reduce((sum: number, stat: PokemonStat) => sum + stat.base_stat, 0);
		return { total, average: Math.round(total / pokemon.stats.length) };
	}

	let pokemonId = $state(25); // Pikachu by default
	let showEvolution = $state(false);
	let showStats = $state(false);
</script>

<svelte:boundary>
	{#snippet pending()}
		<div class="flex min-h-screen items-center justify-center">
			<div class="text-center">
				<div class="mx-auto h-12 w-12 animate-spin rounded-full border-b-2 border-yellow-500"></div>
				<p class="mt-4 text-gray-600">Loading Pokémon data...</p>
			</div>
		</div>
	{/snippet}

	<div class="container mx-auto p-8">
		<h1 class="mb-6 text-3xl font-bold">Pokémon API with Svelte Await Expressions</h1>

		<!-- Pokémon selector -->
		<div class="mb-6 rounded-lg bg-white p-6 shadow-md">
			<h2 class="mb-4 text-xl font-semibold">Choose a Pokémon</h2>
			<div class="mb-4">
				<label class="mb-2 block">
					<span class="text-gray-700">Pokémon ID:</span>
					<input
						type="number"
						bind:value={pokemonId}
						min="1"
						max="1025"
						class="mt-1 block w-32 rounded-md border-gray-300 shadow-sm"
					/>
				</label>
				<p class="text-sm text-gray-500">Try: 25 (Pikachu), 1 (Bulbasaur), 150 (Mewtwo)</p>
			</div>
		</div>

		<!-- Main Pokémon data -->
		{#await fetchPokemon(pokemonId)}
			<div class="rounded-lg bg-white p-6 shadow-md">
				<p class="text-gray-500">Loading Pokémon...</p>
			</div>
		{:then pokemon}
			<div class="mb-6 rounded-lg bg-white p-6 shadow-md">
				<h2 class="mb-4 text-xl font-semibold capitalize">{pokemon.name}</h2>
				<div class="grid grid-cols-1 gap-4 md:grid-cols-2">
					<div>
						<img
							src={pokemon.sprites.other['official-artwork'].front_default}
							alt={pokemon.name}
							class="mx-auto h-48 w-48"
						/>
					</div>
					<div>
						<p><strong>Height:</strong> {pokemon.height / 10}m</p>
						<p><strong>Weight:</strong> {pokemon.weight / 10}kg</p>
						<p><strong>Base Experience:</strong> {pokemon.base_experience}</p>
						<div class="mt-2">
							<strong>Types:</strong>
							<div class="mt-1 flex gap-2">
								{#each pokemon.types as type (type.slot)}
									<span
										class="rounded-full px-3 py-1 text-sm text-white"
										style="background-color: {getTypeColor(type.type.name)}"
									>
										{type.type.name}
									</span>
								{/each}
							</div>
						</div>
						<div class="mt-2">
							<strong>Abilities:</strong>
							<ul class="mt-1 list-inside list-disc">
								{#each pokemon.abilities as ability (ability.slot)}
									<li class="capitalize">
										{ability.ability.name.replace('-', ' ')}
										{#if ability.is_hidden}
											<span class="text-sm text-gray-500">(Hidden)</span>
										{/if}
									</li>
								{/each}
							</ul>
						</div>
					</div>
				</div>
			</div>
		{:catch error}
			<div class="rounded-lg bg-red-50 p-6 shadow-md">
				<p class="text-red-500">Error: {error.message}</p>
			</div>
		{/await}

		<!-- Stats section -->
		<button
			class="mb-4 rounded bg-blue-500 px-4 py-2 font-bold text-white hover:bg-blue-700"
			onclick={() => (showStats = !showStats)}
		>
			{showStats ? 'Hide' : 'Show'} Stats Analysis
		</button>

		{#if showStats}
			<div class="mb-6 rounded-lg bg-white p-6 shadow-md">
				<h2 class="mb-4 text-xl font-semibold">Stats Analysis</h2>
				{#await Promise.all([fetchPokemon(pokemonId), calculateStats(pokemonId)])}
					<p class="text-gray-500">Calculating stats...</p>
				{:then [pokemon, statsInfo]}
					<div class="grid gap-4">
						{#each pokemon.stats as stat (stat.stat.name)}
							<div>
								<div class="flex justify-between">
									<span class="capitalize">{stat.stat.name.replace('-', ' ')}</span>
									<span>{stat.base_stat}</span>
								</div>
								<div class="mt-1 h-4 w-full rounded-full bg-gray-200">
									<div
										class="h-4 rounded-full bg-green-500"
										style="width: {(stat.base_stat / 255) * 100}%"
									></div>
								</div>
							</div>
						{/each}
						<div class="mt-4 border-t pt-4">
							<p><strong>Total:</strong> {statsInfo.total}</p>
							<p><strong>Average:</strong> {statsInfo.average}</p>
						</div>
					</div>
				{:catch error}
					<p class="text-red-500">Error loading stats: {error.message}</p>
				{/await}
			</div>
		{/if}

		<!-- Evolution chain -->
		<button
			class="mb-4 rounded bg-purple-500 px-4 py-2 font-bold text-white hover:bg-purple-700"
			onclick={() => (showEvolution = !showEvolution)}
		>
			{showEvolution ? 'Hide' : 'Show'} Evolution Chain
		</button>

		{#if showEvolution}
			<div class="rounded-lg bg-white p-6 shadow-md">
				<h2 class="mb-4 text-xl font-semibold">Evolution Chain</h2>
				{#await fetchPokemonSpecies(pokemonId)}
					<p class="text-gray-500">Loading evolution data...</p>
				{:then species}
					{#await fetchEvolutionChain(species.evolution_chain.url)}
						<p class="text-gray-500">Fetching evolution chain...</p>
					{:then evolutionData}
						{@const chain = parseEvolutionChain(evolutionData.chain)}
						<div class="flex items-center gap-4">
							{#each chain as pokemon, i (pokemon)}
								<div class="text-center">
									<p class="capitalize">{pokemon}</p>
								</div>
								{#if i < chain.length - 1}
									<span class="text-2xl">→</span>
								{/if}
							{/each}
						</div>
					{:catch error}
						<p class="text-red-500">Error loading evolution chain: {error.message}</p>
					{/await}
				{:catch error}
					<p class="text-red-500">Error loading species data: {error.message}</p>
				{/await}
			</div>
		{/if}
	</div>
</svelte:boundary>
