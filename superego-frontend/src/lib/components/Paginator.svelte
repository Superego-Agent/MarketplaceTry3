<script lang="ts">
import ChevronLeftIcon from '~icons/fluent/chevron-left-24-regular';
import ChevronRightIcon from '~icons/fluent/chevron-right-24-regular';

// --- Props ---

interface Props {
	/** The full list of items to paginate */
	items?: any[];
	/** The width of the container where items are displayed */
	containerWidth?: number;
	/** The minimum desired width for each item */
	minItemWidth?: number;
	children?: import('svelte').Snippet<[any]>;
}

let {
	items = [],
	containerWidth = 0,
	minItemWidth = 400,
	children
}: Props = $props();

// --- Internal State ---
let currentPage = $state(0);
let itemsPerPage = $state(1);
let totalPages = $state(1);
let paginatedItems: any[] = $state([]);

// --- Reactive Calculations ---
$effect(() => {
	if (items.length === 0 || containerWidth <= 0 || minItemWidth <= 0) {
		itemsPerPage = 1;
		totalPages = 1;
		currentPage = 0;
		paginatedItems = [];
		return;
	}

	// 1. Calculate max items per page based on width
	const maxItemsPerPage = Math.max(1, Math.floor(containerWidth / minItemWidth));

	// 2. Calculate initial total pages based on max items
	let calculatedTotalPages = Math.ceil(items.length / maxItemsPerPage);

	// 3. If more than one page, recalculate itemsPerPage for better balance
	let balancedItemsPerPage = maxItemsPerPage;
	if (calculatedTotalPages > 1) {
		balancedItemsPerPage = Math.ceil(items.length / calculatedTotalPages);
	}

	// 4. Final calculation based on balanced itemsPerPage
	itemsPerPage = balancedItemsPerPage;
	totalPages = Math.ceil(items.length / itemsPerPage);

	// 5. Clamp currentPage to valid range
	currentPage = Math.max(0, Math.min(currentPage, totalPages - 1));

	// 6. Slice items for the current page
	const startIndex = currentPage * itemsPerPage;
	const endIndex = startIndex + itemsPerPage;
	paginatedItems = items.slice(startIndex, endIndex);
	// console.log(`Paginator: w=${containerWidth}, minW=${minItemWidth}, maxIPP=${maxItemsPerPage}, calcTP=${calculatedTotalPages}, balIPP=${balancedItemsPerPage}, finalIPP=${itemsPerPage}, finalTP=${totalPages}, currentP=${currentPage}, items=${items.length}, paginated=${paginatedItems.length}`);
});

// --- Functions ---
function goToPage(pageIndex: number) {
	currentPage = Math.max(0, Math.min(pageIndex, totalPages - 1));
}
function nextPage() {
	if (currentPage < totalPages - 1) {
		currentPage++;
	}
}
function prevPage() {
	if (currentPage > 0) {
		currentPage--;
	}
}
</script>

{#if items.length > 0}
	<div class="paginator-wrapper">
		<!-- Default Slot: Exposes paginatedItems -->
		{@render children?.({ paginatedItems, })}

		{#if totalPages > 1}
			<div class="pagination-controls">
				<button
					class="pagination-button"
					onclick={prevPage}
					disabled={currentPage === 0}
					aria-label="Previous Page"
				>
					<ChevronLeftIcon />
				</button>
				<div class="pagination-dots">
					{#each { length: totalPages } as _, i}
						<button
							class="dot"
							class:active={i === currentPage}
							onclick={() => goToPage(i)}
							aria-label="Go to page {i + 1}"
							aria-current={i === currentPage ? 'page' : undefined}
						></button>
					{/each}
				</div>
				<button
					class="pagination-button"
					onclick={nextPage}
					disabled={currentPage === totalPages - 1}
					aria-label="Next Page"
				>
					<ChevronRightIcon />
				</button>
			</div>
		{/if}
	</div>
{/if}

<style lang="scss">
	.paginator-wrapper {
		display: contents; // Allows slot content to flow naturally in parent flex/grid
	}

	.pagination-controls {
		display: flex;
		justify-content: center;
		align-items: center;
		padding: var(--space-xs) 0 var(--space-sm);
		flex-shrink: 0;
		gap: var(--space-md);
		margin-top: var(--space-sm); // Add some space above controls
	}

	.pagination-button {
		background: none;
		border: none;
		color: var(--text-secondary);
		cursor: pointer;
		padding: var(--space-xs);
		border-radius: var(--radius-sm);
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 1.2em;

		&:hover:not(:disabled) {
			background-color: var(--bg-hover);
			color: var(--text-primary);
		}

		&:disabled {
			color: var(--text-disabled);
			cursor: not-allowed;
		}
	}

	.pagination-dots {
		display: flex;
		gap: var(--space-xs);
	}

	.dot {
		width: 10px;
		height: 10px;
		border-radius: 50%;
		background-color: var(--secondary);
		border: none;
		padding: 0;
		cursor: pointer;
		transition: background-color 0.2s ease;

		&:hover {
			background-color: var(--text-secondary);
		}

		&.active {
			background-color: var(--primary);
		}
	}
</style>