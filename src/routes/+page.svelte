<script>
	// Simulated API call that returns user data
	async function fetchUser(id) {
		await new Promise((resolve) => setTimeout(resolve, 1000));
		return {
			id,
			name: `User ${id}`,
			email: `user${id}@example.com`
		};
	}

	// Simulated API call that fetches posts
	async function fetchPosts(userId) {
		await new Promise((resolve) => setTimeout(resolve, 1500));
		return [
			{ id: 1, title: 'First Post', content: 'This is the first post content' },
			{ id: 2, title: 'Second Post', content: 'This is the second post content' },
			{ id: 3, title: 'Third Post', content: 'This is the third post content' }
		];
	}

	// Simple async calculation
	async function add(a, b) {
		await new Promise((resolve) => setTimeout(resolve, 500));
		return a + b;
	}

	let userId = $state(1);
	let showPosts = $state(false);
</script>

<svelte:boundary>
	{#snippet pending()}
		<div class="flex min-h-screen items-center justify-center">
			<div class="text-center">
				<div class="mx-auto h-12 w-12 animate-spin rounded-full border-b-2 border-gray-900"></div>
				<p class="mt-4 text-gray-600">Loading...</p>
			</div>
		</div>
	{/snippet}

	<div class="container mx-auto p-8">
		<h1 class="mb-6 text-3xl font-bold">Svelte Await Expressions Example</h1>

		<!-- Simple example from the docs -->
		<div class="mb-6 rounded-lg bg-white p-6 shadow-md">
			<h2 class="mb-4 text-xl font-semibold">Simple Addition</h2>
			<p>2 + 3 = {await add(2, 3)}</p>
		</div>

		<!-- User data example -->
		<div class="mb-6 rounded-lg bg-white p-6 shadow-md">
			<h2 class="mb-4 text-xl font-semibold">User Information</h2>
			<div class="mb-4">
				<label class="mb-2 block">
					<span class="text-gray-700">User ID:</span>
					<input
						type="number"
						bind:value={userId}
						min="1"
						max="5"
						class="mt-1 block w-32 rounded-md border-gray-300 shadow-sm"
					/>
				</label>
			</div>

			{#await fetchUser(userId)}
				<p class="text-gray-500">Loading user...</p>
			{:then user}
				<p><strong>Name:</strong> {user.name}</p>
				<p><strong>Email:</strong> {user.email}</p>
			{:catch error}
				<p class="text-red-500">Error: {error.message}</p>
			{/await}
		</div>

		<!-- Posts example -->
		<button
			class="mb-6 rounded bg-blue-500 px-4 py-2 font-bold text-white hover:bg-blue-700"
			onclick={() => (showPosts = !showPosts)}
		>
			{showPosts ? 'Hide' : 'Show'} Posts
		</button>

		{#if showPosts}
			<div class="rounded-lg bg-white p-6 shadow-md">
				<h2 class="mb-4 text-xl font-semibold">User Posts</h2>
				{#await fetchPosts(userId)}
					<p class="text-gray-500">Loading posts...</p>
				{:then posts}
					<div class="space-y-4">
						{#each posts as post}
							<div class="border-b pb-4 last:border-b-0">
								<h3 class="font-semibold">{post.title}</h3>
								<p class="text-gray-600">{post.content}</p>
							</div>
						{/each}
					</div>
				{:catch error}
					<p class="text-red-500">Error loading posts: {error.message}</p>
				{/await}
			</div>
		{/if}
	</div>
</svelte:boundary>
