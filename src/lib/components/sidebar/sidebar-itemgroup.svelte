<script lang="ts">
	import * as Collapsible from '$lib/components/ui/collapsible/index.js';
	import * as Sidebar from '$lib/components/ui/sidebar/index.js';
	import ChevronRightIcon from '@lucide/svelte/icons/chevron-right';
	import type { Icon as IconType } from '@lucide/svelte';
	import { page } from '$app/state';

	let {
		items,
		label
	}: {
		label: string;
		items: {
			title: string;
			url: string;
			icon?: typeof IconType;
			items?: {
				title: string;
				url: string;
				icon?: typeof IconType;
			}[];
		}[];
	} = $props();

	// Helper function to check if a URL is active
	function isActiveItem(url: string): boolean {
		return page.url.pathname === url || (page.url.pathname.startsWith(url) && url !== '/');
	}

	// Helper function to check if any child items are active
	function hasActiveChild(items?: { url: string }[]): boolean {
		return items?.some((child) => isActiveItem(child.url)) ?? false;
	}

	// Transform items to include isActive state
	const enhancedItems = $derived(
		items.map((item) => {
			const isItemActive = isActiveItem(item.url);
			const hasActiveSubItem = hasActiveChild(item.items);
			const isActive = isItemActive || hasActiveSubItem;

			return {
				...item,
				isActive,
				items: item.items?.map((subItem) => ({
					...subItem,
					isActive: isActiveItem(subItem.url)
				}))
			};
		})
	);
</script>

<Sidebar.Group>
	<Sidebar.GroupLabel>{label}</Sidebar.GroupLabel>
	<Sidebar.Menu>
		{#each enhancedItems as item (item.title)}
			{#if (item.items?.length ?? 0) > 0}
				<Collapsible.Root open={item.isActive} class="group/collapsible">
					{#snippet child({ props })}
						<Sidebar.MenuItem {...props}>
							<Collapsible.Trigger>
								{#snippet child({ props })}
									{@const Icon = item.icon}
									<Sidebar.MenuButton {...props} tooltipContent={item.title} isActive={item.isActive}>
										{#if item.icon}
											<Icon />
										{/if}
										<span>{item.title}</span>
										<ChevronRightIcon class="ml-auto transition-transform duration-200 group-data-[state=open]/collapsible:rotate-90" />
									</Sidebar.MenuButton>
								{/snippet}
							</Collapsible.Trigger>
							<Collapsible.Content>
								<Sidebar.MenuSub>
									{#each item.items ?? [] as subItem (subItem.title)}
										<Sidebar.MenuSubItem>
											<Sidebar.MenuSubButton isActive={subItem.isActive}>
												{#snippet child({ props })}
													{@const SubIcon = subItem.icon}
													<a href={subItem.url} {...props}>
														{#if subItem.icon}
															<SubIcon />
														{/if}
														<span>{subItem.title}</span>
													</a>
												{/snippet}
											</Sidebar.MenuSubButton>
										</Sidebar.MenuSubItem>
									{/each}
								</Sidebar.MenuSub>
							</Collapsible.Content>
						</Sidebar.MenuItem>
					{/snippet}
				</Collapsible.Root>
			{:else}
				<Sidebar.MenuItem>
					<Sidebar.MenuButton isActive={item.isActive} tooltipContent={item.title}>
						{#snippet child({ props })}
							{@const Icon = item.icon}
							<a href={item.url} {...props}>
								{#if item.icon}
									<Icon />
								{/if}
								<span>{item.title}</span>
							</a>
						{/snippet}
					</Sidebar.MenuButton>
				</Sidebar.MenuItem>
			{/if}
		{/each}
	</Sidebar.Menu>
</Sidebar.Group>
