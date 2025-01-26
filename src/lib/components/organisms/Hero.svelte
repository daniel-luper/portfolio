<script>
	import GitHubIcon from '$lib/icons/socials/github.svelte';
	import LinkedInIcon from '$lib/icons/socials/linkedin.svelte';
	import Email from '$lib/icons/socials/email.svelte';
	import ResumeIcon from '$lib/icons/socials/resume.svelte';
	import ButtonIcon from '$lib/components/atoms/Button.svelte';
</script>

<section id="hero">
	<h1 class="hello">Hi, I'm Daniel Luper 👋</h1>
	<p class="intro">
		a <b>software engineer</b> aiming to build <b>socially beneficial</b> tech sustainably.
		Outside of work, you might catch me playing board games, enjoying nature, or
		dancing to a banger🕺
	</p>
	<div class="ctas">
		<ButtonIcon href="https://github.com/daniel-luper" target="_blank" rel="noopener noreferrer">
			<GitHubIcon slot="icon" />
			GitHub
		</ButtonIcon>
		<ButtonIcon
			href="https://www.linkedin.com/in/daniel-luper/"
			target="_blank"
			rel="noopener noreferrer"
		>
			<LinkedInIcon slot="icon" />
			LinkedIn
		</ButtonIcon>
		<ButtonIcon href="mailto:jdaniel.luper@gmail.com" target="_blank" rel="noopener noreferrer">
			<Email slot="icon" />
			Email
		</ButtonIcon>
		<ButtonIcon href="pdfs/resume.pdf" target="_blank">
			<ResumeIcon slot="icon" />
			Resume
		</ButtonIcon>
	</div>
</section>

<style lang="scss">
	@import '$lib/scss/breakpoints.scss';

	#hero {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		gap: 15px;
		position: relative;
		padding: 80px 0;

		@include for-phone-only {
			padding: 40px 0 50px;
		}

		.hello {
			text-align: center;
			font-size: 2.5rem;
		}

		.intro {
			width: min(100%, 740px);
			padding: 10px;
			font-size: 1.2rem;

			@include for-phone-only {
				// 	display: none;
				font-size: 1rem;
			}

			b {
				color: #2d7749;
				font-weight: 800;
			}
		}

		.ctas {
			display: flex;
			flex-wrap: wrap;
			align-items: center;
			justify-content: center;
			gap: 10px;
			width: 100%;
		}
	}
</style>
