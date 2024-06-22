<script>
	let { event, index, deleteEvent } = $props();

	function handleTimeChange(e) {
		const newTime = e.target.value;
		const [hours, minutes] = newTime.split(':').map(Number);
		event.time.setHours(hours, minutes, 0, 0);
		schedule = schedule.sort((a, b) => a.time - b.time);
	}

	function handleActivityChange(e) {
		event.activity = e.target.value;
	}
</script>

<div class="card">
	<h3><input type="text" bind:value={event.activity} on:input={handleActivityChange} /></h3>
	<p>
		<strong>Time:</strong>
		<input
			type="time"
			value={event.time.toISOString().substring(11, 16)}
			on:input={handleTimeChange}
		/>
	</p>
	<p><strong>Note:</strong> {event.note}</p>
	<button on:click={() => deleteEvent(index)}>X</button>
</div>

<style>
	.card {
		padding: 1rem;
		margin: 0.5rem;
		border: 1px solid #ccc;
		border-radius: 0.5rem;
		background: #f9f9f9;
	}
</style>
