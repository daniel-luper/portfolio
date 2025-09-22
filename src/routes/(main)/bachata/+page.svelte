<script>
	import { onMount } from 'svelte';
	import { browser } from '$app/environment';
	import * as d3 from 'd3';

	let container;
	let songData = [];
	let artistData = [];
	let currentTransform = d3.zoomIdentity;
	let svg, xScale, yScale, zoom;

	// Constants
	const goldenRatio = 1.618033988749;
	const margin = { top: 20, right: 20, bottom: 60, left: 80 };

	onMount(async () => {
		if (browser) {
			await loadAndProcessData();
			initializeChart();
			setupTooltip();

			// Add resize handler to maintain aspect ratio
			const handleResize = () => {
				if (container && svg) {
					initializeChart();
					setupTooltip();
				}
			};

			window.addEventListener('resize', handleResize);

			return () => {
				window.removeEventListener('resize', handleResize);
			};
		}
	});

	async function loadAndProcessData() {
		try {
			const response = await fetch('/bachata-data/song_coverage_report.json');
			const rawData = await response.json();

			// Process song data
			songData = rawData.map((item, index) => {
				const filename = Object.keys(item)[0];
				const data = item[filename];

				// Extract song title and artists from filename
				const cleanName = filename.replace('.json', '');
				const parts = cleanName.split(' - ');
				let artists = [];
				let songTitle = cleanName;

				if (parts.length >= 2) {
					artists = parts[0].split(',').map((a) => a.trim());
					songTitle = parts.slice(1).join(' - ');
				} else {
					const commaIndex = cleanName.indexOf(',');
					if (commaIndex > 0) {
						artists = [cleanName.substring(0, commaIndex).trim()];
						songTitle = cleanName.substring(commaIndex + 1).trim();
					} else {
						artists = ['Unknown Artist'];
					}
				}

				return {
					id: index,
					filename: filename,
					artists: artists,
					songTitle: songTitle,
					specificCoverage: data.specific_coverage,
					generalCoverage: data.general_coverage,
					lemmaCount: data.song_lemma_count,
					visible: true
				};
			});

			// Process artist data
			const artistMap = new Map();
			songData.forEach((song) => {
				song.artists.forEach((artist) => {
					if (!artistMap.has(artist)) {
						artistMap.set(artist, { name: artist, songs: [], visible: true });
					}
					artistMap.get(artist).songs.push(song);
				});
			});

			artistData = Array.from(artistMap.values()).sort((a, b) => b.songs.length - a.songs.length);

			// Assign colors to artists
			artistData.forEach((artist, index) => {
				const hue = (index * goldenRatio * 360) % 360;
				artist.color = `hsl(${hue}, 70%, 50%)`;
			});

			// Assign colors to songs based on primary artist
			songData.forEach((song) => {
				const primaryArtist = artistData.find(
					(artist) =>
						artist.songs.includes(song) &&
						artist.songs.length >=
							Math.max(
								...song.artists.map(
									(a) => artistData.find((art) => art.name === a)?.songs.length || 0
								)
							)
				);
				song.color = primaryArtist?.color || '#666';
				song.primaryArtist = primaryArtist?.name || song.artists[0];
			});
		} catch (error) {
			console.error('Error loading data:', error);
		}
	}

	function initializeChart() {
		if (!container) return;

		const containerRect = container.getBoundingClientRect();
		const availableWidth = containerRect.width - margin.left - margin.right;
		const availableHeight = containerRect.height - margin.top - margin.bottom;

		// Define aspect ratio (width:height) - good for data visualization
		const aspectRatio = 1.6; // 16:10 ratio, good for scatter plots

		// Calculate maximum allowed height to avoid scrolling - scale up for very wide screens
		const isMobile = window.innerWidth <= 768;
		const baseMaxHeight = isMobile ? 300 : 450;
		// For very wide screens (>1600px), allow much larger charts
		const wideScreenMultiplier =
			window.innerWidth > 1600 ? Math.min(2.2, window.innerWidth / 1600) : 1;
		const maxAllowedHeight = Math.min(baseMaxHeight * wideScreenMultiplier, availableHeight - 50);

		// Calculate dimensions based on aspect ratio and constraints
		let width = availableWidth;
		let height = width / aspectRatio;

		// If calculated height exceeds maximum, constrain by height instead
		if (height > maxAllowedHeight) {
			height = maxAllowedHeight;
			width = height * aspectRatio;
		}

		// Ensure minimum usable size
		if (width < 400) {
			width = 400;
			height = width / aspectRatio;
		}

		// Clear any existing SVG
		d3.select(container).select('svg').remove();

		svg = d3
			.select(container)
			.append('svg')
			.attr('width', width + margin.left + margin.right)
			.attr('height', height + margin.top + margin.bottom);

		const g = svg.append('g').attr('transform', `translate(${margin.left},${margin.top})`);

		// Set up scales
		xScale = d3
			.scaleLinear()
			.domain(d3.extent(songData, (d) => d.generalCoverage))
			.range([0, width]);

		yScale = d3
			.scaleLinear()
			.domain(d3.extent(songData, (d) => d.specificCoverage))
			.range([height, 0]);

		// Add axes
		g.append('g')
			.attr('class', 'x-axis')
			.attr('transform', `translate(0,${height})`)
			.call(d3.axisBottom(xScale));

		g.append('g').attr('class', 'y-axis').call(d3.axisLeft(yScale));

		// Add axis labels
		g.append('text')
			.attr('class', 'x-label')
			.attr('x', width / 2)
			.attr('y', height + 50)
			.style('text-anchor', 'middle')
			.text('General Spanish Word Frequency (%)');

		g.append('text')
			.attr('class', 'y-label')
			.attr('transform', 'rotate(-90)')
			.attr('x', -height / 2)
			.attr('y', -50)
			.style('text-anchor', 'middle')
			.text('Bachata-Relative Word Frequency (%)');

		// Add clipping path to constrain circles within plot area
		svg
			.append('defs')
			.append('clipPath')
			.attr('id', 'chart-clip')
			.append('rect')
			.attr('x', 0)
			.attr('y', 0)
			.attr('width', width)
			.attr('height', height);

		// Set up zoom
		zoom = d3
			.zoom()
			.scaleExtent([0.5, 10])
			.on('zoom', function (event) {
				currentTransform = event.transform;
				updateChart();
			});

		svg.call(zoom);

		// Add circles container with clipping
		const circlesContainer = g.append('g').attr('clip-path', 'url(#chart-clip)');

		// Add circles for data points
		circlesContainer
			.selectAll('.song-circle')
			.data(songData)
			.enter()
			.append('circle')
			.attr('class', 'song-circle')
			.attr('r', 4)
			.attr('fill', (d) => d.color)
			.attr('stroke', '#fff')
			.attr('stroke-width', 1)
			.style('cursor', 'pointer');

		updateChart();
		setupTooltip();
	}

	function updateChart() {
		if (!svg) return;

		const g = svg.select('g');
		const circlesContainer = g.select('g[clip-path]');
		const visibleSongs = songData.filter((song) => {
			return song.artists.some((artistName) => {
				const artist = artistData.find((a) => a.name === artistName);
				return artist && artist.visible;
			});
		});

		// Update axes with transformed scales
		if (currentTransform) {
			const newXScale = currentTransform.rescaleX(xScale);
			const newYScale = currentTransform.rescaleY(yScale);

			g.select('.x-axis').call(d3.axisBottom(newXScale));
			g.select('.y-axis').call(d3.axisLeft(newYScale));
		}

		const circles = circlesContainer.selectAll('.song-circle').data(visibleSongs);

		circles.exit().remove();

		circles
			.enter()
			.append('circle')
			.attr('class', 'song-circle')
			.attr('r', 4)
			.attr('fill', (d) => d.color)
			.attr('stroke', '#fff')
			.attr('stroke-width', 1)
			.style('cursor', 'pointer')
			.merge(circles)
			.attr('cx', (d) =>
				currentTransform
					? currentTransform.applyX(xScale(d.generalCoverage))
					: xScale(d.generalCoverage)
			)
			.attr('cy', (d) =>
				currentTransform
					? currentTransform.applyY(yScale(d.specificCoverage))
					: yScale(d.specificCoverage)
			);
	}

	function setupTooltip() {
		const tooltip = d3
			.select('body')
			.append('div')
			.attr('class', 'tooltip')
			.style('position', 'absolute')
			.style('background', 'rgba(0,0,0,0.8)')
			.style('color', 'white')
			.style('padding', '10px')
			.style('border-radius', '5px')
			.style('pointer-events', 'none')
			.style('opacity', 0);

		svg
			.select('g[clip-path]')
			.selectAll('.song-circle')
			.on('mouseover', function (event, d) {
				tooltip.transition().duration(200).style('opacity', 1);
				tooltip
					.html(
						`
					<strong>${d.songTitle}</strong><br/>
					Artists: ${d.artists.join(', ')}<br/>
					Bachata-relative: ${d.specificCoverage.toFixed(2)}%<br/>
					General Spanish: ${d.generalCoverage.toFixed(2)}%<br/>
					Lemma count: ${d.lemmaCount}
				`
					)
					.style('left', event.pageX + 10 + 'px')
					.style('top', event.pageY - 10 + 'px');
			})
			.on('mouseout', function () {
				tooltip.transition().duration(200).style('opacity', 0);
			});
	}

	function resetZoom() {
		if (svg && zoom) {
			svg.transition().duration(750).call(zoom.transform, d3.zoomIdentity);
		}
	}

	function toggleArtist(artist) {
		artist.visible = !artist.visible;
		artistData = [...artistData]; // Trigger reactivity
		if (svg) {
			updateChart();
		}
	}

	function selectAll() {
		artistData.forEach((artist) => (artist.visible = true));
		artistData = [...artistData];
		if (svg) {
			updateChart();
		}
	}

	function deselectAll() {
		artistData.forEach((artist) => (artist.visible = false));
		artistData = [...artistData];
		if (svg) {
			updateChart();
		}
	}
