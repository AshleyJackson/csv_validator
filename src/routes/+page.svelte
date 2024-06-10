<script lang="ts">
	import type { ValidatorConfig } from 'csv-file-validator';
	import CSVFileValidator from 'csv-file-validator';
	import dayjs from 'dayjs';

	let tab_state: string[] = ['2008', '2018', '2026'];
	let selected_tab: string | null = null;
	let valid_rows: string[] = [];
	let invalid_rows: string[] = [];
	let file: File | null = null;

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
				required: true,
			},
		],
		isHeaderNameOptional: false,
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
					console.log(value);
					if (value === null || value === undefined) {
						return false;
					}
					if (typeof value === 'string') {
						console.log(value);
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

	async function validate_csv(file: File, config: ValidatorConfig) {
		const validator = await CSVFileValidator(file, config);
		const valid_rows = validator.data;
		const invalid_rows = validator.inValidData;

		for (const row of valid_rows) {
			console.log(row);
		}

		function reset() {
			// console.log('reset');
			// file = null;
			// validation_messages = [];
		}
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
				on:click={() => ((selected_tab = state), reset())}
				class="w-1/4 p-4 border-2 rounded-lg hover:bg-green-700 text-center bg-blue-500 hover:transition-colors {selected_tab ===
				state
					? 'bg-green-700'
					: ''}"
			>
				{state}
			</button>
		{/each}
	</div>
	<!-- {#if selected_tab} -->
	<input
		class="mt-4 border p-2 rounded-lg justify-center items-center ml-auto mr-auto flex text-2xl"
		type="file"
		on:change={(e: any) => validate_csv(file=e.target.files[0], config_2026)}
	/>
	<!-- {#if validation_messages.length > 0 && !selected_tab} -->
	<div class="mt-4 text-red-500">
		<!-- {#each validation_messages as message}
			{message}
		{/each} -->
	</div>
	<!-- {/if} -->
	<!-- {/if} -->
</section>
