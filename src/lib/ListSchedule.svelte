<script>
	import { onMount } from 'svelte';
	import EventCard from './EventCard.svelte';
	import winkNLP from 'wink-nlp';
	import model from 'wink-eng-lite-web-model';
	const nlp = winkNLP(model);
	const its = nlp.its;

	let schedule = $state([]);
	let noteInput = $state('');
	let scheduleTitle = $state('Event Production Schedule');
	let dayStartTime = $state('08:00');
	let dayEndTime = $state('18:00');
	let totalHours = $derived(calculateTotalHours(dayStartTime, dayEndTime));

	const inputRef = null;

	function processInput(input, date) {
		const doc = nlp.readDoc(input);
		let dateTime = null;
		let activity = input;

		doc.entities().filter((e) => {
			if (e.out(its.type) === 'TIME') {
				dateTime = convertToDateTime(e.out(), date);
				activity = activity.replace(e.out(), '').trim();
			}
		});

		return { time: dateTime, activity, note: '' };
	}

	function convertToDateTime(timeString, date) {
		timeString = timeString.replace(/[^a-zA-Z0-9:]/g, '');
		const is24HourFormat = /^\d{1,2}:\d{2}$/.test(timeString);

		let hours;
		let minutes = 0;

		if (is24HourFormat) {
			[hours, minutes] = timeString.split(':').map(Number);
		} else {
			const match = timeString.match(/^(\d{1,2})(?::(\d{2}))?([AaPp][Mm])?$/);
			if (!match) {
				throw new Error('Invalid time format');
			}
			hours = Number(match[1]);
			minutes = match[2] ? Number(match[2]) : 0;
			const modifier = match[3] ? match[3].toUpperCase() : null;

			if (modifier === 'PM' && hours < 12) hours += 12;
			if (modifier === 'AM' && hours === 12) hours = 0;
		}

		const newDate = new Date(date);
		newDate.setHours(hours, minutes, 0, 0);
		return newDate;
	}

	function handleEnter(event) {
		if (event.key === 'Enter') {
			const { time, activity } = processInput(noteInput, new Date());
			if (time && activity) {
				schedule.push({ time, activity, note: '' });
				schedule = schedule.sort((a, b) => a.time - b.time);
				noteInput = '';
			}
		}
	}

	function calculateTotalHours(start, end) {
		const [startHour, startMinute] = start.split(':').map(Number);
		const [endHour, endMinute] = end.split(':').map(Number);
		const startDate = new Date();
		const endDate = new Date();

		startDate.setHours(startHour, startMinute, 0, 0);
		endDate.setHours(endHour, endMinute, 0, 0);

		const diff = endDate - startDate;
		return diff / (1000 * 60 * 60);
	}

	onMount(() => {
		inputRef.focus();
	});

	function deleteEvent(index) {
		schedule.splice(index, 1);
	}
</script>

<div class="container">
	<div class="controls">
		<label for="schedule-title">Schedule Title</label>
		<input type="text" id="schedule-title" bind:value={scheduleTitle} />

		<div class="flex-row">
			<div>
				<label for="day-start-time">Day Start Time</label>
				<input type="time" id="day-start-time" bind:value={dayStartTime} />
			</div>
			<div>
				<label for="day-end-time">Day End Time</label>
				<input type="time" id="day-end-time" bind:value={dayEndTime} />
			</div>
		</div>

		<p>Total Hours: {totalHours}</p>
	</div>

	<input
		type="text"
		class="input-large"
		bind:value={noteInput}
		on:keydown={handleEnter}
		placeholder="Enter a time and activity..."
		this={inputRef}
	/>
	<button
		on:click={() => {
			navigator.clipboard.writeText(JSON.stringify(schedule));
		}}>Copy Schedule</button
	>

	{#each schedule as event, index}
		<EventCard {event} {index} {deleteEvent} />
	{/each}
</div>

<style>
	.container {
		max-width: 600px;
		margin: 0 auto;
		padding: 2rem;
	}

	.controls {
		display: flex;
		flex-direction: column;
		gap: 1rem;
		margin-bottom: 1rem;
	}

	.controls input,
	.controls label {
		display: block;
		width: 100%;
		margin-bottom: 0.5rem;
	}

	input,
	button {
		width: 100%;
		padding: 0.5rem;
		margin-bottom: 1rem;
		border: 1px solid #ccc;
		border-radius: 0.5rem;
	}

	button {
		background-color: #007bff;
		color: white;
		border: none;
		cursor: pointer;
	}

	button:hover {
		background-color: #0056b3;
	}

	.input-large {
		font-size: 1.25rem;
		font-weight: bold;
	}

	.flex-row {
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		gap: 1rem;
	}
</style>
