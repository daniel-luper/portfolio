<script lang="ts">
	import HeaderLinks from './HeaderLinks.svelte';

	export let open: boolean = false;
</script>

{#if open}
	<div
		class="center-page"
		on:wheel|nonpassive={(e) => {
			if (open) e.preventDefault();
		}}
	>
		<HeaderLinks
			on:click={(e) => {
				open = false;
			}}
		/>
	</div>
{/if}

<style lang="scss">
	@import '../../scss/breakpoints.scss';

	.center-page {
		display: flex;
		justify-content: center;
		align-items: center;
		line-height: 300%;
		height: 90vh;
	}
</style>
