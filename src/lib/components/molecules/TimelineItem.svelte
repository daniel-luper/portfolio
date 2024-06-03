<script lang="ts">
	import Circle from '$lib/icons/circle.svelte';

	export let title: string;
	export let year: any = undefined;
</script>

<li class="group">
	<div class="year">
		{#if year}
			<h5>{year}</h5>
		{/if}
	</div>
	<div class="separator">
		<div class="icon">
			<Circle />
		</div>
		<div class="vertical-line" />
	</div>
	<div class="content">
		{#if title}
			<h4>{title}</h4>
		{/if}
		<slot id="description" />
	</div>
</li>

<style>
	div :global(p) {
		margin-top: 6px;
		font-size: 0.85rem;
	}
	.group {
		display: flex;
		flex-direction: row;
		height: 92px;
		gap: 15px;
		margin: 0px;
	}
	.year {
		display: flex;
		flex-direction: column;
		width: 70px;
		min-width: 70px; /* Add this line */
		height: 100%;
		align-items: flex-end;
		margin-right: 36px;
	}
	h5 {
		font-weight: 400;
		text-align: right;
		color: var(--color--text-inverse-shade);
	}
	.separator {
		margin-top: 6px;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 6px;
	}
	.icon {
		width: 14px;
	}
	.vertical-line {
		width: 2px;
		height: 100%;
		background-color: black;
		margin-left: 0.5px;
	}
</style>
