<script>
	import { Type, AArrowDown, AArrowUp, Sun, Moon, MoreVertical, Square, LineChart, RulerDimensionLine, SquarePlus, SquareMinus, ListChevronsDownUp, ListChevronsUpDown, Download, Copy } from '@lucide/svelte';
	import { toPng, toCanvas } from 'html-to-image';

	let { settings = $bindable(), previewRef } = $props();

	const fontOptions = [
		{ value: 'sans-serif', label: 'Sans Serif' },
		{ value: 'serif', label: 'Serif' },
		{ value: 'mono', label: 'Monospace' }
	];

	let showWidthDropdown = $state(false);

	function increaseFontSize() {
		if (settings.fontSize < 72) settings.fontSize += 2;
	}

	function decreaseFontSize() {
		if (settings.fontSize > 12) settings.fontSize -= 2;
	}

	function increasePadding() {
		if (settings.padding < 80) settings.padding += 5;
	}

	function decreasePadding() {
		if (settings.padding > 10) settings.padding -= 5;
	}

	function increaseLineHeight() {
		if (settings.lineHeight < 3) settings.lineHeight += 0.1;
	}

	function decreaseLineHeight() {
		if (settings.lineHeight > 1) settings.lineHeight -= 0.1;
	}

	function getCurrentFontLabel() {
		return fontOptions.find(opt => opt.value === settings.fontFamily)?.label || 'Sans';
	}

	function closeAllDropdowns() {
		showWidthDropdown = false;
	}

	function handleClickOutside(event) {
		if (!event.target.closest('.dropdown-container')) {
			closeAllDropdowns();
		}
	}

	let isExporting = $state(false);
	let copySuccess = $state(false);
	let showNotification = $state(false);
	let notificationMessage = $state('');

	async function downloadImage() {
		if (!previewRef) return;

		isExporting = true;

		try {
			const canvas = await generateCanvas();

			// Download
			const link = document.createElement('a');
			link.download = `text-image-${Date.now()}.png`;
			link.href = canvas.toDataURL('image/png');
			link.click();

			// Show download notification
			showDownloadNotification();
		} catch (error) {
			console.error('Error generating image:', error);
			alert('Error al generar la imagen. Por favor, intenta nuevamente.');
		} finally {
			isExporting = false;
		}
	}

	async function generateCanvas() {
		if (!previewRef) throw new Error('Preview element not found');

		const canvas = await toCanvas(previewRef, {
			pixelRatio: 2, // High resolution
		});

		return canvas;
	}


	async function copyToClipboard() {
		if (!previewRef) return;

		isExporting = true;
		copySuccess = false;

		try {
			const canvas = await generateCanvas();

			// Copy to clipboard
			canvas.toBlob(async (blob) => {
				try {
					await navigator.clipboard.write([
						new ClipboardItem({ 'image/png': blob })
					]);
					copySuccess = true;
					showCopyNotification();
					setTimeout(() => copySuccess = false, 2000);
				} catch (error) {
					console.error('Error copying to clipboard:', error);
					alert('Error al copiar al portapapeles. Tu navegador podría no soportar esta función.');
				}
			}, 'image/png');
		} catch (error) {
			console.error('Error generating image for clipboard:', error);
			alert('Error al generar la imagen. Por favor, intenta nuevamente.');
		} finally {
			isExporting = false;
		}
	}

	function showCopyNotification() {
		notificationMessage = 'Image copied to clipboard!';
		showNotification = true;
		setTimeout(() => {
			showNotification = false;
		}, 3000);
	}

	function showDownloadNotification() {
		notificationMessage = 'Image downloaded successfully!';
		showNotification = true;
		setTimeout(() => {
			showNotification = false;
		}, 3000);
	}
</script>

<svelte:window onclick={handleClickOutside} />

