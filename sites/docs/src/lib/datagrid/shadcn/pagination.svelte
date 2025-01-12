<script lang="ts">
	import * as Select from '$lib/components/ui/select';
	import { Pagination } from 'bits-ui';
	import { getContext } from 'svelte';
	import MaterialSymbolsChevronLeft from '~icons/material-symbols/chevron-left';
	import MaterialSymbolsChevronRight from '~icons/material-symbols/chevron-right';
	import type { TzezarDatagrid } from '../tzezar-datagrid.svelte';

	const datagrid = getContext<TzezarDatagrid<unknown>>('datagrid');

	let {}: {} = $props();

	const getPageOptions = (count: number, perPage: number) => {
		let res = [];
		for (let i = 1; i <= Math.ceil(count / perPage); i++) {
			res.push({ label: i.toString(), value: i });
		}
		return res;
	};

	let pageOptions = $derived(
		getPageOptions(datagrid.state.pagination.count, datagrid.state.pagination.perPage)
	);
</script>

<div class="flex flex-row flex-wrap justify-center gap-4 pt-2 sm:justify-between">
	<Pagination.Root
		count={datagrid.state.pagination.count}
		perPage={datagrid.state.pagination.perPage}
		page={datagrid.state.pagination.page}
		let:range
		onPageChange={(v) => {
			datagrid.state.pagination.page = v;
			datagrid.onPageChange();
			datagrid.onChange();
		}}
	>
		<div class="flex items-center gap-2">
			<Pagination.PrevButton
				class="hover:bg-dark-10 active:scale-98 h-8 items-center justify-center   rounded-[9px] border bg-transparent  disabled:cursor-not-allowed disabled:text-muted-foreground hover:disabled:bg-transparent"
			>
				<MaterialSymbolsChevronLeft class="mx-2 h-8" />
			</Pagination.PrevButton>
			<div class="flex items-center">
				<Select.Root
					selected={{
						value: datagrid.state.pagination.page,
						label: `Page ${datagrid.state.pagination.page}`
					}}
					onSelectedChange={(selected) => {
						datagrid.state.pagination.page = Number(selected?.value);
						datagrid.onPageChange();
						datagrid.onChange();
					}}
				>
					<Select.Trigger class="h-8">
						<Select.Value />
					</Select.Trigger>
					<Select.Content sameWidth={false} align="start" class="max-h-96 overflow-auto">
						{#each pageOptions as page (page)}
							<Select.Item value={page.value}>Page {page.label}</Select.Item>
						{/each}
					</Select.Content>
				</Select.Root>
			</div>
			<Pagination.NextButton
				class="hover:bg-dark-10 active:scale-98 h-8 items-center justify-center   rounded-[9px] border bg-transparent  disabled:cursor-not-allowed disabled:text-muted-foreground hover:disabled:bg-transparent"
			>
				<MaterialSymbolsChevronRight class="mx-2 h-8" />
			</Pagination.NextButton>
		</div>
		<p class="pt-1 text-center text-xs text-muted-foreground">
			{range.start} : {range.end} / {datagrid.state.pagination.count}
		</p>
	</Pagination.Root>
	<div class="flex flex-row gap-1 text-nowrap text-xs">
		<Select.Root
			onSelectedChange={(selected) => {
				datagrid.state.pagination.page = 1;
				datagrid.state.pagination.perPage = Number(selected?.value);
				datagrid.onPerPageChange();
				datagrid.onChange();
			}}
			selected={{
				value: datagrid.state.pagination.perPage,
				label: `Per page: ${datagrid.state.pagination.perPage}`
			}}
		>
			<Select.Trigger class="h-8">
				<Select.Value asChild>{datagrid.state.pagination.perPage}</Select.Value>
			</Select.Trigger>
			<Select.Content sameWidth={false} align="start" class="max-h-96 overflow-auto">
				<Select.Label>Rows per page</Select.Label>
				<Select.Item checkbox value={1}>1</Select.Item>
				<Select.Item checkbox value={2}>2</Select.Item>
				<Select.Item checkbox value={10}>10</Select.Item>
				<Select.Item checkbox value={20}>20</Select.Item>
				<Select.Item checkbox value={30}>30</Select.Item>
				<Select.Item checkbox value={40}>40</Select.Item>
				<Select.Item checkbox value={50}>50</Select.Item>
				<Select.Item checkbox value={100}>100</Select.Item>
				<Select.Item checkbox value={250}>250</Select.Item>
			</Select.Content>
		</Select.Root>
	</div>
</div>
