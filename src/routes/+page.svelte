<script>
	import { fade, fly } from 'svelte/transition';
	import { base } from '$app/paths';
	let currentLine = 0;
	let answer = '';
	let answered = false;
	let answeredNo = false;
	let yesScale = 1;
	let currentNoAnswer = '';
	let noClickCount = 0;

	const lines = [
		'Hi Liane ❤️',
		'I built this little letter just for you...',
		"Because you're truly special.",
		'Since we started talking, everything feels a little lighter.',
		'You make me smile. A LOT.',
		'And I keep thinking about you.',
		'So I just have one very important question...',
		'Will you be my girlfriend? 💌'
	];

	const noAnswers = ['Are you sure? 🥺'];

	function nextLine() {
		if (currentLine < lines.length - 1) {
			currentLine++;
		}
	}

	let showHeartExplosion = false;
	/** @type {{id: string, x: number, y: number, delay: number}[]} */
	let heartExplosionItems = [];

	function createHeartExplosion() {
		heartExplosionItems = Array.from({ length: 30 }, (_, i) => ({
			id: crypto.randomUUID(),
			x: Math.random() * 200 - 100, // seitlich verteilt
			y: Math.random() * -150 - 50, // nach oben
			delay: i * 0.02
		}));
		showHeartExplosion = true;

		// Nach 2 Sekunden wieder ausblenden
		setTimeout(() => {
			showHeartExplosion = false;
		}, 2000);
	}

	function sayYes() {
		answer =
			"YAAAY! I'm so happy right now! 🥰\n P.S. There’s still so much I want to tell you… 💖";
		answered = true;
		createHeartExplosion();
	}

	function sayNo() {
		yesScale += 0.2;
		if (noClickCount < noAnswers.length) {
			currentNoAnswer = noAnswers[noClickCount];
			noClickCount++;
		} else {
			answer =
				'Oh no... I understand. I just wanted to let you know how I feel. I still think you are amazing! 💔';
			answered = true;
			answeredNo = true;
		}
	}

	const heartCount = 100;
	const hearts = Array.from({ length: heartCount }, () => ({
		id: crypto.randomUUID(),
		left: Math.random() * 100,
		size: Math.random() * 16 + 8,
		delay: Math.random() * 10,
		duration: Math.random() * 10 + 10,
		opacity: Math.random() * 0.5 + 0.3
	}));
</script>

<svelte:head>
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
	<link
		rel="preload"
		href="https://fonts.googleapis.com/css2?family=Dancing+Script:wght@400..700&display=swap"
		as="style"
	/>
	<link
		rel="preload"
		href="https://fonts.gstatic.com/s/dancingscript/v25/If2cXTr6YS-zF4S-kcSWSVi_sxjsohD9F50Ruu7BMSo3ROp-.woff2"
		as="font"
		type="font/woff2"
		crossorigin
	/>
	<link rel="preload" href="{base}/imgs/cat.gif" as="image" />
	<link rel="preload" href="{base}/imgs/hai.gif" as="image" />
</svelte:head>

<div
	class="letter-container bg-gradient-to-br from-pink-200 via-rose-100 to-amber-100"
	on:click={nextLine}
	on:keydown={(e) => (e.key === 'Enter' || e.key === ' ') && nextLine()}
	role="button"
	tabindex="0"
	aria-label="Next line"
