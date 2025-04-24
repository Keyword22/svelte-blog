<script lang="ts">
	import { fade } from 'svelte/transition';
	
	let options: string[] = [];
	let newOption: string = '';
	let spinning: boolean = false;
	let result: string = '';
	let spinAngle: number = 0;
	let showResult: boolean = false;

	const colors = [
		'#FF3E00', '#40B3FF', '#676778', '#FF40B3', 
		'#B3FF40', '#4040FF', '#FFB340', '#40FFB3',
		'#B340FF', '#FF4040', '#40FF40', '#4040B3'
	];

	function addOptions() {
		const newOptions = newOption
			.split(',')
			.map(opt => opt.trim())
			.filter(opt => opt && !options.includes(opt));
		
		if (newOptions.length > 0) {
			options = [...options, ...newOptions];
			newOption = '';
		}
	}

	function removeOption(option: string) {
		options = options.filter(o => o !== option);
	}

	function spin() {
		if (options.length < 2) {
			alert('Please add at least 2 options');
			return;
		}

		spinning = true;
		showResult = false;
		spinAngle = 0;
		
		// First, determine the winner
		const winningIndex = Math.floor(Math.random() * options.length);
		result = options[winningIndex];

		// Calculate the angle range for the winning segment
		const segmentSize = 360 / options.length;
		const winningSegmentStart = 360 - (winningIndex * segmentSize);
		
		// Pick a random position within the winning segment
		const randomPositionInSegment = Math.random() * segmentSize;
		const finalAngle = winningSegmentStart - randomPositionInSegment;
		
		// Add some full rotations and calculate target angle
		const fullRotations = 5;
		const targetAngle = (360 * fullRotations) + finalAngle;
		
		let currentSpeed = 15;
		const minSpeed = 2;
		const deceleration = 0.98;
		const startSlowdownAt = targetAngle * 0.7; // Start slowing down at 70% of the way

		const spinInterval = setInterval(() => {
			spinAngle += currentSpeed;

			// If we're past the slowdown point, start decreasing speed
			if (spinAngle >= startSlowdownAt) {
				currentSpeed = Math.max(minSpeed, currentSpeed * deceleration);
			}

			if (spinAngle >= targetAngle) {
				clearInterval(spinInterval);
				spinning = false;
				// Show result after a small delay
				setTimeout(() => {
					showResult = true;
				}, 500);
			}
		}, 20);
	}

	$: sliceAngle = 360 / options.length;
</script>

<svelte:head>
	<title>Roulette Game</title>
	<meta name="description" content="Spin the roulette and get a random result" />
</svelte:head>

<section>
	<h1>Roulette Game</h1>

	<div class="input-section">
		<textarea
			bind:value={newOption}
			placeholder="Enter options separated by commas"
			rows="3"
		></textarea>
		<button on:click={addOptions}>Add Options</button>
	</div>

	<div class="options-list">
		{#each options as option}
			<div class="option-item">
				<span>{option}</span>
				<button on:click={() => removeOption(option)}>×</button>
			</div>
		{/each}
	</div>

	<div class="roulette-container">
		<div class="arrow"></div>
		<div
			class="roulette-wheel"
			style="transform: rotate({spinAngle}deg)"
			class:spinning
		>
			<svg width="100%" height="100%" viewBox="0 0 100 100">
				{#each options as option, i}
					{@const startAngle = i * sliceAngle}
					{@const endAngle = (i + 1) * sliceAngle}
					{@const startRad = (startAngle - 90) * Math.PI / 180}
					{@const endRad = (endAngle - 90) * Math.PI / 180}
					{@const x1 = 50 + 50 * Math.cos(startRad)}
					{@const y1 = 50 + 50 * Math.sin(startRad)}
					{@const x2 = 50 + 50 * Math.cos(endRad)}
					{@const y2 = 50 + 50 * Math.sin(endRad)}
					{@const largeArc = sliceAngle > 180 ? 1 : 0}
					<path
						d="M 50 50 L {x1} {y1} A 50 50 0 {largeArc} 1 {x2} {y2} Z"
						fill={colors[i % colors.length]}
					/>
					{@const textAngle = startAngle + sliceAngle / 2}
					{@const textRad = (textAngle - 90) * Math.PI / 180}
					{@const textX = 50 + 35 * Math.cos(textRad)}
					{@const textY = 50 + 35 * Math.sin(textRad)}
					<text
						x={textX}
						y={textY}
						text-anchor="middle"
						dominant-baseline="middle"
						fill="white"
						font-size="4"
						transform="rotate({textAngle}, {textX}, {textY})"
					>
						{option}
					</text>
				{/each}
			</svg>
		</div>
	</div>

	<button class="spin-button" on:click={spin} disabled={spinning}>
		{spinning ? 'Spinning...' : 'Spin!'}
	</button>

	{#if showResult && !spinning}
		<div class="result" in:fade>
			Result: {result}
		</div>
	{/if}
</section>

<style>
	section {
		display: flex;
		flex-direction: column;
		align-items: center;
		padding: 2rem;
		gap: 2rem;
	}

	h1 {
		font-size: 2.5rem;
		margin-bottom: 1rem;
	}

	.input-section {
		display: flex;
		flex-direction: column;
		gap: 1rem;
		margin-bottom: 1rem;
		width: 100%;
		max-width: 500px;
	}

	textarea {
		padding: 0.5rem;
		font-size: 1rem;
		border: 2px solid #ccc;
		border-radius: 4px;
		resize: vertical;
		min-height: 100px;
	}

	button {
		padding: 0.5rem 1rem;
		font-size: 1rem;
		background-color: #ff3e00;
		color: white;
		border: none;
		border-radius: 4px;
		cursor: pointer;
	}

	button:disabled {
		background-color: #ccc;
		cursor: not-allowed;
	}

	.options-list {
		display: flex;
		flex-wrap: wrap;
		gap: 0.5rem;
		margin-bottom: 2rem;
	}

	.option-item {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		background-color: #f0f0f0;
		padding: 0.5rem 1rem;
		border-radius: 4px;
	}

	.option-item button {
		background: none;
		color: #666;
		padding: 0;
		font-size: 1.2rem;
	}

	.roulette-container {
		position: relative;
		width: 300px;
		height: 300px;
		margin: 2rem 0;
	}

	.arrow {
		position: absolute;
		top: -20px;
		left: 50%;
		transform: translateX(-50%);
		width: 0;
		height: 0;
		border-left: 15px solid transparent;
		border-right: 15px solid transparent;
		border-top: 30px solid #ff3e00;
		z-index: 2;
	}

	.roulette-wheel {
		position: absolute;
		width: 100%;
		height: 100%;
		border-radius: 50%;
		border: 4px solid #333;
		transition: transform 0.1s linear;
		overflow: hidden;
	}

	.roulette-wheel.spinning {
		transition: transform 0.1s linear;
	}

	.spin-button {
		font-size: 1.5rem;
		padding: 1rem 2rem;
	}

	.result {
		font-size: 1.5rem;
		font-weight: bold;
		margin-top: 1rem;
		text-align: center;
		padding: 1rem;
		background-color: #ff3e00;
		color: white;
		border-radius: 4px;
		box-shadow: 0 2px 4px rgba(0,0,0,0.2);
	}
</style>
