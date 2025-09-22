<script>
	import { onMount } from 'svelte';
	import Controls from './Controls.svelte';

	let settings = $state({
		fontFamily: 'sans-serif',
		fontSize: 16,
		theme: 'light',
		padding: 20,
		lineHeight: 1.5,
		imageWidth: 800
	});

	let text = $state('Escribe o pega tu texto aquí...');
	let previewRef;

	onMount(() => {
		const savedSettings = localStorage.getItem('text2image-settings');
		if (savedSettings) {
			settings = { ...settings, ...JSON.parse(savedSettings) };
		}

		const savedText = localStorage.getItem('text2image-text');
		if (savedText) {
			text = savedText;
		}
	});

	$effect(() => {
		localStorage.setItem('text2image-settings', JSON.stringify(settings));
	});

	$effect(() => {
		localStorage.setItem('text2image-text', text);
	});

	const fontFamilyMap = {
		'sans-serif': '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif',
		'serif': 'Georgia, "Times New Roman", Times, serif',
		'mono': '"SF Mono", Monaco, "Cascadia Code", "Roboto Mono", Consolas, "Courier New", monospace'
	};

	const previewStyle = $derived(`
		font-family: ${fontFamilyMap[settings.fontFamily]};
		font-size: ${settings.fontSize}px;
		line-height: ${settings.lineHeight};
		padding: ${settings.padding}px;
		background-color: ${settings.theme === 'dark' ? '#1f2937' : '#ffffff'} !important;
		color: ${settings.theme === 'dark' ? '#f9fafb' : '#111827'} !important;
		border: 1px solid ${settings.theme === 'dark' ? '#374151' : '#e5e7eb'};
		border-radius: 8px;
		word-wrap: break-word;
		white-space: pre-wrap;
		min-height: 200px;
		outline: none;
	`);

	function handleInput(event) {
		// Store cursor position before updating state
		const selection = window.getSelection();
		const range = selection.getRangeAt(0);
		const offset = range.startOffset;
		const container = range.startContainer;

		// Update the text state
		text = event.target.textContent || '';

		// Restore cursor position after next tick
		setTimeout(() => {
			try {
				const newRange = document.createRange();
				newRange.setStart(container, Math.min(offset, container.textContent?.length || 0));
				newRange.collapse(true);
				selection.removeAllRanges();
				selection.addRange(newRange);
			} catch (e) {
				// Fallback: place cursor at end
				const newRange = document.createRange();
				newRange.selectNodeContents(event.target);
				newRange.collapse(false);
				selection.removeAllRanges();
				selection.addRange(newRange);
			}
		}, 0);
	}

	function handleFocus(event) {
		// Only clear placeholder text if it's the default placeholder
		if (event.target.textContent === 'Click here to write your text...') {
			event.target.textContent = '';
			text = '';
		}
	}

	function handleBlur(event) {
		// Only show placeholder if truly empty
		if (event.target.textContent.trim() === '') {
			text = 'Click here to write your text...';
		}
	}
</script>

<div class="h-full flex flex-col max-w-4xl mx-auto">

	<!-- Main Content - Centered WYSIWYG Editor -->
		
	<div class="sticky top-0  flex justify-center py-4">
		<Controls bind:settings {previewRef} />
	</div>

	<div class="flex-1 flex items-center justify-center py-8">

		<div >
			<div class="flex justify-center">
				<div
				bind:this={previewRef}
					style={previewStyle}
					class="shadow-lg cursor-text focus:ring-2 focus:ring-blue-500 transition-all"
					contenteditable="true"
					oninput={handleInput}
					onfocus={handleFocus}
					onblur={handleBlur}
				>
					{text || 'Click here to write your text...'}
				</div>
			</div>
		</div>
	</div>

</div>