>
	<div class="heart-bg">
		{#each hearts as heart (heart.id)}
			<span
				class="heart"
				style="
					left: {heart.left}%;
					width: {heart.size}px;
					height: {heart.size}px;
					animation-delay: {heart.delay}s;
					animation-duration: {heart.duration}s;
					opacity: {heart.opacity};
				"
			></span>
		{/each}
	</div>
	{#if showHeartExplosion}
		<div class="heart-explosion">
			{#each heartExplosionItems as h (h.id)}
				<span
					class="explosion-heart"
					style="--x: {h.x}px; --y: {h.y}px; animation-delay: {h.delay}s">💖</span
				>
			{/each}
		</div>
	{/if}

	<div class="content">
		{#key currentLine}
			{#if currentLine < lines.length - 1}
				<p in:fly={{ y: 20, duration: 500 }} out:fade={{ duration: 300 }}>{lines[currentLine]}</p>
			{:else}
				<div class="question" in:fly={{ y: 20, duration: 500, delay: 300 }}>
					{#if !answered}
						<p>{lines[currentLine]}</p>
					{/if}
					{#key answered}
						{#if !answered}
							<div class="buttons mb-4" in:fade out:fade={{ duration: 300 }}>
								<button
									class="yes"
									on:click|stopPropagation={sayYes}
									style="transform: scale({yesScale})"
								>
									Yes 💖
								</button>
								<button class="no" on:click|stopPropagation={sayNo}>No 😢</button>
							</div>

							{#if currentNoAnswer}
								<p class="no-answer" in:fade>{currentNoAnswer}</p>
							{/if}

							<img
								src="{base}/imgs/cat.gif"
								alt="cat asking question"
								class="responsive"
								in:fade
								out:fade={{ duration: 300 }}
							/>
						{:else}
							<p class="answer" in:fly={{ y: 20, duration: 500, delay: 300 }}>{answer}</p>
							{#if !answeredNo}
								<div class="align-items-center flex justify-center">
									<img
										src="{base}/imgs/hai.gif"
										alt="cat jumping happily"
										class="responsive"
										in:fade
										out:fade={{ duration: 300 }}
									/>
								</div>
							{/if}
						{/if}
					{/key}
				</div>
			{/if}
		{/key}
	</div>
</div>

<style lang="scss">
	@use 'sass:color';
	@import url('https://fonts.googleapis.com/css2?family=Dancing+Script:wght@400..700&display=swap');

	$heart-color: #ff5ca2;
	$text-color: #c3195d;
	$card-color: rgba(255, 255, 255, 0.85);

	$font-family: 'Dancing Script', cursive;

	.letter-container {
		position: relative;
		display: flex;
		align-items: center;
		justify-content: center;
		flex-direction: column;
		min-height: 100vh;
		width: 100%;
		text-align: center;
		padding: 2rem 1rem;
		overflow: auto;
		cursor: pointer;
		outline: none;
		font-family: $font-family;
		color: $text-color;
	}

	.big-heart {
		font-size: 4rem;
		animation: pulse 1s infinite alternate;
		margin-top: 1rem;
	}

	@keyframes pulse {
		from {
			transform: scale(1);
		}
		to {
			transform: scale(1.2);
		}
	}

	.heart-bg {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		pointer-events: none;
		overflow: hidden;
		z-index: 0;
		background: linear-gradient(135deg, rgba(255, 255, 255, 0.1), rgba(255, 255, 255, 0));
		backdrop-filter: blur(10px);
		-webkit-backdrop-filter: blur(10px);
		border-radius: 20px;

		.heart {
			position: absolute;
			bottom: -20px;
			background-color: $heart-color;
			mask-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 24 24' fill='white' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.41 4.42 3 7.5 3c1.74 0 3.41 0.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.41 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z'/%3E%3C/svg%3E");
			mask-size: contain;
			mask-repeat: no-repeat;
			mask-position: center;
			-webkit-mask-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 24 24' fill='white' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.41 4.42 3 7.5 3c1.74 0 3.41 0.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.41 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z'/%3E%3C/svg%3E");
			-webkit-mask-size: contain;
			-webkit-mask-repeat: no-repeat;
			-webkit-mask-position: center;
			animation-name: floatHeart;
			animation-timing-function: linear;
			animation-iteration-count: infinite;
		}
	}

	@keyframes floatHeart {
		from {
			transform: translateY(0) scale(1);
		}
		to {
			transform: translateY(-120vh) scale(1.2);
		}
	}

	.content {
		position: relative;
		z-index: 1;
		max-width: 600px;
		width: 100%;
		background: $card-color;
		padding: 2rem;
		border-radius: 1rem;
		box-shadow: 0 0 30px rgba(0, 0, 0, 0.1);
		margin: auto;

		@media (max-width: 768px) {
			padding: 1.5rem;
			margin: 1rem auto;
		}

		p {
			font-size: 2.2rem;
			line-height: 1.8;
			margin: 1rem 0;
			color: $text-color;
		}
	}

	.question {
		margin-top: 2rem;

		.buttons {
			display: flex;
			justify-content: center;
			gap: 2rem;
			margin-top: 1rem;

			button {
				font-size: 1.3rem;
				padding: 0.75rem 1.8rem;
				border: none;
				border-radius: 8px;
				cursor: pointer;
				transition:
					transform 0.3s ease,
					background-color 0.3s ease;
				box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
			}

			.yes {
				background-color: $heart-color;
				color: white;

				&:hover {
					background-color: color.scale($heart-color, $lightness: 10%);
				}
			}

			.no {
				background-color: #ddd;
				color: #444;

				&:hover {
					background-color: #bbb;
				}
			}
		}
	}

	.answer {
		margin-top: 2rem;
		font-size: clamp(1.5rem, 2vw, 2.2rem);
		color: color.scale($heart-color, $lightness: -14.7%);
		font-weight: bold;
		white-space: pre-line;
	}

	.extra-btn,
	.flower-btn {
		margin-top: 1rem;
		padding: 0.8rem 1.6rem;
		font-size: 1.2rem;
		border: none;
		border-radius: 8px;
		cursor: pointer;
		background-color: #ffb6c1;
		color: white;
		box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
		transition: background-color 0.3s;
	}

	.extra-btn:hover,
	.flower-btn:hover {
		background-color: #f48fb1;
	}

	.flower-container {
		position: relative;
		width: 100%;
		height: 300px;
		display: flex;
		flex-wrap: wrap;
		justify-content: center;
		align-items: center;
		gap: 10px;
		margin-top: 2rem;
		animation: fadeIn 1s ease-in;
	}

	.flower {
		font-size: 2rem;
		animation: float 5s ease-in-out infinite;
		animation-delay: calc(var(--i) * 0.1s);
	}

	@keyframes float {
		0% {
			transform: translateY(0) scale(1);
			opacity: 0;
		}
		50% {
			transform: translateY(-20px) scale(1.2);
			opacity: 1;
		}
		100% {
			transform: translateY(0) scale(1);
			opacity: 0;
		}
	}

	@keyframes fadeIn {
		from {
			opacity: 0;
		}
		to {
			opacity: 1;
		}
	}

	.heart-explosion {
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		pointer-events: none;
		z-index: 2;
	}

	.explosion-heart {
		position: absolute;
		font-size: 1.5rem;
		animation: explode 1.2s ease-out forwards;
	}

	@keyframes explode {
		0% {
			transform: translate(0, 0) scale(1);
			opacity: 1;
		}
		100% {
			transform: translate(var(--x), var(--y)) scale(1.5);
			opacity: 0;
		}
	}
</style>
