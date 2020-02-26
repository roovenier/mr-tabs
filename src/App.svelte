<script>
	let tabs = [];
	let search_query = '';
	let current_window_id = 0;

	chrome.windows.getCurrent({}, (data) => {
		current_window_id = data.id;
		get_tabs();
	});

	const get_tabs = () => {
		chrome.tabs.query({}, (data) => {
			tabs = data
					.filter((tab) => tab.windowId === current_window_id)
					.map((tab, index) => ({...tab, position: index}));
		});
	}

	const clear_search = () => {
		search_query = '';
	}

	const select_tab = (tab) => {
		chrome.tabs.highlight({'tabs': tab.position});
	}

	const remove_tab = (id) => {
		chrome.tabs.remove(id, () => {
			setTimeout(get_tabs, 50);
		});
	}

	$: filtered_tabs = tabs.filter((tab) => tab.title.toLowerCase().indexOf(search_query.toLowerCase().trim()) !== -1);

	chrome.tabs.onUpdated.addListener(get_tabs);
</script>

<main>
	<div class="search">
		<input class="search__input" type="text" bind:value="{search_query}" placeholder="Search by title...">
	</div>

	<ul class="tabs">
		{#each filtered_tabs as tab, i}
			<li on:click={() => select_tab(tab)} class='tabs__item {tab.highlighted ? "active": ""}'>
				<div class="tabs__favicon-wrap">
					{#if tab.favIconUrl}
						<img class="tabs__favicon" src={tab.favIconUrl} alt="">
					{/if}
				</div>
				<p class="tabs__info">
					<span class="tab__title">{tab.title}</span>
					<span class="tab__url">{tab.url}</span>
				</p>
				<button class="tab__remove" title="Remove tab" on:click|stopPropagation={() => remove_tab(tab.id)}>
					<img class="tab__remove-icon" src="assets/delete.png" alt="">
				</button>
			</li>
		{/each}
	</ul>

	{#if filtered_tabs.length === 0}
		<p class="not-found">No tabs found</p>
	{/if}
</main>