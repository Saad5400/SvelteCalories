<script lang="ts">
    import { onMount } from 'svelte';
	import '../app.postcss';
	import { Toast, autoModeWatcher } from '@skeletonlabs/skeleton';
	import { initializeStores } from '@skeletonlabs/skeleton';

	initializeStores();
</script>

<Toast />

<svelte:head>{@html `<script>${autoModeWatcher.toString()} autoModeWatcher();</script>`}</svelte:head>

<div class="flex flex-col min-h-[100dvh] items-center justify-between py-40">
	<slot />
</div>
