<script lang="ts">
	import { cn } from '$lib/utils';
	import Button from '$lib/components/ui/button/button.svelte';
	import { onMount, setContext } from 'svelte';
	import MaterialSymbolsDeleteOutline from '~icons/material-symbols/delete-outline';
	import MaterialSymbolsEditSquareOutlineRounded from '~icons/material-symbols/edit-square-outline-rounded';
	import { clientNames, companyNames, generateData, updateTotals } from '$lib/generateData';
	import { columns } from './columns.svelte';
	import { TzezarDatagrid } from '$lib/datagrid/tzezar-datagrid.svelte';
	import * as Datagrid from '$lib/datagrid';
	import { toast } from 'svelte-sonner';
	import { removeRow } from '$lib/datagrid/fns/remove-row';
	import EditForm from '../examples/advanced/_components/edit-form.svelte';
	let intervalId: number | undefined;
	let isRunning = false;

	// Function that runs at an interval
	function doSomethingAtInterval(task: () => void, interval: number) {
		intervalId = setInterval(() => {
			task();
		}, interval);
	}

	// Function to clear the interval
	function stopInterval() {
		if (intervalId) {
			clearInterval(intervalId);
			intervalId = undefined;
		}
	}

	// Example task
	function exampleTask() {
		datagrid.data = updateTotals(datagrid.data);
	}
	let datagrid = setContext(
		`datagrid`,
		new TzezarDatagrid({
			data: generateData(100000),
			columns,
			title: 'The best datagrid ever',
			options: {
				pagination: { display: true },
				rows: { striped: true },
				footer: { display: true },
				topbar: {
					display: true,
					displayCopyDataMenu: true,
					displayExportDataMenu: true,
					displayFullscreenToggle: true,
					displayHeadFilterToggle: true,
					settingsMenu: { display: true }
				}
			}
		})
	);

	import * as ContextMenu from '$lib/components/ui/context-menu';
	import CellWithContextMenu from '../examples/advanced/_components/cell-with-context-menu.svelte';
	import { getNestedValue } from '$lib/datagrid/fns/get-nested-value';


	const toggleData = () => {
		if (datagrid.internal.paginatedData) {
			datagrid.internal.paginatedData = [];
		} else {
			// datagrid.internal.paginatedData = applyInternalLogic(
			// 	datagrid.data,
			// 	datagrid.state.filters,
			// 	datagrid.state.sortingArray,
			// 	datagrid.state.pagination.page,
			// 	datagrid.state.pagination.perPage
			// );
		}
		return datagrid.internal.paginatedData;
	};

	function toggleInterval() {
		if (isRunning) {
			stopInterval();
		} else {
			doSomethingAtInterval(exampleTask, 1000); // Runs every 1000 ms
		}
		isRunning = !isRunning;
	}
</script>

