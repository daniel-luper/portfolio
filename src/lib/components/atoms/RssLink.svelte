<script lang="ts">
	import RssIcon from '$lib/icons/rss.svelte';
</script>

<a href="/rss.xml" target="_blank" rel="noopener noreferrer" title="Subscribe to my RSS Feed">
	<RssIcon />
</a>

<style lang="scss">
	a {
		transition: all 0.2s ease-in-out;
		width: 24px;
		color: var(--color--text-shade);
		fill: var(--color--text-shade);

		&:hover {
			color: var(--color--text);
			fill: var(--color--text);
			width: 28px;
		}
	}
</style>
