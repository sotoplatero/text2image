<script>
	import { onMount, onDestroy } from 'svelte';
	import Controls from './Controls.svelte';
	import { EditorView, minimalSetup } from 'codemirror';
	import { markdown } from '@codemirror/lang-markdown';
	import { oneDark } from '@codemirror/theme-one-dark';
	import { EditorState } from '@codemirror/state';
	import { history, historyKeymap } from '@codemirror/commands';
	import { searchKeymap, highlightSelectionMatches } from '@codemirror/search';
	import { keymap, highlightActiveLine, highlightActiveLineGutter, drawSelection, highlightSpecialChars } from '@codemirror/view';
	import { defaultKeymap, indentWithTab } from '@codemirror/commands';
	import { bracketMatching, indentOnInput, syntaxHighlighting, defaultHighlightStyle } from '@codemirror/language';
	import { closeBrackets } from '@codemirror/autocomplete';
	import { autocompletion } from '@codemirror/autocomplete';

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
	let editorContainer;
	let editorView = null;

	const fontFamilyMap = {
		'sans-serif': '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif',
		'serif': 'Georgia, "Times New Roman", Times, serif',
		'mono': '"SF Mono", Monaco, "Cascadia Code", "Roboto Mono", Consolas, "Courier New", monospace'
	};


	function createEditor() {
		if (!editorContainer) return;

		const fontFamily = fontFamilyMap[settings.fontFamily];
		const isDark = settings.theme === 'dark';

		const extensions = [
			// Custom setup without line numbers
			history(),
			highlightActiveLineGutter(),
			highlightSpecialChars(),
			drawSelection(),
			EditorState.allowMultipleSelections.of(true),
			indentOnInput(),
			bracketMatching(),
			closeBrackets(),
			autocompletion(),
			highlightActiveLine(),
			highlightSelectionMatches(),
			keymap.of([
				...defaultKeymap,
				...searchKeymap,
				...historyKeymap,
				indentWithTab,
			]),
			syntaxHighlighting(defaultHighlightStyle, { fallback: true }),
			markdown(),
			EditorView.updateListener.of((update) => {
				if (update.docChanged) {
					text = update.state.doc.toString();
				}
			}),
			EditorView.theme({
				'&': {
					fontSize: `${settings.fontSize}px`,
					fontFamily: fontFamily,
					lineHeight: settings.lineHeight.toString()
				},
				'.cm-editor': {
					borderRadius: '8px',
					border: `1px solid ${isDark ? '#374151' : '#e5e7eb'}`
				},
				'.cm-focused': {
					outline: 'none',
					borderColor: '#3b82f6',
					boxShadow: '0 0 0 2px rgba(59, 130, 246, 0.2)'
				},
				'.cm-content': {
					padding: `${settings.padding}px`,
					minHeight: '200px',
					lineHeight: settings.lineHeight.toString()
				},
				'.cm-scroller': {
					fontFamily: fontFamily
				}
			})
		];

		if (isDark) {
			extensions.push(oneDark);
		}

		const state = EditorState.create({
			doc: text === 'Escribe o pega tu texto aquí...' ? '' : text,
			extensions
		});

		if (editorView) {
			editorView.destroy();
		}

		editorView = new EditorView({
			state,
			parent: editorContainer
		});

		previewRef = editorContainer;
	}

	onMount(() => {
		const savedSettings = localStorage.getItem('text2image-settings');
		if (savedSettings) {
			settings = { ...settings, ...JSON.parse(savedSettings) };
		}

		const savedText = localStorage.getItem('text2image-text');
		if (savedText) {
			text = savedText;
		}

		createEditor();
	});

	onDestroy(() => {
		if (editorView) {
			editorView.destroy();
		}
	});

	$effect(() => {
		localStorage.setItem('text2image-settings', JSON.stringify(settings));
	});

	$effect(() => {
		localStorage.setItem('text2image-text', text);
	});

	// Recreate editor when settings change
	$effect(() => {
		if (editorContainer && editorView) {
			createEditor();
		}
	});


</script>

<style>
	.editor-container {
		width: 100%;
		box-shadow: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);
		border-radius: 8px;
	}

	/* Global CodeMirror overrides */
	:global(.cm-editor.cm-focused) {
		outline: none !important;
	}
</style>

<div class="h-full flex flex-col max-w-4xl mx-auto">

	<!-- Main Content - Centered WYSIWYG Editor -->
		
	<div class="sticky top-0  flex justify-center py-4">
		<Controls bind:settings {previewRef} />
	</div>

	<div class="flex-1 flex items-center">
		<div class="flex justify-center w-full">
			<div class="editor-container" bind:this={editorContainer}></div>
		</div>
	</div>

</div>