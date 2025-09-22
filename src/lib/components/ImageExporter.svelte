<script>
	import { onMount } from 'svelte';

	let { previewRef, settings } = $props();

	let isExporting = $state(false);
	let copySuccess = $state(false);

	async function downloadImage() {
		if (!previewRef) return;

		isExporting = true;

		try {
			// Dynamic import of html2canvas
			const html2canvas = await import('html2canvas');

			const canvas = await html2canvas.default(previewRef, {
				backgroundColor: settings.theme === 'dark' ? '#1f2937' : '#ffffff',
				scale: 2, // Higher resolution
				useCORS: true,
				allowTaint: true,
				logging: false
			});

			// Create download link
			const link = document.createElement('a');
			link.download = `text-image-${Date.now()}.png`;
			link.href = canvas.toDataURL('image/png');
			link.click();
		} catch (error) {
			console.error('Error generating image:', error);
			alert('Error al generar la imagen. Por favor, intenta nuevamente.');
		} finally {
			isExporting = false;
		}
	}

	async function copyToClipboard() {
		if (!previewRef) return;

		isExporting = true;
		copySuccess = false;

		try {
			// Dynamic import of html2canvas
			const html2canvas = await import('html2canvas');

			const canvas = await html2canvas.default(previewRef, {
				backgroundColor: settings.theme === 'dark' ? '#1f2937' : '#ffffff',
				scale: 2,
				useCORS: true,
				allowTaint: true,
				logging: false
			});

			// Convert canvas to blob
			canvas.toBlob(async (blob) => {
				try {
					await navigator.clipboard.write([
						new ClipboardItem({ 'image/png': blob })
					]);
					copySuccess = true;
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
</script>

<div class="flex gap-2">
	<button
		onclick={downloadImage}
		disabled={isExporting}
		class="bg-blue-600 hover:bg-blue-700 disabled:bg-blue-400
			   text-white text-sm font-medium py-2 px-3 rounded
			   transition-colors duration-200
			   flex items-center gap-1"
	>
		{#if isExporting}
			<svg class="animate-spin h-4 w-4" fill="none" viewBox="0 0 24 24">
				<circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
				<path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
			</svg>
			Generando...
		{:else}
			<svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
				<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 10v6m0 0l-3-3m3 3l3-3m2 8H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
			</svg>
			Descargar PNG
		{/if}
	</button>

	<button
		onclick={copyToClipboard}
		disabled={isExporting}
		class="bg-green-600 hover:bg-green-700 disabled:bg-green-400
			   text-white text-sm font-medium py-2 px-3 rounded
			   transition-colors duration-200
			   flex items-center gap-1
			   {copySuccess ? 'bg-green-700' : ''}"
	>
		{#if isExporting}
			<svg class="animate-spin h-4 w-4" fill="none" viewBox="0 0 24 24">
				<circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
				<path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
			</svg>
			Copiando...
		{:else if copySuccess}
			<svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
				<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
			</svg>
			¡Copiado!
		{:else}
			<svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
				<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z" />
			</svg>
			Copiar imagen
		{/if}
	</button>
</div>