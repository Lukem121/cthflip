<script>
	import { getContext } from 'svelte';
	import { TABS } from './Tabs.svelte';
    import { fade } from 'svelte/transition';

	const panel = {};
	const { registerPanel, selectedPanel } = getContext(TABS);

	registerPanel(panel);
</script>

{#if $selectedPanel === panel}
    <div 
        in:fade={{delay: 350, duration: 350}}
        out:fade={{delay: 0, duration: 300}}
    >
        <slot></slot>
    </div>
{/if}