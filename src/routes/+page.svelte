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
		<div class="flex min-h-screen items-center justify-center bg-gradient-to-br from-slate-900 via-blue-900 to-slate-800">
			<div class="text-center">
				<div class="pokeball-loader mx-auto"></div>
				<p class="mt-6 pixel-text text-2xl text-yellow-400">Loading Pokémon data...</p>
			</div>
		</div>
	{/snippet}

	<div class="min-h-screen bg-gradient-to-br from-slate-900 via-blue-900 to-slate-800 p-4 sm:p-8">
		<div class="mx-auto max-w-4xl">
			<!-- Main Pokédex Container -->
			<div class="pokedex-container mb-8">
				<!-- Pokédex Header -->
				<div class="mb-6 text-center">
					<h1 class="retro-title mb-4">POKÉDEX v2.0</h1>
					<p class="retro-subtitle text-lg text-gray-300">Digital Pokémon Encyclopedia</p>
				</div>

				<!-- Pokémon Selector Screen -->
				<div class="pokedex-screen mb-8">
					<h2 class="retro-subtitle mb-6 text-center text-xl">SELECT POKÉMON</h2>
					<div class="crt-monitor">
						<div class="mb-6">
							<label class="mb-4 block">
								<span class="pixel-text text-lg text-yellow-400">POKÉMON ID:</span>
								<input
									type="number"
									bind:value={pokemonId}
									min="1"
									max="1025"
									class="retro-input mt-2 block w-full max-w-xs text-center text-xl"
								/>
							</label>
							<div class="mt-4 rounded-lg bg-gray-800/50 p-4">
								<p class="pixel-text text-green-400">QUICK ACCESS:</p>
								<div class="mt-2 flex flex-wrap gap-2">
									<button onclick={() => (pokemonId = 25)} class="retro-button text-xs">PIKACHU</button>
									<button onclick={() => (pokemonId = 1)} class="retro-button text-xs">BULBASAUR</button>
									<button onclick={() => (pokemonId = 150)} class="retro-button text-xs">MEWTWO</button>
									<button onclick={() => (pokemonId = 6)} class="retro-button text-xs">CHARIZARD</button>
								</div>
							</div>
						</div>
					</div>
				</div>
			</div>

			<!-- Main Pokémon Data Display -->
			<div class="pokedex-container mb-8">
				{#await fetchPokemon(pokemonId)}
					<div class="pokedex-screen">
						<div class="crt-monitor text-center">
							<div class="pokeball-loader mx-auto mb-4"></div>
							<p class="pixel-text text-2xl text-green-400">LOADING POKÉMON DATA...</p>
						</div>
					</div>
				{:then pokemon}
					<div class="pokemon-card p-6">
						<!-- Pokémon Name Header -->
						<div class="mb-6 text-center">
							<h2 class="retro-title text-2xl uppercase text-red-600">{pokemon.name}</h2>
							<p class="pixel-text text-lg text-gray-600">No. {pokemon.id.toString().padStart(3, '0')}</p>
						</div>

						<div class="grid grid-cols-1 gap-8 lg:grid-cols-2">
							<!-- Left Side - Image -->
							<div class="relative">
								<div class="pokedex-screen p-4">
									<div class="crt-monitor text-center">
										<img
											src={pokemon.sprites.other['official-artwork'].front_default}
											alt={pokemon.name}
											class="mx-auto h-64 w-64 object-contain drop-shadow-2xl"
										/>
									</div>
								</div>
							</div>

							<!-- Right Side - Info -->
							<div class="space-y-6">
								<!-- Basic Stats -->
								<div class="rounded-lg bg-gray-800/20 p-4">
									<h3 class="retro-subtitle mb-4 text-lg">BASIC DATA</h3>
									<div class="space-y-2 pixel-text text-lg">
										<div class="flex justify-between">
											<span class="text-yellow-600">HEIGHT:</span>
											<span class="text-white">{pokemon.height / 10}m</span>
										</div>
										<div class="flex justify-between">
											<span class="text-yellow-600">WEIGHT:</span>
											<span class="text-white">{pokemon.weight / 10}kg</span>
										</div>
										<div class="flex justify-between">
											<span class="text-yellow-600">EXP:</span>
											<span class="text-white">{pokemon.base_experience}</span>
										</div>
									</div>
								</div>

								<!-- Types -->
								<div class="rounded-lg bg-gray-800/20 p-4">
									<h3 class="retro-subtitle mb-4 text-lg">TYPE</h3>
									<div class="flex gap-3">
										{#each pokemon.types as type (type.slot)}
											<span
												class="type-badge text-white"
												style="background-color: {getTypeColor(type.type.name)}"
											>
												{type.type.name}
											</span>
										{/each}
									</div>
								</div>

								<!-- Abilities -->
								<div class="rounded-lg bg-gray-800/20 p-4">
									<h3 class="retro-subtitle mb-4 text-lg">ABILITIES</h3>
									<ul class="space-y-2">
										{#each pokemon.abilities as ability (ability.slot)}
											<li class="pixel-text text-lg text-white">
												<span class="text-green-400">▶</span>
												{ability.ability.name.replace('-', ' ').toUpperCase()}
												{#if ability.is_hidden}
													<span class="text-yellow-400">(HIDDEN)</span>
												{/if}
											</li>
										{/each}
									</ul>
								</div>
							</div>
						</div>
					</div>
				{:catch error}
					<div class="pokedex-screen">
						<div class="crt-monitor text-center">
							<div class="glitch">
								<p class="pixel-text text-2xl text-red-400">ERROR: {error.message}</p>
								<p class="pixel-text mt-2 text-yellow-400">SYSTEM MALFUNCTION</p>
							</div>
						</div>
					</div>
				{/await}
			</div>

			<!-- Stats Analysis Section -->
			<div class="pokedex-container mb-8">
				<button
					class="retro-button blue mb-6 block w-full"
					onclick={() => (showStats = !showStats)}
				>
					{showStats ? '◄ HIDE' : '▶ SHOW'} STATS ANALYSIS
				</button>

				{#if showStats}
					<div class="pokedex-screen">
						<h2 class="retro-subtitle mb-6 text-center text-xl">BATTLE STATISTICS</h2>
						<div class="crt-monitor">
							{#await Promise.all([fetchPokemon(pokemonId), calculateStats(pokemonId)])}
								<div class="text-center">
									<div class="pokeball-loader mx-auto mb-4"></div>
									<p class="pixel-text text-2xl text-green-400">CALCULATING STATS...</p>
								</div>
							{:then [pokemon, statsInfo]}
								<div class="space-y-4">
									{#each pokemon.stats as stat (stat.stat.name)}
										<div class="space-y-2">
											<div class="flex justify-between pixel-text text-lg">
												<span class="uppercase text-yellow-400">{stat.stat.name.replace('-', ' ')}</span>
												<span class="text-white">{stat.base_stat}</span>
											</div>
											<div class="retro-stat-bar">
												<div
													class="retro-stat-fill"
													style="width: {Math.min((stat.base_stat / 255) * 100, 100)}%"
												></div>
											</div>
										</div>
									{/each}
									
									<div class="mt-8 grid grid-cols-2 gap-4 rounded-lg bg-gray-800/30 p-4">
										<div class="text-center">
											<p class="pixel-text text-2xl text-green-400">{statsInfo.total}</p>
											<p class="retro-subtitle text-sm">TOTAL</p>
										</div>
										<div class="text-center">
											<p class="pixel-text text-2xl text-blue-400">{statsInfo.average}</p>
											<p class="retro-subtitle text-sm">AVERAGE</p>
										</div>
									</div>
								</div>
							{:catch error}
								<div class="glitch text-center">
									<p class="pixel-text text-2xl text-red-400">STAT ERROR: {error.message}</p>
								</div>
							{/await}
						</div>
					</div>
				{/if}
			</div>

			<!-- Evolution Chain Section -->
			<div class="pokedex-container mb-8">
				<button
					class="retro-button purple mb-6 block w-full"
					onclick={() => (showEvolution = !showEvolution)}
				>
					{showEvolution ? '◄ HIDE' : '▶ SHOW'} EVOLUTION CHAIN
				</button>

				{#if showEvolution}
					<div class="pokedex-screen">
						<h2 class="retro-subtitle mb-6 text-center text-xl">EVOLUTION DATA</h2>
						<div class="crt-monitor">
							{#await fetchPokemonSpecies(pokemonId)}
								<div class="text-center">
									<div class="pokeball-loader mx-auto mb-4"></div>
									<p class="pixel-text text-2xl text-green-400">LOADING EVOLUTION DATA...</p>
								</div>
							{:then species}
								{#await fetchEvolutionChain(species.evolution_chain.url)}
									<div class="text-center">
										<p class="pixel-text text-2xl text-blue-400">FETCHING EVOLUTION CHAIN...</p>
									</div>
								{:then evolutionData}
									{@const chain = parseEvolutionChain(evolutionData.chain)}
									<div class="flex flex-wrap items-center justify-center gap-6">
										{#each chain as pokemon, i (pokemon)}
											<div class="text-center">
												<div class="pokemon-card mb-4 p-4">
													<p class="pixel-text text-xl uppercase text-gray-800">{pokemon}</p>
												</div>
											</div>
											{#if i < chain.length - 1}
												<div class="evolution-arrow">▶</div>
											{/if}
										{/each}
									</div>
									
									{#if chain.length === 1}
										<div class="mt-6 text-center">
											<p class="pixel-text text-lg text-yellow-400">NO EVOLUTION DATA AVAILABLE</p>
										</div>
									{/if}
								{:catch error}
									<div class="glitch text-center">
										<p class="pixel-text text-2xl text-red-400">EVOLUTION ERROR: {error.message}</p>
									</div>
								{/await}
							{:catch error}
								<div class="glitch text-center">
									<p class="pixel-text text-2xl text-red-400">SPECIES ERROR: {error.message}</p>
								</div>
							{/await}
						</div>
					</div>
				{/if}
			</div>
		</div>
	</div>
</svelte:boundary>