<div class="flex items-center justify-between gap-4 p-2 rounded border border-gray-300  w-full">
	<!-- Left side - Editing controls -->
	<div class="flex items-center gap-1">
		<!-- Font Family Dropdown -->
	<div class="dropdown">
		<div tabindex="0" role="button" class="btn btn-square btn-sm sm:btn-md" title="Typography">
			<Type class="size-5" />
		</div>
		<ul tabindex="0" class="dropdown-content menu bg-base-100 rounded-box z-[1] w-52 p-2 shadow" >
			{#each fontOptions as option}
				<li>
					<button
						onclick={() => settings.fontFamily = option.value}
						class="{settings.fontFamily === option.value ? 'active' : ''}"
						aria-label={option.label}
					>
						{option.label}
					</button>
				</li>
			{/each}
		</ul>
	</div>

	<!-- Font Size Controls -->
	<div class="join">
		<button
			onclick={decreaseFontSize}
			class="btn btn-square  join-item btn-sm sm:btn-md"
			title="Decrease font size"
		>
			<AArrowDown class="size-5" />
		</button>
		<button
			onclick={increaseFontSize}
			class="btn btn-square  join-item btn-sm sm:btn-md"
			title="Increase font size"
		>
			<AArrowUp class="size-5"/>
		</button>
	</div>


	<!-- Theme Toggle -->
	<div class="join">
		<button
			onclick={() => settings.theme = 'light'}
			class="btn btn-square  join-item btn-sm sm:btn-md
				   {settings.theme === 'light' ? 'btn-active' : ''}"
			title="Light theme"
		>
			<Sun class="size-5" />
		</button>
		<button
			onclick={() => settings.theme = 'dark'}
			class="btn btn-square  join-item btn-sm sm:btn-md
				   {settings.theme === 'dark' ? 'btn-active' : ''}"
			title="Dark theme"
		>
			<Moon class="size-5" />
		</button>
	</div>


	<!-- Padding Controls -->
	<div class="join">
		<button
			onclick={decreasePadding}
			class="btn btn-square  join-item btn-sm sm:btn-md"
			title="Decrease padding"
		>
			<SquareMinus class="size-5" />
		</button>
		<button
			onclick={increasePadding}
			class="btn btn-square  join-item btn-sm sm:btn-md"
			title="Increase padding"
		>
			<SquarePlus class="size-5" />
		</button>
	</div>

	<!-- Line Height Controls -->
	<div class="join">
		<button
			onclick={decreaseLineHeight}
			class="btn btn-square  join-item btn-sm sm:btn-md"
			title="Decrease line height"
		>
			<ListChevronsDownUp class="size-5" />
		</button>
		<button
			onclick={increaseLineHeight}
			class="btn btn-square  join-item btn-sm sm:btn-md"
			title="Increase line height"
		>
			<ListChevronsUpDown class="size-5" />
		</button>
	</div>



	</div>

	<!-- Right side - Export controls -->
	<div class="flex items-center gap-2">
		<button
			onclick={downloadImage}
			disabled={isExporting}
			class="btn btn-square  btn-primary btn-sm sm:btn-md"
			title="Download PNG"
		>
			<Download class="size-5" />
		</button>

		<button
			onclick={copyToClipboard}
			disabled={isExporting}
			class="btn btn-square btn-success btn-sm sm:btn-md"
			title="Copy to clipboard"
		>
			<Copy class="size-5" />
		</button>
	</div>
</div>

<!-- Notification Toast -->
{#if showNotification}
	<div class="fixed top-4 right-4 z-50 transform transition-all duration-300 ease-in-out">
		<div class="alert alert-success shadow-lg flex items-center gap-3 min-w-0">
			<svg class="w-6 h-6 text-green-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
				<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
			</svg>
			<span class="text-sm font-medium">{notificationMessage}</span>
		</div>
	</div>
{/if}

<style>
	.slider::-webkit-slider-thumb {
		appearance: none;
		height: 12px;
		width: 12px;
		border-radius: 50%;
		background: #3b82f6;
		cursor: pointer;
		box-shadow: 0 1px 2px rgba(0, 0, 0, 0.2);
	}

	.slider::-moz-range-thumb {
		height: 12px;
		width: 12px;
		border-radius: 50%;
		background: #3b82f6;
		cursor: pointer;
		border: none;
		box-shadow: 0 1px 2px rgba(0, 0, 0, 0.2);
	}

	.vertical-slider {
		writing-mode: bt-lr; /* IE */
		-webkit-appearance: slider-vertical; /* WebKit */
		-moz-appearance: slider-vertical; /* Firefox */
		appearance: slider-vertical;
	}

	.vertical-slider::-webkit-slider-thumb {
		appearance: none;
		width: 12px;
		height: 12px;
		border-radius: 50%;
		background: #3b82f6;
		cursor: pointer;
		box-shadow: 0 1px 2px rgba(0, 0, 0, 0.2);
	}
</style>