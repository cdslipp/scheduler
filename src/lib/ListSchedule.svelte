<script>
	import EventCard from './EventCard.svelte';
	import winkNLP from 'wink-nlp';
	import model from 'wink-eng-lite-web-model';
	const nlp = winkNLP(model);
	const its = nlp.its;

	let schedule = $state([]);
	let noteInput = $state('');
	let scheduleTitle = $state('Event Production Schedule');
	let selectedDate = $state(new Date().toISOString().substring(0, 10));
	let dayStartTime = $state('08:00');
	let dayEndTime = $state('18:00');

	let totalHours = $derived(calculateTotalHours(dayStartTime, dayEndTime, selectedDate));
	let isNextDay = $derived(checkIfNextDay(dayStartTime, dayEndTime));

	let inputRef;

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
			const { time, activity } = processInput(noteInput, new Date(selectedDate));
			if (time && activity) {
				schedule = [...schedule, { time, activity, note: '' }].sort((a, b) => a.time - b.time);
				noteInput = '';
			}
		}
	}

	function calculateTotalHours(start, end, date) {
		const [startHour, startMinute] = start.split(':').map(Number);
		const [endHour, endMinute] = end.split(':').map(Number);
		const startDate = new Date(date);
		const endDate = new Date(date);

		startDate.setHours(startHour, startMinute, 0, 0);
		endDate.setHours(endHour, endMinute, 0, 0);

		if (endDate <= startDate) {
			endDate.setDate(endDate.getDate() + 1);
		}

		const diffMs = endDate - startDate;
		const diffMins = Math.floor(diffMs / (1000 * 60));
		const hours = Math.floor(diffMins / 60);
		const minutes = diffMins % 60;

		return `${hours} hours and ${minutes} minutes`;
	}

	function checkIfNextDay(start, end) {
		const [startHour, startMinute] = start.split(':').map(Number);
		const [endHour, endMinute] = end.split(':').map(Number);
		return endHour < startHour || (endHour === startHour && endMinute < startMinute);
	}

	function deleteEvent(index) {
		schedule.splice(index, 1);
	}
</script>

<div class="container">
	<div class="content">
		<input
			type="text"
			class="input-large"
			bind:value={noteInput}
			on:keydown={handleEnter}
			placeholder="Enter a time and activity..."
			bind:this={inputRef}
		/>

		<div class="controls">
			<label for="schedule-title">Schedule Title</label>
			<input type="text" id="schedule-title" bind:value={scheduleTitle} />

			<div class="date-pickers">
				<label for="selected-date">Select Date</label>
				<input type="date" id="selected-date" bind:value={selectedDate} />

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
			</div>

			<p>Total Hours: {totalHours}</p>
			{#if isNextDay}
				<p class="next-day-warning">End time is on the next day!</p>
			{/if}
		</div>

		{#each schedule as event, index}
			<EventCard {event} {index} {deleteEvent} bind:schedule />
		{/each}
		<button
			on:click={() => {
				navigator.clipboard.writeText(JSON.stringify(schedule));
			}}>Copy Schedule</button
		>
	</div>
</div>

<style>
	:global(body) {
		background-color: #4a148c;
		margin: 0;
		padding: 0;
		font-family: Arial, sans-serif;
	}

	.container {
		max-width: 800px;
		margin: 2rem auto;
		padding: 2rem;
	}

	.content {
		background-color: white;
		border-radius: 1rem;
		padding: 2rem;
		box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
	}

	.controls {
		display: flex;
		flex-direction: column;
		gap: 1rem;
		margin-top: 2rem;
	}

	.controls input,
	.controls label {
		display: block;
		width: 100%;
		margin-bottom: 0.5rem;
	}

	.date-pickers input {
		border: 1px solid #ccc;
		border-radius: 0.5rem;
		padding: 0.5rem;
	}

	input,
	button {
		width: 100%;
		padding: 0.5rem;
		margin-bottom: 1rem;
		border: 1px solid #ccc;
		border-radius: 0.5rem;
	}

	.input-large {
		font-size: 1.25rem;
		font-weight: bold;
		padding: 1rem;
		border-radius: 2rem;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
		transition: box-shadow 0.3s ease;
		margin-bottom: 2rem;
	}

	.input-large:focus {
		outline: none;
		box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
	}

	button {
		background-color: #9c27b0;
		color: white;
		border: none;
		cursor: pointer;
		transition: background-color 0.3s ease;
	}

	button:hover {
		background-color: #7b1fa2;
	}

	.flex-row {
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		gap: 1rem;
	}

	.next-day-warning {
		color: #f44336;
		font-weight: bold;
	}
</style>
