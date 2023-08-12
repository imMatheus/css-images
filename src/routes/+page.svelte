<script lang="ts">
	let selectedImage: string | undefined =
		'https://www.politico.com/dims4/default/96ef7e6/2147483647/strip/true/crop/5404x3603+0+0/resize/1260x840!/quality/90/?url=https%3A%2F%2Fstatic.politico.com%2Fef%2Fd2%2F5c2efdae4430a0a8c8bff09e1a1b%2Felection-2024-trump-09443.jpg';

	function generateRandomColor(): string {
		const letters = '0123456789ABCDEF';
		let color = '#';

		for (let i = 0; i < 6; i++) {
			color += letters[Math.floor(Math.random() * 16)];
		}

		return color;
	}

	let blocksX = 20;
	let blocksY = 50;
	$: totalBlocks = blocksX * blocksY;
	$: blocks = Array(totalBlocks).fill(null).map(generateRandomColor);

	let imageColors: string[] = [];
	let img: HTMLImageElement | null = null;
	let currentPixelIndex = 0;
	const pixelsPerFrame = 1000; // Adjust this number as needed

	async function processImageChunk() {
		if (!img) return;

		const canvas = document.createElement('canvas');
		canvas.width = img.width;
		canvas.height = img.height;
		const context = canvas.getContext('2d');

		if (context) {
			context.drawImage(img, 0, 0);
			const imageData = context.getImageData(0, 0, img.width, img.height);
			const data = imageData.data;

			for (
				let i = currentPixelIndex;
				i < currentPixelIndex + pixelsPerFrame && i < data.length;
				i += 4
			) {
				const color = `rgb(${data[i]}, ${data[i + 1]}, ${data[i + 2]})`;
				imageColors.push(color);
			}

			currentPixelIndex += pixelsPerFrame;

			imageColors = [...imageColors];
			if (currentPixelIndex < data.length) {
				requestAnimationFrame(processImageChunk);
			}
		}
	}

	// Call the function when selectedImage changes
	$: {
		console.log('asass');

		imageColors.length = 0; // Clear previous image colors
		currentPixelIndex = 0;
		if (selectedImage) {
			console.log('aaaa');
			processImageChunk();
		}
	}

	console.log(imageColors);

	function handleImageLoad(event: Event) {
		img = event.target as HTMLImageElement;
	}

	const handleImageChange = (event: Event) => {
		const inputElement = event.target as HTMLInputElement;
		const file = inputElement.files && inputElement.files[0];
		handleImageLoad(event);
		if (file) {
			const reader = new FileReader();
			reader.onload = (e) => {
				selectedImage = e.target?.result as string;
			};
			reader.readAsDataURL(file);
		}
	};
</script>

<svelte:head>
	<title>CSS images</title>
	<meta name="description" content="Svelte demo app" />
</svelte:head>

<div class="p-32">
	<h2 class="text-5xl font-black tracking-wider text-center">Choose your image</h2>

	<div class="h-96 grid grid-cols-2 bg-black">
		<div class="bg-red-500 relative">
			{#if selectedImage}
				<img
					src={selectedImage}
					alt="Selected"
					class="w-full h-full object-cover absolute"
					crossOrigin="anonymous"
				/>
			{/if}
		</div>
		<div class="bg-blue-500 flex flex-wrap">
			<!-- style="grid-template-columns: repeat({blocksX}, 1fr); grid-template-rows: repeat({blocksY}, 1fr);" -->
			<!-- {#each blocks as block}
				<div class="w-[1px] h-[1px]" style="background: {block}" />
			{/each} -->

			{#each imageColors as color}
				<div class="w-[1px] h-[1px]" style="background: {color}" />
			{/each}
		</div>
	</div>
	<label for="avatar">Upload a picture: {totalBlocks}</label>
	<input type="file" accept="image/*" on:change={handleImageChange} />

	<div class="my-4">
		<label for="xBlocks">X blocks</label>
		<input id="xBlocks" type="range" min="5" max="300" step="10" bind:value={blocksX} />
	</div>
	<div>
		<label for="yBlocks">X blocks</label>
		<input id="yBlocks" type="range" min="5" max="300" step="10" bind:value={blocksY} />
	</div>
</div>

<img
	src={selectedImage}
	alt="Selected"
	on:load={handleImageLoad}
	style="display: none;"
	crossOrigin="anonymous"
/>
