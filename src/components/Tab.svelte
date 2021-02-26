<script>
    export let loading;
    export let lable;

	import { getContext } from 'svelte';
	import { TABS } from './Tabs.svelte';

	const tab = {};
	const { registerTab, selectTab, selectedTab } = getContext(TABS);

	registerTab(tab);
</script>

<style>
	button {
		border: none;
		border-bottom: 2px solid white;
		margin: 0;
	}

    button:focus {
        outline: none;
    }
	
	.selected {
		border-bottom: 3px solid rgb(244, 114, 182);
		color: #333;
	}
</style>

<button disabled={loading} class:selected="{$selectedTab === tab}" class:text-pink-300={lable} class="w-40 text-gray-400 text-center font-bold py-2 px-4" on:click="{() => selectTab(tab)}">
	<slot></slot>
</button>