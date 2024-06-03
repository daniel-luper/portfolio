<script lang="ts">
	import { page } from '$app/stores';

	import Logo from '$lib/components/atoms/Logo.svelte';
	import RssLink from '$lib/components/atoms/RssLink.svelte';

	export let showBackground = false;
</script>

<header class:has-background={showBackground}>
	<nav class="container">
		<a class="logo" href="/" aria-label="Site logo">
			<Logo />
		</a>
		<div class="links">
			<button class:active={$page.url.pathname === '/'} class="tab-button">
				<a href="/">Home</a>
			</button>
			<button class:active={$page.url.pathname.startsWith('/blog')} class="tab-button">
				<a href="/blog">Blog</a>
			</button>
			<button class:active={$page.url.pathname.startsWith('/timeline')} class="tab-button">
				<a href="/timeline">Timeline</a>
			</button>
			<RssLink />
		</div>
	</nav>
</header>

<style lang="scss">
	@import '$lib/scss/breakpoints.scss';

	header {
		position: relative;
		padding: 30px 0;

		@include for-phone-only {
			padding: 20px 0;
		}

		.container {
			display: flex;
			align-items: center;
			height: 44px;

			@include for-phone-only {
				.links {
					a {
						display: none;
					}
				}
			}
		}

		.logo {
			flex: 1;
		}

		.links {
			display: flex;
			align-items: center;
			justify-content: flex-end;
			gap: 30px;

			a {
				text-decoration: none;

				color: var(--color--text-shade);
				font-weight: 500;

				display: inline-block;
				transition: transform 0.3s;

				&:hover {
					transform: scale(1.1);
				}
			}

			.tab-button {
				&.active {
					a {
						color: var(--color--text);
					}
				}
				border: none; // Add this line to remove the border
				background-color: transparent; // Add this line to remove the background color
				flex: 0 0 auto; // Add this line to prevent the buttons from growing
			}
		}
	}
</style>
