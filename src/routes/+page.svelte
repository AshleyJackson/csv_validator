<script lang="ts">
	import CSVFileValidator, { type ValidatorConfig } from 'csv-file-validator';
	import dayjs from 'dayjs';
	import { onMount } from 'svelte';

	type messages_type = {
		status: 'valid' | 'invalid';
		message: string;
	};

	let tab_state: string[] = ['2008', '2018', '2026'];
	let selected_tab: string | null = null;
	let valid_rows: string[] = [];
	let invalid_rows: string[] = [];
	let file: File | null = null;
	let file_name: string | null = null;
	let messages: messages_type[] = [];
	let output: string[] = [];

	onMount(async () => {
		console.log('onMount');
		// output.push('test');
		console.log(output.length);
	});

	function format_date(date: Date): string {
		const formatted_date = dayjs(date).format('Y-m-d');
		// console.log(formatted_date);
		return formatted_date;
	}

	const config_2008: ValidatorConfig = {
		headers: [
			{
				name: 'First Name',
				inputName: 'first_name',
				required: false,
			},
			{
				name: 'Transaction Date',
				inputName: 'transaction_date',
				required: false,
			},
		],
		isHeaderNameOptional: false,
		parserConfig: {
			delimiter: ',',
		},
	};

	const config_2018: ValidatorConfig = {
		headers: [],
		isHeaderNameOptional: true,
	};

	const config_2026: ValidatorConfig = {
		headers: [
			{
				name: 'Transaction Date',
				inputName: 'transaction_date',
				required: true,
				validate: function (value: any): boolean {
					// console.log(value);
					if (value === null || value === undefined) {
						return false;
					}
					if (typeof value === 'string') {
						// console.log(value);
						return format_date(new Date(value)) !== '';
					}
					return false;
				},
				validateError: function (
					headerName: string,
					rowNumber: number,
					columnNumber: number,
				) {
					return `Invalid date format in column ${columnNumber} of row ${rowNumber}`;
				},
			},
		],
		parserConfig: {
			delimiter: ',',
		},
		isHeaderNameOptional: false,
	};

	let selected_config = [config_2008, config_2018, config_2026];

	async function validate_csv(e: any, config: string) {
		let validator_config;
		file = e.target.files[0];
		if (file === null) {
			return;
		}
		if (config === 'config_2008') {
			validator_config = config_2008;
		}
		if (config === 'config_2018') {
			validator_config = config_2018;
		}
		if (config === 'config_2026') {
			validator_config = config_2026;
		}
		if (!validator_config) {
			return;
		}
		const validator = await CSVFileValidator(file, validator_config);
		// console.log(validator);
		if (validator.inValidData.length > 0) {
			for (const row of validator.inValidData) {
				messages.push({
					status: 'invalid',
					message: row.message,
				});
			}
		}
		if (validator.data.length > 0) {
			// console.log(validator.output);
			for (const row of validator.data) {
				messages.push({
					status: 'valid',
					message: row.message,
				});
			}
		}
		// console.log(messages);
		// console.log(typeof messages);
		for (const message of messages) {
			if (message.status === 'invalid') {
				if (message.message.startsWith('Header name')) {
					output.push(message.message);
				}
				// console.log(message.message);
			}
		}
		// console.log(output);
		// const valid_rows = validator.output;
		// const invalid_rows = validator.inValidoutput;
		// if (valid_rows.length === 0) {
		// console.log(valid_rows);
		// }
		// console.log(messages);
		// console.log(messages);
		console.log(output.length);
	}

	async function reset(e: any) {
		console.clear();
		console.log('reset');
		// console.log(valid_messages);
		// console.log(e);
		// const state = e.target.value
		// selected_tab = state;
		// file = null;
	}
</script>

<section>
	<div
		class="w-full flex font-bold flex-col mt-8 text-4xl items-center justify-center"
	>
		<h1>MyAFF CSV File Validator</h1>
	</div>
	<div class="w-full flex mt-8 text-4xl gap-4 items-center justify-center">
		{#each tab_state as state}
			<button
				on:click={(e) => {
					selected_tab = state;
					reset(e);
					selected_config = [config_2008, config_2018, config_2026];
				}}
				class="w-1/4 p-4 border-2 rounded-lg hover:bg-green-700 text-center bg-blue-500 hover:transition-colors {selected_tab ===
				state
					? 'bg-green-700'
					: ''}"
			>
				{state}
			</button>
		{/each}
	</div>
	{#if selected_tab === '2008'}
		<input
			class="mt-4 border p-2 rounded-lg justify-center items-center ml-auto mr-auto flex text-2xl"
			type="file"
			accept=".csv"
			on:input={(e) => validate_csv(e, 'config_2008')}
		/>
	{/if}
	{#if selected_tab === '2018'}
		<input
			class="mt-4 border p-2 rounded-lg justify-center items-center ml-auto mr-auto flex text-2xl"
			type="file"
			accept=".csv"
			on:input={(e) => validate_csv(e, 'config_2018')}
		/>
	{/if}
	{#if selected_tab === '2026'}
		<input
			class="mt-4 border p-2 rounded-lg justify-center items-center ml-auto mr-auto flex text-2xl"
			type="file"
			accept=".csv"
			on:input={(e) => validate_csv(e, 'config_2026')}
		/>
	{/if}

</section>