</script>

<svelte:head>
	<title>Bachata Songs: Word Frequency Analysis</title>
</svelte:head>

<div class="bachata-container">
	<div class="chart-area">
		<h1>Bachata Songs: Word Frequency Analysis</h1>
		<div class="chart-controls">
			<button on:click={resetZoom}>Reset Zoom</button>
		</div>
		<div class="chart-container" bind:this={container} />
	</div>
	<div class="artist-panel">
		<h2>Artists (<span>{artistData.filter((a) => a.visible).length}</span>)</h2>
		<div class="select-all-controls">
			<button on:click={selectAll}>Select All</button>
			<button on:click={deselectAll}>Deselect All</button>
		</div>
		<div class="artist-list">
			{#each artistData as artist (artist.name)}
				<div
					class="artist-item"
					class:visible={artist.visible}
					on:click={() => toggleArtist(artist)}
					on:keydown={(e) => {
						if (e.key === 'Enter' || e.key === ' ') {
							e.preventDefault();
							toggleArtist(artist);
						}
					}}
					role="button"
					tabindex="0"
				>
					<div class="color-indicator" style="background-color: {artist.color}" />
					<span class="artist-name">{artist.name}</span>
					<span class="song-count">({artist.songs.length})</span>
				</div>
			{/each}
		</div>
	</div>
</div>

<style lang="scss">
	.bachata-container {
		display: grid;
		grid-template-columns: 1fr minmax(320px, 400px);
		gap: 20px;
		padding: 20px;
		width: 100%;
		max-width: none; // Override global container max-width
		margin-left: auto;
		margin-right: auto;
		height: calc(100vh - 200px); // Constrain to available space on desktop
		overflow: hidden; // Prevent container from growing beyond viewport on desktop

		// Enhanced responsive grid for wider screens
		@media (min-width: 1200px) {
			padding: 30px;
			gap: 30px;
			grid-template-columns: 1fr minmax(350px, 450px);
		}

		@media (min-width: 1400px) {
			padding: 40px;
			gap: 40px;
			grid-template-columns: 1fr minmax(380px, 500px);
		}

		@media (min-width: 1600px) {
			padding: 50px;
			gap: 50px;
			grid-template-columns: 1fr minmax(400px, 550px);
		}
	}

	.chart-area {
		background: white;
		border-radius: 8px;
		box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
		padding: 20px;
		display: flex;
		flex-direction: column;
		height: 100%; // Take full height of grid cell on desktop
		overflow: hidden; // Prevent overflow
	}

	.chart-area h1 {
		text-align: center;
		margin-bottom: 20px;
		color: var(--color--text);
		font-size: 24px;
	}

	.chart-controls {
		display: flex;
		justify-content: center;
		margin-bottom: 15px;
	}

	.chart-controls button {
		background: var(--color--brand);
		color: white;
		border: none;
		padding: 8px 16px;
		border-radius: 4px;
		cursor: pointer;
		font-size: 14px;
		transition: background-color 0.2s;

		&:hover {
			background: var(--color--brand-dark);
		}
	}

	.chart-container {
		flex: 1;
		min-height: 300px;
		overflow: hidden;
		display: flex;
		justify-content: center;
		align-items: center;

		// Remove fixed max-heights to allow flexible sizing

		// Ensure chart stays within bounds
		:global(svg) {
			max-width: 100%;
			max-height: 100%;
		}
	}

	.artist-panel {
		background: white;
		border-radius: 8px;
		box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
		padding: 20px;
		display: flex;
		flex-direction: column;
		height: 100%; // Take full height of grid cell on desktop
		overflow: hidden; // Prevent overflow
	}

	.artist-panel h2 {
		color: var(--color--text);
		margin-bottom: 15px;
		font-size: 18px;
	}

	.select-all-controls {
		display: flex;
		gap: 10px;
		margin-bottom: 15px;
	}

	.select-all-controls button {
		background: #2563eb;
		color: #ffffff;
		border: 1px solid #1d4ed8;
		padding: 8px 16px;
		border-radius: 6px;
		cursor: pointer;
		font-size: 14px;
		font-weight: 500;
		flex: 1;
		transition: all 0.2s;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);

		&:hover {
			background: #1d4ed8;
			border-color: #1e40af;
			box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
			transform: translateY(-1px);
		}

		&:active {
			transform: translateY(0);
			box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
		}
	}

	.artist-list {
		flex: 1;
		overflow-y: auto;
		border: 1px solid #e0e0e0;
		border-radius: 4px;
		padding: 10px;
		min-height: 0; // Allow flex child to shrink

		// Better scrollbar styling
		&::-webkit-scrollbar {
			width: 8px;
		}

		&::-webkit-scrollbar-track {
			background: #f1f1f1;
			border-radius: 4px;
		}

		&::-webkit-scrollbar-thumb {
			background: #c1c1c1;
			border-radius: 4px;
		}

		&::-webkit-scrollbar-thumb:hover {
			background: #a8a8a8;
		}
	}

	.artist-item {
		display: flex;
		align-items: center;
		padding: 8px;
		margin-bottom: 4px;
		border-radius: 4px;
		cursor: pointer;
		transition: background-color 0.2s;
		opacity: 0.5;

		&.visible {
			opacity: 1;
		}

		&:hover {
			background-color: #f5f5f5;
		}
	}

	.color-indicator {
		width: 12px;
		height: 12px;
		border-radius: 50%;
		margin-right: 8px;
		flex-shrink: 0;
	}

	.artist-name {
		flex: 1;
		font-size: 14px;
		color: var(--color--text);
	}

	.song-count {
		font-size: 12px;
		color: var(--color--text-shade);
		margin-left: 4px;
	}

	@media (max-width: 768px) {
		.bachata-container {
			grid-template-columns: 1fr;
			grid-template-rows: auto auto;
			height: auto; // Override desktop height constraint
			max-height: none;
			min-height: auto;
			gap: 15px;
			padding: 15px;
			overflow: visible; // Allow content to flow naturally
		}

		.chart-area {
			height: auto; // Override desktop height constraint
			max-height: 450px;
		}

		.chart-container {
			max-height: 350px; // Constrain height for mobile
			min-height: 250px;
		}

		.artist-panel {
			height: auto; // Override desktop height constraint
			max-height: 400px;
			min-height: 300px;
		}

		.artist-list {
			max-height: 250px;
		}
	}

	:global(.tooltip) {
		font-size: 12px;
		line-height: 1.4;
	}

	:global(.song-circle) {
		transition: r 0.2s;
	}

	:global(.song-circle:hover) {
		r: 6;
	}

	:global(.x-axis, .y-axis) {
		font-size: 12px;
	}

	:global(.x-label, .y-label) {
		font-size: 14px;
		fill: var(--color--text);
	}
</style>
