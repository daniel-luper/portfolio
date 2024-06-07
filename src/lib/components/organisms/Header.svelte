<script>
	import Logo from '$lib/components/atoms/Logo.svelte';

	import { Hamburger } from 'svelte-hamburgers';
	import HeaderLinks from '../molecules/HeaderLinks.svelte';
	import MobileNav from '../molecules/MobileNav.svelte';

	let open = false;

	export let showBackground = false;
</script>

<header class:has-background={showBackground}>
	<nav class="container">
		<a class="logo" href="/" aria-label="Site logo" on:click={(e) => {open = false;}}>
			<Logo />
		</a>
		<div class="desktop">
			<HeaderLinks />
		</div>
		<div class="mobile">
			<Hamburger bind:open --layer-height="2px" --layer-width="24px" />
		</div>
	</nav>
	<div class="mobile">
		<MobileNav bind:open />
	</div>
</header>

<style lang="scss">
	@import '$lib/scss/breakpoints.scss';

	header {
		position: relative;
		padding: 30px 0;

		@include for-phone-only {
			padding: 20px;
		}

		.container {
			display: flex;
			align-items: center;
			height: 44px;
		}

		.desktop {
			display: block;
			@include for-phone-only {
				display: none;
			}
		}

		.mobile {
			display: none;
			@include for-phone-only {
				display: block;
			}
		}

		.logo {
			flex: 1;
		}
	}
</style>
