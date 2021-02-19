<script>
	import { getContext } from 'svelte';
	import { TABS } from './Tabs.svelte';

	const tab = {};
	const { registerTab, selectTab, selectedTab } = getContext(TABS);

	registerTab(tab);
</script>

<style>
	button {
		background: none;
		border: none;
		border-bottom: 2px solid white;
		border-radius: 0;
		margin: 0;
		color: #ccc;
	}
	
	.selected {
		border-bottom: 2px solid teal;
		color: #333;
	}
</style>

<button class:selected="{$selectedTab === tab}" class="w-40 text-center text-white font-bold py-2 px-4" on:click="{() => selectTab(tab)}">
	<slot></slot>
</button>