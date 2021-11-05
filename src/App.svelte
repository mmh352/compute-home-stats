<script lang="ts">
	import Dashboard from './components/Dashboard.svelte';
</script>

<main class="w-screen h-screen flex flex-col">
	<div class="flex-0 bg-blue-700 text-white px-3 py-1">
		<h1 class="text-xl">Compute Home Usage Dashboard</h1>
	</div>
	<div class="flex-auto px-3 py-2 overflow-hidden">
		<Dashboard/>
	</div>
</main>

<style global lang="postcss">
	@tailwind base;
	@tailwind components;
	@tailwind utilities;
</style>
