<script>
	import entries from '$lib/data/entries.json';

	// Fisher-Yates shuffle
	function shuffle(array) {
		const arr = [...array];
		for (let i = arr.length - 1; i > 0; i--) {
			const j = Math.floor(Math.random() * (i + 1));
			[arr[i], arr[j]] = [arr[j], arr[i]];
		}
		return arr;
	}

	let deck = $state(shuffle(entries));
	let currentIndex = $state(0);

	let currentEntry = $derived(deck[currentIndex]);
	let nextEntry = $derived(deck[(currentIndex + 1) % deck.length]);

	function handleClick() {
		if (currentIndex >= deck.length - 1) {
			// Reshuffle when we've shown all entries
			deck = shuffle(entries);
			currentIndex = 0;
		} else {
			currentIndex++;
		}
	}
</script>

<svelte:head>
	<title>Portfolio</title>
	<link rel="preconnect" href="https://fonts.googleapis.com">
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous">
	<link href="https://fonts.googleapis.com/css2?family=DM+Sans:ital,wght@0,400;0,500;0,700;1,400&display=swap" rel="stylesheet">
</svelte:head>

<main>
	<section class="intro">
		<p>
			I am an independent qualitative researcher, strategist and designer.
			I help organizations understand people—their needs, behaviors, and contexts—and
			translate those insights into meaningful products, services, and experiences.
		</p>
	</section>

	<section class="rotating">
		<p class="entry">
			In {currentEntry.year} I {currentEntry.text}.
		</p>
		<button onclick={handleClick}>
			{nextEntry.button}
		</button>
	</section>
</main>

<style>
	main {
		max-width: 650px;
		margin: 0 auto;
		padding: 4rem 2rem;
	}

	.intro {
		margin-bottom: 3rem;
	}

	.intro p {
		font-size: 1.25rem;
		line-height: 1.6;
		color: var(--text-primary);
	}

	.rotating {
		padding-top: 2rem;
		border-top: 1px solid var(--border);
	}

	.entry {
		font-size: 1.25rem;
		line-height: 1.6;
		color: var(--text-primary);
		margin-bottom: 2rem;
	}

	button {
		font-family: inherit;
		font-size: 1rem;
		font-weight: 500;
		padding: 0.75rem 1.5rem;
		background: var(--bg-secondary);
		border: 1px solid var(--border);
		border-radius: 4px;
		cursor: pointer;
		transition: all 0.15s ease;
		color: var(--text-secondary);
	}

	button:hover {
		background: var(--bg-hover);
		border-color: var(--border-hover);
		color: var(--text-primary);
	}

	button:active {
		transform: scale(0.98);
	}
</style>
