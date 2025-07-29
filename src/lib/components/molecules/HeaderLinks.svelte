<script>
	import { page } from '$app/stores';
	import RssLink from '$lib/components/atoms/RssLink.svelte';
</script>

<div class="links" on:click on:keydown>
	<button class:active={$page.url.pathname === '/'} class="tab-button">
		<a href="/">Home</a>
	</button>
	<!-- <button class:active={$page.url.pathname.startsWith('/blog')} class="tab-button">
		<a href="/blog">Blog</a>
	</button> -->
	<!-- <button class:active={$page.url.pathname.startsWith('/timeline')} class="tab-button">
		<a href="/timeline">Timeline</a>
	</button> -->
	<RssLink />
</div>

<style lang="scss">
	@import '$lib/scss/breakpoints.scss';

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

		@include for-phone-only {
			flex-direction: column;
			justify-content: flex-start;
			font-size: 1.5rem;
		}
	}
</style>
