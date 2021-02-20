<script>
    export let loading;

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

    button:focus {
        outline: none;
    }
	
	.selected {
		border-bottom: 3px solid rgb(244, 114, 182);
		color: #333;
	}
</style>

<button disabled={loading} class:selected="{$selectedTab === tab}" class="w-40 text-center text-white font-bold py-2 px-4" on:click="{() => selectTab(tab)}">
	<slot></slot>
</button>