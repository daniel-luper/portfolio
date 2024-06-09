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
			title: 'NLP Course',
			content:
				'One of my fav Baylor courses - we traced the evolution of NLP by reading major papers from BLEU (2002) to BERT (2018) and beyond.'
		},
		{
			title: 'S-STEM Scholars Conference',
			year: 2023,
			content:
				"Neat conference in D.C. for a scholarship program I was in. They pushed a lot for going to grad school. I'm thinking about it..."
		},
		{
			title: 'Amazon',
			content:
				'Software Development internship for Amazon.com warranties in Denver. Seeing Big Tech firsthand was super insightful.'
		},
		{
			title: 'Spring & Travel ✈️',
			year: 2022,
			content: '2nd internship working remotely for Spring + traveled a bit in the U.S. and abroad.'
		},
		{
			title: 'Spring & Robotics Research',
			year: 2021,
			content:
				'First summer internship at Spring. Also did robotics research at Baylor. Started realizing how little I know 🥴'
		},
		{
			title: 'Started at Baylor',
			year: 2020,
			content: 'Started my undergrad at Baylor University in Waco, TX.'
		},
		{
			title: 'Self-Driving Picar',
			year: 2019,
			content:
				'First AI project: built a miniature self-driving car. Classified images as left/right/straight using a basic dense neural net.'
		},
		{
			title: 'Graphing Calculator Game',
			year: 2016,
			content: "Made a game in CASIO BASIC using way too many 'if' statements. 🥲"
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
