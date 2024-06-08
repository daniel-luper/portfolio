<script lang="ts">
	import ContentSection from '$lib/components/organisms/ContentSection.svelte';
	import TimelineItem from '$lib/components/molecules/TimelineItem.svelte';
	import Timeline from '$lib/components/organisms/Timeline.svelte';

	let data = [
		{
			title: 'Created Portfolio Website',
			year: 2024,
			content: 'Made this cool portfolio website!'
		},
		{
			title: 'Graduated Baylor 🐻',
			content:
				"Sic'Em Bears! Earned a BS in Computer Science with a Math Minor. Completed the BIC honors program. Summa Cum Laude."
		},
		{
			title: 'S-STEM Scholars Conference',
			year: 2023,
			content:
				"Neat conference in D.C. They pushed a lot for going to grad school. I'm thinking about it..."
		},
		{
			title: 'Amazon',
			content: 'Software Development Engineer Internship for Amazon.com warranties in Denver.'
		},
		{
			title: 'Spring & Travel',
			year: 2022,
			content: '2nd internship working remotely for Spring + traveled a bit in the U.S. and abroad.'
		},
		{
			title: 'Spring & Robotics Research',
			year: 2021,
			content: 'First summer internship at Spring. Also did robotics research at Baylor.'
		},
		{
			title: 'Started at Baylor',
			year: 2020,
			content: 'Started my undergrad at Baylor University in Waco, TX.'
		},
		{
			title: 'First "Hello World"',
			year: 2015,
			content: 'First time I wrote a "Hello World" program.'
		}
	];
</script>

<div class="container">
	<ContentSection title="Timeline">
		<Timeline>
			{#each data as item}
				<TimelineItem title={item.title} year={item.year}>
					<p>{item.content}</p>
				</TimelineItem>
			{/each}
		</Timeline>
	</ContentSection>
</div>
