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
	
	// Function to calculate required margins based on actual text measurements
	const getMargin = () => {
		const isMobile = window.innerWidth <= 768;
		
		// Use more generous fixed margins for desktop to avoid clipping
		if (!isMobile) {
			return { top: 30, right: 30, bottom: 80, left: 100 };
		}
		
		return { 
			top: 2,
			right: 2,
			bottom: 20,
			left: 30
		};
	};

	onMount(async () => {
		if (browser) {
			await loadAndProcessData();
			initializeChart();
			setupTooltip();

			// Add resize handler to maintain aspect ratio and height matching
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

		const margin = getMargin();
		const containerRect = container.getBoundingClientRect();
		const availableWidth = containerRect.width - margin.left - margin.right;
		const availableHeight = containerRect.height - margin.top - margin.bottom;

		// Define aspect ratio (width:height) - good for data visualization
		const aspectRatio = 1.6; // 16:10 ratio, good for scatter plots

		// Simple approach: scale chart height based on screen height
		const isMobile = window.innerWidth <= 768;
		const isVeryNarrow = window.innerWidth <= 400;
		
		let maxAllowedHeight;
		
		if (isMobile) {
			// For mobile, calculate chart height to leave space for explanation
			// Use a simpler approach: take most of available height, reserve fixed space for explanation
			const explanationSpace = 180; // Fixed space for explanation
			const minChartHeight = 200; // Minimum chart height
			const maxChartHeight = 400; // Maximum chart height
			
			// Calculate available height for chart
			const chartHeight = Math.max(minChartHeight, availableHeight - explanationSpace);
			maxAllowedHeight = Math.min(maxChartHeight, chartHeight);
		} else {
			// For desktop, scale chart height based on viewport height with better scaling
			const viewportHeight = window.innerHeight;
			const viewportWidth = window.innerWidth;
			
			if (viewportHeight >= 900) {
				// Large screens: scale with screen size, allow generous heights
				const baseHeight = 500;
				const widthBonus = Math.min(200, (viewportWidth - 1200) * 0.1); // Extra height for wide screens
				const heightBonus = Math.min(300, (viewportHeight - 900) * 0.3); // Extra height for tall screens
				maxAllowedHeight = baseHeight + widthBonus + heightBonus;
			} else if (viewportHeight >= 700) {
				// Medium screens: proportional scaling
				maxAllowedHeight = Math.max(350, viewportHeight * 0.4);
			} else {
				// Small screens: conservative height
				maxAllowedHeight = Math.max(280, viewportHeight * 0.35);
			}
		}

		// Calculate dimensions based on aspect ratio and constraints
		let width = availableWidth;
		let height = width / aspectRatio;

		// If calculated height exceeds maximum, constrain by height instead
		if (height > maxAllowedHeight) {
			height = maxAllowedHeight;
			width = height * aspectRatio;
		}

		// Clear any existing SVG
		d3.select(container).select('svg').remove();
		
		// After chart is rendered, match artist panel height to chart area height
		setTimeout(() => {
			const chartArea = container.closest('.chart-area');
			const artistPanel = container.closest('.bachata-container')?.querySelector('.artist-panel');
			
			if (chartArea && artistPanel && !isMobile) {
				const chartAreaHeight = chartArea.offsetHeight;
				artistPanel.style.height = `${chartAreaHeight}px`;
			}
		}, 100); // Small delay to ensure chart is fully rendered

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

		// Add axes with mobile-optimized tick formatting
		const xAxis = d3.axisBottom(xScale);
		const yAxis = d3.axisLeft(yScale);
		
		if (isMobile) {
			// Reduce tick count on mobile for better readability
			xAxis.ticks(4).tickFormat(d3.format('.1f'));
			yAxis.ticks(5).tickFormat(d3.format('.1f'));
		}
		
		g.append('g')
			.attr('class', 'x-axis')
			.attr('transform', `translate(0,${height})`)
			.call(xAxis);

		g.append('g').attr('class', 'y-axis').call(yAxis);

		// Add axis labels (only on desktop, mobile labels will be in explanation)
		if (!isMobile) {
			g.append('text')
				.attr('class', 'x-label')
				.attr('x', width / 2)
				.attr('y', height + 55)
				.style('text-anchor', 'middle')
				.style('font-size', '14px')
				.text('General Spanish Word Frequency (%)');

			g.append('text')
				.attr('class', 'y-label')
				.attr('transform', 'rotate(-90)')
				.attr('x', -height / 2)
				.attr('y', -60)
				.style('text-anchor', 'middle')
				.style('font-size', '14px')
				.text('Bachata-Relative Word Frequency (%)');
		}

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

		// Set up zoom with mobile-optimized settings
		zoom = d3
			.zoom()
			.scaleExtent([0.5, isMobile ? 5 : 10])
			.on('zoom', function (event) {
				currentTransform = event.transform;
				updateChart();
			});
		
		// Configure touch behavior for mobile
		if (isMobile) {
			zoom.filter(function(event) {
				// Allow zoom with pinch/wheel but prevent conflicts with scrolling
				return event.type !== 'mousedown' || event.button === 0;
			});
		}

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

		// Update axes with transformed scales with mobile-optimized tick formatting
		if (currentTransform) {
			const newXScale = currentTransform.rescaleX(xScale);
			const newYScale = currentTransform.rescaleY(yScale);
			
			const isMobile = window.innerWidth <= 768;
			const xAxis = d3.axisBottom(newXScale);
			const yAxis = d3.axisLeft(newYScale);
			
			if (isMobile) {
				// Reduce tick count on mobile and ensure they're visible
				xAxis.ticks(4).tickFormat(d3.format('.1f'));
				yAxis.ticks(5).tickFormat(d3.format('.1f'));
			}

			g.select('.x-axis')
				.transition()
				.duration(50)
				.call(xAxis);
			g.select('.y-axis')
				.transition()
				.duration(50)
				.call(yAxis);
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
		
		<div class="chart-container" bind:this={container} />
		
		<div class="chart-explanation">
			<h3>How to Read This Chart</h3>
			<div class="mobile-axis-labels">
				<div class="axis-label">
					<strong>X-axis (horizontal):</strong> General Spanish Word Frequency (%)
				</div>
				<div class="axis-label">
					<strong>Y-axis (vertical):</strong> Bachata-Relative Word Frequency (%)
				</div>
			</div>
			<p>This graph shows average word frequencies using <strong>"lemmas"</strong> - the base form of words (e.g., "cantó," "canta," and "cantando" all count as the lemma "cantar").</p>
			<p><strong>For language learners:</strong> Start with songs in the <strong>top-right</strong> (easy songs with common words) and work your way toward the <strong>bottom-left</strong> (harder songs with less common vocabulary).</p>
		</div>
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
		grid-template-columns: 1fr minmax(320px, 800px);
		gap: 20px;
		padding: 20px;
		width: 100%;
		max-width: none; // Override global container max-width
		margin-left: auto;
		margin-right: auto;
		align-items: start; // Align to top, let chart area determine height

		// Medium screens: constrain artist panel width
		@media (min-width: 769px) and (max-width: 1199px) {
			grid-template-columns: 1fr minmax(280px, 350px);
		}

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
		// Let content determine natural height
	}

	.chart-area h1 {
		text-align: center;
		color: var(--color--text);
		font-size: 24px;
	}
	.chart-explanation {
		background: #f8f9fa;
		border-radius: 6px;
		padding: 16px;
		margin-top: 10px;
		border-left: 4px solid var(--color--brand);
		width: 100%;
		box-sizing: border-box;
		flex-shrink: 0; // Don't shrink this box
		max-height: 180px; // Increased to accommodate axis labels
		overflow-y: auto; // Allow scrolling if needed

		h3 {
			margin: 0 0 12px 0;
			color: var(--color--text);
			font-size: 16px;
			font-weight: 600;
		}

		.mobile-axis-labels {
			display: none; // Hidden by default
			margin-bottom: 12px;
			
			@media (max-width: 768px) {
				display: block; // Show only on mobile
			}
			
			.axis-label {
				margin-bottom: 6px;
				font-size: 13px;
				color: var(--color--text);
				
				&:last-child {
					margin-bottom: 0;
				}
				
				strong {
					color: var(--color--brand);
				}
			}
		}

		p {
			margin: 0 0 8px 0;
			color: var(--color--text);
			font-size: 14px;
			line-height: 1.5;

			&:last-child {
				margin-bottom: 0;
			}
		}
	}

	.chart-container {
		min-height: 300px;
		overflow: visible; // Allow labels to show outside bounds
		display: flex;
		justify-content: center;
		align-items: center;
		width: 100%;
		flex: 1;

		// Ensure chart stays within bounds but allow overflow for labels
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
		overflow: hidden; // Enable internal scrolling
		// Height will be set dynamically by JavaScript to match chart area
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

		.chart-explanation {
			padding: 12px;
			margin-top: 15px;
			margin-bottom: 20px; // Add bottom margin to prevent clipping
			max-height: none; // Remove height constraint on mobile
			flex-shrink: 0;

			h3 {
				font-size: 15px;
				margin-bottom: 8px; // Reduced margin
			}

			p {
				font-size: 13px;
				line-height: 1.4; // Tighter line height for mobile
				margin-bottom: 6px; // Reduced margin between paragraphs
			}
		}

		.chart-area {
			height: auto; // Override desktop height constraint
			max-height: none; // Remove height constraint to let content determine height
			overflow: visible; // Allow overflow on mobile
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

	@media (max-width: 400px) {
		.bachata-container {
			padding: 10px; // Reduced padding for very narrow screens
			gap: 10px;
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

	:global(.x-axis),
	:global(.y-axis) {
		font-size: 12px;
		
		@media (max-width: 768px) {
			font-size: 11px;
		}
	}
	
	:global(.x-axis path),
	:global(.x-axis line),
	:global(.y-axis path),
	:global(.y-axis line) {
		stroke: var(--color--text);
		stroke-width: 1;
	}
	
	:global(.x-axis text),
	:global(.y-axis text) {
		fill: var(--color--text);
		stroke: none;
	}

	:global(.x-label, .y-label) {
		font-size: 14px;
		fill: var(--color--text);
		font-weight: 500;
		
		@media (max-width: 768px) {
			font-size: 11px;
			font-weight: 600;
			// Add text shadow for better readability on mobile
			filter: drop-shadow(0 0 2px rgba(255, 255, 255, 0.8));
		}
		
		@media (max-width: 400px) {
			font-size: 11px;
		}
	}
</style>
