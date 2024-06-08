<ol class="timeline">
	<li class="group">
		<div class="year" />
		<div class="vertical-dotted-line" />
	</li>
	<slot />
</ol>

<style lang="scss">
	@import '$lib/scss/breakpoints.scss';

	.timeline {
		display: flex;
		flex-direction: column;
		list-style-type: none;
		width: min(100%, 640px);
		margin: 0;
		margin-top: -20px;
	}

	.group {
		display: flex;
		flex-direction: row;
		height: 36px;
		gap: 10px;
		margin: 0px;
	}

	.year {
		display: flex;
		flex-direction: column;
		min-width: 70px;
		height: 100%;
		align-items: flex-end;
		margin-right: 36px;

		@include for-phone-only {
			margin-right: 10px;
			min-width: 64x;
		}
	}

	.separator {
		margin-top: 6px;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 6px;
	}

	.vertical-dotted-line {
		margin-left: 11.5px;
		border-left: 2px dotted black;
		height: 100%;
		margin-bottom: 0px;
	}
</style>