<Datagrid.Datagrid>
	{#snippet topBar()}
		<Datagrid.TopBar />
	{/snippet}
	{#snippet head()}
		{#each datagrid.columns as column, i (column.id)}
			{#if column.id === 'checkbox'}
				<Datagrid.HeaderWithoutSpacing {column} title={column.title}>
					{#snippet custom()}
						<Datagrid.HeaderRowSelectionDropdown />
					{/snippet}
				</Datagrid.HeaderWithoutSpacing>
			{:else if column.id === 'expand'}
				<Datagrid.HeaderWithoutSpacing {column} title="" class={{ container: '' }} />
			{:else}
				<Datagrid.Header {column} class={{ header: '' }}>
					{#snippet filter()}
						<Datagrid.ColumnFilter {column} />
					{/snippet}
				</Datagrid.Header>
			{/if}
		{/each}
	{/snippet}
	{#snippet body()}
		{#each datagrid.internal.paginatedData as row, rowIndex}
			<Datagrid.Row rowId={row.id} {rowIndex}>
				{#each datagrid.columns as column, columnIndex}
					{#if column.id === 'checkbox'}
						<Datagrid.CellWithoutSpacing {row} {column} {columnIndex} {rowIndex}>
							{#snippet custom()}
								<Datagrid.CellRowSelectionCheckbox {row} />
							{/snippet}
						</Datagrid.CellWithoutSpacing>
					{:else if column.id === 'expand'}
						<Datagrid.CellWithoutSpacing {row} {column} {columnIndex} {rowIndex}>
							{#snippet custom()}
								<Datagrid.ExpandRowToggler rowId={row.id} />
							{/snippet}
						</Datagrid.CellWithoutSpacing>
					{:else if column.id === 'actions'}
					<Datagrid.Cell {row} {column} {columnIndex} {rowIndex}>
						{#snippet custom()}
							<div class={cn('flex flex-row gap-2')}>
								<Button
									size="sm"
									variant="destructive"
									onclick={() => {
										removeRow(row.id, datagrid);
										toast.success('Row removed');
									}}
								>
									<MaterialSymbolsDeleteOutline />
								</Button>
								<EditForm />
							</div>
						{/snippet}
					</Datagrid.Cell>
					{:else}
					<ContextMenu.Root>
						<ContextMenu.Trigger asChild let:builder>
							<CellWithContextMenu
								{builder}
								{columnIndex}
								{rowIndex}
								{column}
								{row}
								class={{
									data: cn('overflow-hidden text-nowrap text-ellipsis'),
									cell: cn(
										'overflow-hidden text-nowrap',
										column.id === 'total' && row['total'] < 2000 && 'text-red-400',
										column.id === 'total' && row['total'] > 6000 && 'text-green-400',
										column.id === 'profit' && row['profit'] < 200 && 'text-red-400',
										column.id === 'profit' && row['profit'] > 5000 && 'text-green-400',
										row['status'] === 'canceled' && 'text-primary/40 line-through',
										column.id === 'receiver' &&
											row['receiver'] === 'Sebastian "Tzezar" Drozd' &&
											'font-bold text-orange-500'
									)
								}}
							/>
						</ContextMenu.Trigger>
						<ContextMenu.Content>
							<ContextMenu.Item>{getNestedValue(row, column.id)}</ContextMenu.Item>
						</ContextMenu.Content>
					</ContextMenu.Root>
					
				
					{/if}
				{/each}
			</Datagrid.Row>
			{#if Datagrid.isRowExpanded(datagrid, row.id)}
				<Datagrid.Row
					class={`border-b ${Datagrid.STAY_IN_PLACE} ${Datagrid.HIDE_BEHIND_PARENT_ROW}`}
					{rowIndex}
				>
					<div class="p-2 pl-3">
						Lorem ipsum dolor sit amet consectetur adipisicing elit. Harum deserunt tenetur debitis
						praesentium aliquam error quibusdam explicabo nam voluptates, dignissimos minima quasi
						aliquid. Repellat, voluptatibus. Natus cumque temporibus nostrum quos. Assumenda
						laboriosam nostrum laborum impedit dolorem consectetur praesentium doloribus iusto
						accusamus recusandae! Sint, natus dolorem perferendis nesciunt similique nihil optio
						repellat adipisci ad expedita numquam quaerat incidunt cum consectetur praesentium.
						Pariatur tempore delectus sunt necessitatibus at voluptatum beatae molestias ratione
						modi nostrum a neque dolor illo magnam vero, natus dolorem, corporis eum aspernatur
						quaerat? Quibusdam ab velit neque rerum excepturi.
					</div>
				</Datagrid.Row>
			{/if}
		{/each}
	{/snippet}
	<!-- Custom TopBar via snippet goes here -->
	<!-- Custom Loading Indicator via snippet goes here -->
	<!-- Custom Data Indicator via snippet goes here -->
	<!-- Custom Footer via snippet goes here -->
	<!-- Custom Pagination via snippet goes here -->
</Datagrid.Datagrid>
