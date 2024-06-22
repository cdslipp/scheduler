<script>
	let { event, index, deleteEvent, schedule = $bindable() } = $props();

	function handleTimeChange(e) {
		const newTime = e.target.value;
		const [hours, minutes] = newTime.split(':').map(Number);
		const newDate = new Date(event.time);
		newDate.setHours(hours, minutes, 0, 0);
		event.time = newDate;
		schedule = schedule.sort((a, b) => a.time - b.time);
	}

	function handleActivityChange(e) {
		event.activity = e.target.value;
	}
</script>

<div class="card">
	<input
		type="text"
		class="activity-input"
		bind:value={event.activity}
		on:input={handleActivityChange}
	/>
	<div class="time-container">
		<input
			type="time"
			class="time-input"
			value={event.time.toISOString().substring(11, 16)}
			on:input={handleTimeChange}
		/>
	</div>
	{#if event.note}
		<p class="note"><strong>Note:</strong> {event.note}</p>
	{/if}
	<button class="delete-button" on:click={() => deleteEvent(index)}>×</button>
</div>

<style>
	.card {
		padding: 1rem;
		margin: 0.5rem 0;
		border: 1px solid #e0e0e0;
		border-radius: 0.5rem;
		background: #ffffff;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
		position: relative;
		transition: box-shadow 0.3s ease;
	}

	.card:hover {
		box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
	}

	.activity-input {
		width: calc(100% - 80px);
		font-size: 1rem;
		padding: 0.5rem;
		border: 1px solid #e0e0e0;
		border-radius: 0.25rem;
		margin-bottom: 0.5rem;
	}

	.time-container {
		display: inline-block;
		margin-right: 0.5rem;
	}

	.time-input {
		padding: 0.25rem;
		border: 1px solid #e0e0e0;
		border-radius: 0.25rem;
	}

	.note {
		margin: 0.5rem 0 0 0;
		font-size: 0.9rem;
		color: #666;
	}

	.delete-button {
		position: absolute;
		top: 0.5rem;
		right: 0.5rem;
		background: none;
		border: none;
		font-size: 1.5rem;
		color: #999;
		cursor: pointer;
		transition: color 0.3s ease;
	}

	.delete-button:hover {
		color: #f44336;
	}
</style>
