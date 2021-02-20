<script>
	import { getContext } from 'svelte';
	import { TABS } from './Tabs.svelte';
    import { fade } from 'svelte/transition';

	const panel = {};
	const { registerPanel, selectedPanel } = getContext(TABS);

	registerPanel(panel);
</script>

{#if $selectedPanel === panel}
    <div class="absolute w-screen"
        in:fade={{delay: 350, duration: 300}}
        out:fade={{delay: 0, duration: 300}}
    >
        <slot></slot>
    </div>
{/if}