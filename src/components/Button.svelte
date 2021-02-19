<script>
	export let loading;
</script>

<button on:click class="w-40 text-center bg-pink-400 hover:bg-blue-dark text-white font-bold py-2 px-4 rounded">
    {#if loading}
        <i class="fas fa-sync-alt animate-spin"></i>
		{:else}
            <slot></slot>
	{/if}
</button>