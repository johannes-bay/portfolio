<script>
	import { marked } from 'marked';
	import contentRaw from '$lib/content.md?raw';

	// Make all links open in new tab
	const renderer = new marked.Renderer();
	renderer.link = ({ href, text }) =>
		`<a href="${href}" target="_blank" rel="noopener">${text}</a>`;

	// Parse markdown content
	const [introMd, entriesMd] = contentRaw.split('---');

	const introHtml = marked(introMd.trim(), { renderer });

	const entries = entriesMd
		.trim()
		.split('\n')
		.filter((line) => line.startsWith('- '))
		.map((line) => {
			const content = line.slice(2);
			const [button, year, ...rest] = content.split(' | ');
			const text = marked.parseInline(rest.join(' | ').trim(), { renderer });
			return { button: button.trim(), year: parseInt(year.trim()), text };
		});

	// Fisher-Yates shuffle
	function shuffle(array) {
		const arr = [...array];
		for (let i = arr.length - 1; i > 0; i--) {
			const j = Math.floor(Math.random() * (i + 1));
			[arr[i], arr[j]] = [arr[j], arr[i]];
		}
		return arr;
	}

	function stripHtml(html) {
		const div = document.createElement('div');
		div.innerHTML = html;
		return div.textContent;
	}

	let deck = $state(shuffle(entries));
	let currentIndex = $state(0);
	let animating = $state(false);
	let displayText = $state('');
	let showHtml = $state(true);

	let currentEntry = $derived(deck[currentIndex]);
	let nextEntry = $derived(deck[(currentIndex + 1) % deck.length]);

	const CHARS_PER_FRAME = 3;

	// Initialize display text
	$effect(() => {
		if (!animating) {
			displayText = stripHtml(currentEntry.text);
		}
	});

	function step(fn) {
		requestAnimationFrame(() => fn());
	}

	function handleClick() {
		if (animating) return;
		animating = true;
		showHtml = false;

		const currentPlain = stripHtml(currentEntry.text);
		displayText = currentPlain;
		let i = currentPlain.length;

		// Delete phase
		function deleteStep() {
			i = Math.max(0, i - CHARS_PER_FRAME);
			displayText = currentPlain.slice(0, i);
			if (i > 0) {
				step(deleteStep);
			} else {
				// Advance to next entry
				if (currentIndex >= deck.length - 1) {
					deck = shuffle(entries);
					currentIndex = 0;
				} else {
					currentIndex++;
				}

				// Type phase
				const newPlain = stripHtml(deck[currentIndex].text);
				let j = 0;
				function typeStep() {
					j = Math.min(newPlain.length, j + CHARS_PER_FRAME);
					displayText = newPlain.slice(0, j);
					if (j < newPlain.length) {
						step(typeStep);
					} else {
						showHtml = true;
						animating = false;
					}
				}
				step(typeStep);
			}
		}
		step(deleteStep);
	}
</script>

<svelte:head>
	<title>Johannes Bay — researcher, strategist, designer</title>
	<meta name="description" content="Johannes Bay — independent qualitative researcher, strategist and designer" />
	<meta name="author" content="Johannes Bay" />
	<link rel="canonical" href="https://johannes.co.nz" />

	<!-- Open Graph -->
	<meta property="og:type" content="website" />
	<meta property="og:title" content="Johannes Bay" />
	<meta property="og:description" content="Independent qualitative researcher, strategist and designer" />
	<meta property="og:url" content="https://johannes.co.nz" />

	<!-- Twitter -->
	<meta name="twitter:card" content="summary" />
	<meta name="twitter:title" content="Johannes Bay" />
	<meta name="twitter:description" content="Independent qualitative researcher, strategist and designer" />
</svelte:head>

<main>
	<section class="intro">
		{@html introHtml}
	</section>

	<section class="rotating">
		<p class="entry">
			In {currentEntry.year} I {#if showHtml}{@html currentEntry.text}{:else}{displayText}{/if}.
		</p>
		<button onclick={handleClick} disabled={animating}>
			{nextEntry.button}
		</button>
	</section>
</main>

<style>
	main {
		max-width: 480px;
		padding: 2rem;
	}

	.intro {
		margin-bottom: 3rem;
	}

	.intro :global(p) {
		font-size: 1rem;
		line-height: 1.375;
		color: var(--text-primary);
		margin-bottom: 1rem;
	}

	.intro :global(p:last-child) {
		margin-bottom: 0;
	}

	.rotating {
		padding-top: 2rem;
		border-top: 1px solid var(--border);
	}

	.entry {
		font-size: 1rem;
		line-height: 1.375;
		color: var(--text-primary);
		margin-bottom: 2rem;
	}

	button {
		font-family: inherit;
		font-size: 1rem;
		font-weight: 500;
		padding: 8px;
		background: #fff;
		border: none;
		border-radius: 8px;
		line-height: 1;
		text-box-trim: both;
		text-box-edge: cap alphabetic;
		cursor: pointer;
		transition: all 0.15s ease;
		color: #0000ff;
	}

	button:hover {
		background: #f0f0f0;
		color: #0000ff;
	}

	button:active {
		transform: scale(0.98);
	}
</style>
