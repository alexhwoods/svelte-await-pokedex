# 🎮 Pokédex - Svelte 5 Await Expressions Demo

A retro-styled Pokémon encyclopedia that demonstrates Svelte 5's new experimental **await expressions** functionality. Built with SvelteKit, TypeScript, and Tailwind CSS, featuring a nostalgic 90's Game Boy aesthetic.

🚀 **[Live Demo: https://svelte-await-pokedex.vercel.app/](https://svelte-await-pokedex.vercel.app/)**

![Svelte 5](https://img.shields.io/badge/Svelte-5.38-ff3e00?style=for-the-badge&logo=svelte)
![TypeScript](https://img.shields.io/badge/TypeScript-007ACC?style=for-the-badge&logo=typescript)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=for-the-badge&logo=tailwind-css)
![Vercel](https://img.shields.io/badge/Vercel-000000?style=for-the-badge&logo=vercel&logoColor=white)

## 🚀 Quick Start

### Prerequisites

- Node.js 18+
- pnpm (recommended) or npm

### Installation & Running

```bash
# Clone and navigate to the project
cd sveltekit-test-await

# Install dependencies
pnpm install

# Start the development server
pnpm dev

# Visit http://localhost:5174
```

### Available Scripts

```bash
pnpm dev          # Start development server
pnpm build        # Build for production
pnpm preview      # Preview production build
pnpm test         # Run unit tests
pnpm test:e2e     # Run end-to-end tests
pnpm lint         # Run linting
pnpm format       # Format code with Prettier
```

## 🔬 Exploring Svelte 5's Await Expressions

This app serves as a comprehensive exploration of Svelte 5's new **await expressions** feature, demonstrating various patterns and use cases through a real-world Pokémon API integration.

### 🛠️ Configuration

The experimental await functionality is enabled in `svelte.config.js`:

```javascript
export default {
	compilerOptions: {
		experimental: {
			async: true // Enable await expressions
		}
	}
};
```

### 🔑 Core Concept: `<svelte:boundary>`

**All await expressions must be wrapped in a `<svelte:boundary>` with a `pending` snippet:**

```svelte
<svelte:boundary>
	{#snippet pending()}
		<div class="loading-state">
			<!-- Unified loading UI for all await expressions -->
			<div class="pokeball-loader"></div>
			<p>Loading Pokémon data...</p>
		</div>
	{/snippet}

	<!-- All your await expressions go here -->
</svelte:boundary>
```

## 📋 Await Expression Patterns Demonstrated

### 1. **Traditional `{#await}` Blocks**

_Still supported and useful for complex loading states_

```svelte
{#await fetchPokemon(pokemonId)}
	<div class="custom-loader">Loading Pokémon...</div>
{:then pokemon}
	<div class="pokemon-card">
		<h2>{pokemon.name}</h2>
		<img src={pokemon.sprites.front_default} alt={pokemon.name} />
	</div>
{:catch error}
	<div class="error-state">Error: {error.message}</div>
{/await}
```

**Location in app:** Main Pokémon data display (`src/routes/+page.svelte:138`)

### 2. **Parallel Async Operations with Promise.all()**

_Multiple API calls executed simultaneously_

```svelte
{#await Promise.all([fetchPokemon(pokemonId), calculateStats(pokemonId)])}
	<p>Calculating battle statistics...</p>
{:then [pokemon, statsInfo]}
	<div class="stats-display">
		<h3>{pokemon.name} Battle Stats</h3>
		<p>Total Power: {statsInfo.total}</p>
		<p>Average: {statsInfo.average}</p>
	</div>
{/await}
```

**Location in app:** Stats analysis section (`src/routes/+page.svelte:246`)

### 3. **Nested Await Expressions**

_Dependent API calls where one result feeds into another_

```svelte
{#await fetchPokemonSpecies(pokemonId)}
	<p>Loading species data...</p>
{:then species}
	{#await fetchEvolutionChain(species.evolution_chain.url)}
		<p>Processing evolution chain...</p>
	{:then evolutionData}
		{@const chain = parseEvolutionChain(evolutionData.chain)}
		<div class="evolution-display">
			{#each chain as pokemon, i (pokemon)}
				<span>{pokemon}</span>
				{#if i < chain.length - 1}→{/if}
			{/each}
		</div>
	{/await}
{/await}
```

**Location in app:** Evolution chain section (`src/routes/+page.svelte:302`)

### 4. **Reactive Await Expressions**

_Automatically re-execute when dependencies change_

When `pokemonId` changes:

- All await expressions automatically re-run
- Loading states are shown via the `pending` snippet
- UI updates when new data arrives
- Error boundaries catch and display failures

## 🎯 Real-World API Integration

### PokéAPI Integration

The app demonstrates await expressions with real HTTP requests to the [PokéAPI](https://pokeapi.co/):

```typescript
// Fetch main Pokémon data
async function fetchPokemon(id: number) {
	const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${id}`);
	if (!response.ok) throw new Error('Pokémon not found');
	return response.json();
}

// Fetch species information for evolution data
async function fetchPokemonSpecies(id: number) {
	const response = await fetch(`https://pokeapi.co/api/v2/pokemon-species/${id}`);
	if (!response.ok) throw new Error('Species data not found');
	return response.json();
}

// Calculate derived stats asynchronously
async function calculateStats(pokemonId: number) {
	const pokemon = await fetchPokemon(pokemonId);
	const total = pokemon.stats.reduce((sum, stat) => sum + stat.base_stat, 0);
	return { total, average: Math.round(total / pokemon.stats.length) };
}
```

### Error Handling Patterns

Each await expression includes proper error handling:

```svelte
{:catch error}
  <div class="error-display">
    <p>⚠️ ERROR: {error.message}</p>
    <p>System malfunction detected</p>
  </div>
{/await}
```

## 🎨 UI/UX Features

### Retro 90's Pokémon Theme

- **Game Boy Color Palette**: Authentic retro colors and styling
- **Pixel-Perfect Typography**: Monospace fonts and ASCII decorations
- **Custom Animations**: Pokéball loaders, stat bar fills, holographic effects
- **Responsive Design**: Works seamlessly across desktop and mobile

### Interactive Elements

- **Quick Select Buttons**: Popular Pokémon for easy testing
- **Manual ID Input**: Support for all 1,025 Pokémon
- **Expandable Sections**: Toggle stats and evolution data
- **Loading States**: Themed loading animations for each section

## 🔧 Technical Architecture

### Project Structure

```
src/
├── routes/
│   └── +page.svelte          # Main Pokédex component
├── app.html                  # HTML template
├── app.css                   # Global styles (Tailwind imports)
└── lib/                      # Shared utilities (if any)
```

### Key Technologies

- **Svelte 5.38**: Latest version with await expressions
- **SvelteKit**: Full-stack framework
- **TypeScript**: Type safety for API responses
- **Tailwind CSS**: Utility-first styling with custom classes
- **Vite**: Build tool and dev server

### Type Definitions

```typescript
interface EvolutionChain {
	species: { name: string };
	evolves_to: EvolutionChain[];
}

interface PokemonStat {
	base_stat: number;
	stat: { name: string };
}
```

## 🎮 How to Test Await Expressions

### Try These Pokémon IDs:

- **25**: Pikachu (iconic starter)
- **150**: Mewtwo (legendary with high stats)
- **1**: Bulbasaur (has evolution chain)
- **999**: Invalid ID (test error handling)

### Testing Scenarios:

1. **Change Pokémon ID** - Watch all sections reactively update
2. **Toggle Stats** - See parallel API calls with `Promise.all()`
3. **View Evolution** - Experience nested await expressions
4. **Network Issues** - Test error boundaries and recovery

## 🔍 Key Learning Points

### What Makes This Different from Svelte 4?

1. **Unified Loading States**: One `pending` snippet handles all awaits
2. **Automatic Reactivity**: Awaits re-run when dependencies change
3. **Simplified Syntax**: Less boilerplate than traditional patterns
4. **Better Error Boundaries**: Centralized error handling

### Performance Benefits

- **Parallel Execution**: Multiple awaits can run simultaneously
- **Efficient Re-renders**: Only affected components update
- **Smart Batching**: Related updates are batched together

## ⚠️ Important Notes

### Experimental Status

- Await expressions are **experimental** in Svelte 5
- May have breaking changes in future versions
- Will be stabilized in Svelte 6

### Browser Support

- Modern browsers with ES2022 support
- Uses native `fetch()` API
- Requires JavaScript enabled

### Performance Considerations

- API responses are not cached (intentionally for demo purposes)
- Each ID change triggers new API calls
- Consider implementing caching for production use

## 🤝 Contributing

This is a learning/demo project, but contributions are welcome:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Run tests and linting: `pnpm lint && pnpm test`
5. Submit a pull request

### Development Guidelines

- Follow TypeScript best practices
- Maintain the retro aesthetic
- Add comments explaining await patterns
- Test with various Pokémon IDs

## 📝 License

MIT License - feel free to use this code for learning and experimentation!

---

## 🔗 Useful Links

- [Svelte 5 Await Expressions Docs](https://svelte.dev/docs/svelte/await-expressions)
- [PokéAPI Documentation](https://pokeapi.co/docs/v2)
- [SvelteKit Documentation](https://svelte.dev/docs/kit)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)

**Happy coding, and gotta catch 'em all! 🎮✨**
