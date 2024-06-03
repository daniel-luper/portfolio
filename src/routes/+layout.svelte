<script>
	import Footer from '$lib/components/organisms/Footer.svelte';
	import '$lib/scss/global.scss';
	import Header from '$lib/components/organisms/Header.svelte';
</script>

<div class="page">
	<Header />

	<main>
		<slot />
	</main>

	<div class="spacer" />

	<Footer />
</div>

<style>
	.page {
		display: flex;
		flex-direction: column;
		min-height: 100vh;
	}

	.spacer {
		flex: 1;
	}
</style>