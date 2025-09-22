<script>
	import GithubIcon from '$lib/icons/socials/github.svelte';
</script>

<footer>
	<div class="content">
		<div>&copy; 2025 Daniel Luper. All rights reserved.</div>
	</div>
	<div class="content">
		<div>
			Source code on <a
				href="https://github.com/daniel-luper/portfolio"
				target="_blank"
				rel="noopener noreferrer">Github</a
			>.
			<!-- Art by -->
			<!-- <a href="https://itsjools.com" target="_blank" rel="noopener noreferrer">Julie Hines Pool</a>. -->
		</div>
	</div>
</footer>

<style lang="scss">
	footer {
		display: flex;
		justify-content: space-between;
		position: relative;
		border-top: 1px solid #e6e6e6;

		.content {
			display: flex;
			flex-direction: row;
			align-items: center;
			gap: 15px;
			font-weight: 600;
			text-align: center;
			font-weight: 400;
			font-size: 90%;
			color: var(--color--text-shade);
			padding: 20px;
		}

		a {
			&:hover {
				font-size: 1.05em;
			}
		}
	}
</style>
