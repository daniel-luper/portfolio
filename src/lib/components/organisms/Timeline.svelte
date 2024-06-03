<ol class="timeline">
	<li class="group">
		<div class="year" />
		<div class="vertical-dotted-line" />
	</li>
	<slot />
</ol>

<style>
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
		width: 70px;
		min-width: 70px; /* Add this line */
		height: 100%;
		align-items: flex-end;
		margin-right: 36px;
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
