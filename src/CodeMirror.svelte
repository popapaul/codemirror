<script lang="ts">
	import { onMount } from 'svelte';
	import { browser } from '$app/environment';
	import {
		EditorView,
		lineNumbers,
		highlightActiveLineGutter,
		highlightSpecialChars,
		drawSelection,
		dropCursor,
		rectangularSelection,
		crosshairCursor,
		highlightActiveLine,
		keymap
	} from '@codemirror/view';
	import { EditorState } from '@codemirror/state';
	import {
		foldGutter,
		indentOnInput,
		syntaxHighlighting,
		defaultHighlightStyle,
		bracketMatching,
		foldKeymap
	} from '@codemirror/language';
	import { history, defaultKeymap, historyKeymap, indentWithTab } from '@codemirror/commands';
	import { highlightSelectionMatches, searchKeymap } from '@codemirror/search';
	import { lintKeymap } from '@codemirror/lint';
	import {
		closeBrackets,
		autocompletion,
		closeBracketsKeymap,
		completionKeymap
	} from '@codemirror/autocomplete';
	import { oneDarkTheme, oneDark } from '@codemirror/theme-one-dark';

	onMount(async () => {
		if (!browser) return;

		const langMode = async () => {
			switch (mode) {
				case 'CSS':
					return import('@codemirror/lang-css').then((x) => x.css());
				case 'JS':
					return import('@codemirror/lang-javascript').then((x) => x.javascript());
				case 'HTML':
					return import('@codemirror/lang-html').then((x) => x.html());
			}
		};

		let timer;
		editor = new EditorView({
			state: EditorState.create({
				doc: await beautify(value),
				extensions: [
					EditorView.lineWrapping,

					lineNumbers(),
					highlightActiveLineGutter(),
					highlightSpecialChars(),
					history(),
					foldGutter(),
					drawSelection(),
					dropCursor(),
					EditorState.allowMultipleSelections.of(true),
					indentOnInput(),
					syntaxHighlighting(defaultHighlightStyle, { fallback: true }),
					bracketMatching(),
					closeBrackets(),
					autocompletion(),
					rectangularSelection(),
					crosshairCursor(),
					oneDark,
					highlightActiveLine(),
					highlightSelectionMatches(),
					keymap.of([
						indentWithTab,
						...closeBracketsKeymap,
						...defaultKeymap,
						...searchKeymap,
						...historyKeymap,
						...foldKeymap,
						...completionKeymap,
						...lintKeymap
					]),
					await langMode(),
					EditorView.updateListener.of((v) => {
						if (!v.docChanged) return;
						if (timer) clearTimeout(timer);

						timer = setTimeout(() => {
							oldValue = editor.state.doc.toString();
							value = oldValue;
						}, 300);
					}),
					...extensions
				]
			}),
			parent: element
		});
	});

	let klass = '';
	export { klass as class };
	export let value = '';
	let editor: EditorView = null;
	export let extensions = [];
	export let mode: 'JS' | 'HTML' | 'CSS' = 'HTML';
	let element: HTMLDivElement;
	let oldValue: string;

	const beautify = async (code: string) => {
		const beautify = (await import('js-beautify')).default;

		switch (mode) {
			case 'CSS':
				return beautify.css(code);
			case 'JS':
				return beautify.js(code);
			case 'HTML':
				return beautify.html(code, { indent_inner_html: true });
		}
	};

	const setValue = async (code: string) => {
		code = await beautify(code);
		editor.dispatch({ changes: { from: 0, to: editor.state.doc.length, insert: code } });
	};

	$: oldValue != value && editor && setValue(value);
</script>

<div bind:this={element} class={klass}></div>

<style>
	:global(.cm-editor) {
		height: 100%;
	}
	.ͼc,
	.ͼd {
		color: #d5bc41;
	}
	.ͼb {
		color: #1affca;
	}
	.ͼj {
		color: #22c3e3;
	}
	.ͼg {
		color: rgb(128, 128, 255);
	}
</style>
