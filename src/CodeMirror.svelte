<script lang="ts">
	import { run } from 'svelte/legacy';

	import { onMount, untrack } from 'svelte';
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
	import { oneDark } from '@codemirror/theme-one-dark';
	import { debounce } from '$utils/debounce';
	
	onMount(async () => {
		if (!browser) return;

		const langMode = async () => {
			switch (mode.toUpperCase()) {
				case 'CSS':
					return import('@codemirror/lang-css').then((x) => x.css());
				case 'JS':
					return import('@codemirror/lang-javascript').then((x) => x.javascript());
				default:
					return import('@codemirror/lang-html').then((x) => x.html());
			}
		};

		let timer;
		editor = new EditorView({
			state: EditorState.create({
				doc: await beautify(value),
				extensions: [
					oneDark,
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
					bracketMatching(),
					closeBrackets(),
					autocompletion(),
					rectangularSelection(),
					crosshairCursor(),
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
						handleChange(editor.state.doc.toString());
					})
				]
			}),
			parent: element
		});
	});

	
	let editor: EditorView = $state(null);
	interface Props {
		class?: string;
		value?: string;
		extensions?: any;
		mode?: 'JS' | 'HTML' | 'CSS';
	}

	let {
		class: klass = '',
		value = $bindable(),
		extensions = [],
		mode = 'HTML'
	}: Props = $props();
	let element: HTMLDivElement = $state();
	let oldValue: string = $state();

	const beautify = async (code: string) => {
		const beautify = (await import('js-beautify')).default;

		switch (mode) {
			case 'CSS':
				return beautify.css(code);
			case 'JS':
				return beautify.js(code);
			case 'HTML':
				return beautify.html(code, {
					indent_inner_html: true,
					indent_body_inner_html: true,
					indent_with_tabs: true,
					preserve_newlines: true
				});
		}
	};

	const setValue = async (code: string) => {
		code = await beautify(code);
		editor.dispatch({ changes: { from: 0, to: editor.state.doc.length, insert: code } });
	};

	const handleChange = debounce((code:string )=>{
		oldValue = code;
		value = oldValue;
	 }, 300);
	 $effect(()=>{
		if(!editor) return;
		untrack(() => {
			if (value == oldValue) return;
			setValue(value);
			oldValue = value;
		});
	 })
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
