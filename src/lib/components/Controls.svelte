<script>
	import { Type, AArrowDown, AArrowUp, Sun, Moon, MoreVertical, Square, LineChart, RulerDimensionLine, SquarePlus, SquareMinus, ListChevronsDownUp, ListChevronsUpDown, Download, Copy } from '@lucide/svelte';

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
			const canvas = document.createElement('canvas');
			const ctx = canvas.getContext('2d');

			// Set canvas size
			const scale = 2; // For high resolution
			canvas.width = settings.imageWidth * scale;
			canvas.height = Math.max(200, calculateTextHeight()) * scale;

			// Scale context for high resolution
			ctx.scale(scale, scale);

			// Fill background
			ctx.fillStyle = settings.theme === 'dark' ? '#1f2937' : '#ffffff';
			ctx.fillRect(0, 0, settings.imageWidth, canvas.height / scale);

			// Set text properties
			const fontFamilyMap = {
				'sans-serif': 'Arial, Helvetica, sans-serif',
				'serif': 'Georgia, Times New Roman, serif',
				'mono': 'Consolas, Courier New, monospace'
			};
			const fontFamily = fontFamilyMap[settings.fontFamily] || 'Arial, sans-serif';
			ctx.font = `${settings.fontSize}px ${fontFamily}`;
			ctx.fillStyle = settings.theme === 'dark' ? '#f9fafb' : '#111827';
			ctx.textBaseline = 'top';

			// Draw text with line wrapping
			const text = previewRef.textContent || 'Click here to write your text...';
			drawWrappedText(ctx, text, settings.padding, settings.padding,
							settings.imageWidth - (settings.padding * 2), settings.lineHeight * settings.fontSize);

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

	function calculateTextHeight() {
		const text = previewRef?.textContent || '';

		// Count manual line breaks
		const paragraphs = text.split('\n');
		let totalLines = 0;

		// Estimate character width (rough approximation)
		const avgCharWidth = settings.fontSize * 0.6;
		const maxCharsPerLine = Math.floor((settings.imageWidth - (settings.padding * 2)) / avgCharWidth);

		paragraphs.forEach(paragraph => {
			if (paragraph.trim() === '') {
				totalLines += 1; // Empty line
			} else {
				// Calculate wrapped lines for this paragraph
				const wrappedLines = Math.max(1, Math.ceil(paragraph.length / maxCharsPerLine));
				totalLines += wrappedLines;
			}
		});

		const lineHeight = settings.lineHeight * settings.fontSize;
		return (totalLines * lineHeight) + (settings.padding * 2) + 20;
	}

	function drawWrappedText(ctx, text, x, y, maxWidth, lineHeight) {
		// Split by line breaks first to preserve manual line breaks
		const paragraphs = text.split('\n');
		let currentY = y;

		paragraphs.forEach((paragraph, paragraphIndex) => {
			if (paragraph.trim() === '') {
				// Empty line - just advance Y position
				currentY += lineHeight;
				return;
			}

			const words = paragraph.split(' ');
			let line = '';

			for (let i = 0; i < words.length; i++) {
				const testLine = line + words[i] + ' ';
				const metrics = ctx.measureText(testLine);
				const testWidth = metrics.width;

				if (testWidth > maxWidth && i > 0) {
					// Draw current line and start new one
					ctx.fillText(line.trim(), x, currentY);
					line = words[i] + ' ';
					currentY += lineHeight;
				} else {
					line = testLine;
				}
			}

			// Draw remaining text in line
			if (line.trim()) {
				ctx.fillText(line.trim(), x, currentY);
			}

			// Move to next line for next paragraph
			currentY += lineHeight;
		});
	}

	async function copyToClipboard() {
		if (!previewRef) return;

		isExporting = true;
		copySuccess = false;

		try {
			const canvas = document.createElement('canvas');
			const ctx = canvas.getContext('2d');

			// Set canvas size
			const scale = 2; // For high resolution
			canvas.width = settings.imageWidth * scale;
			canvas.height = Math.max(200, calculateTextHeight()) * scale;

			// Scale context for high resolution
			ctx.scale(scale, scale);

			// Fill background
			ctx.fillStyle = settings.theme === 'dark' ? '#1f2937' : '#ffffff';
			ctx.fillRect(0, 0, settings.imageWidth, canvas.height / scale);

			// Set text properties
			const fontFamilyMap = {
				'sans-serif': 'Arial, Helvetica, sans-serif',
				'serif': 'Georgia, Times New Roman, serif',
				'mono': 'Consolas, Courier New, monospace'
			};
			const fontFamily = fontFamilyMap[settings.fontFamily] || 'Arial, sans-serif';
			ctx.font = `${settings.fontSize}px ${fontFamily}`;
			ctx.fillStyle = settings.theme === 'dark' ? '#f9fafb' : '#111827';
			ctx.textBaseline = 'top';

			// Draw text with line wrapping
			const text = previewRef.textContent || 'Click here to write your text...';
			drawWrappedText(ctx, text, settings.padding, settings.padding,
							settings.imageWidth - (settings.padding * 2), settings.lineHeight * settings.fontSize);

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

<div class="flex items-center justify-between gap-4 p-2 bg-gray-100 dark:bg-gray-700 rounded border border-gray-300 dark:border-gray-600 w-full">
	<!-- Left side - Editing controls -->
	<div class="flex items-center gap-2">
		<!-- Font Family Dropdown -->
	<div class="dropdown">
		<div tabindex="0" role="button" class="btn btn-square" title="Typography">
			<Type class="size-6" />
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
			class="btn btn-square  join-item"
			title="Decrease font size"
		>
			<AArrowDown class="size-6" />
		</button>
		<button
			onclick={increaseFontSize}
			class="btn btn-square  join-item"
			title="Increase font size"
		>
			<AArrowUp class="size-6"/>
		</button>
	</div>


	<!-- Theme Toggle -->
	<div class="join">
		<button
			onclick={() => settings.theme = 'light'}
			class="btn btn-square  join-item
				   {settings.theme === 'light' ? 'btn-active' : ''}"
			title="Light theme"
		>
			<Sun class="size-6" />
		</button>
		<button
			onclick={() => settings.theme = 'dark'}
			class="btn btn-square  join-item
				   {settings.theme === 'dark' ? 'btn-active' : ''}"
			title="Dark theme"
		>
			<Moon class="size-6" />
		</button>
	</div>


	<!-- Padding Controls -->
	<div class="join">
		<button
			onclick={decreasePadding}
			class="btn btn-square  join-item"
			title="Decrease padding"
		>
			<SquareMinus class="size-6" />
		</button>
		<button
			onclick={increasePadding}
			class="btn btn-square  join-item"
			title="Increase padding"
		>
			<SquarePlus class="size-6" />
		</button>
	</div>

	<!-- Line Height Controls -->
	<div class="join">
		<button
			onclick={decreaseLineHeight}
			class="btn btn-square  join-item"
			title="Decrease line height"
		>
			<ListChevronsDownUp class="size-6" />
		</button>
		<button
			onclick={increaseLineHeight}
			class="btn btn-square  join-item"
			title="Increase line height"
		>
			<ListChevronsUpDown class="size-6" />
		</button>
	</div>


	<!-- Width Dropdown -->
	<div class="relative dropdown-container">
		<button
			onclick={() => showWidthDropdown = !showWidthDropdown}
			class="btn btn-square "
			title="Width: {settings.imageWidth}px"
		>
			<RulerDimensionLine class="size-6" />
		</button>
		{#if showWidthDropdown}
			<div class="absolute top-full left-0 mt-1 bg-white dark:bg-gray-600 border border-gray-300 dark:border-gray-500 rounded shadow-lg z-10 p-3">
				<div class="flex flex-col items-center gap-2">
					<span class="text-xs text-gray-600 dark:text-gray-300">Image width</span>
					<input
						type="range"
						bind:value={settings.imageWidth}
						min="400"
						max="1200"
						step="50"
						class="h-20 w-1 bg-gray-300 rounded slider vertical-slider"
					/>
					<span class="text-xs text-gray-500 dark:text-gray-400">{settings.imageWidth}px</span>
				</div>
			</div>
		{/if}
		</div>
	</div>

	<!-- Right side - Export controls -->
	<div class="flex items-center gap-2">
		<button
			onclick={downloadImage}
			disabled={isExporting}
			class="btn btn-square  btn-primary"
			title="Download PNG"
		>
			<Download class="size-6" />
		</button>

		<button
			onclick={copyToClipboard}
			disabled={isExporting}
			class="btn btn-square  {copySuccess ? 'btn-success' : ''} btn-success"
			title="Copy to clipboard"
		>
			<Copy class="size-6" />
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