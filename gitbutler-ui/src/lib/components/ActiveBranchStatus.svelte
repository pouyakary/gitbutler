<script lang="ts">
	import Tag from '$lib/components/Tag.svelte';
	import ViewPrContextMenu from '$lib/components/ViewPrContextMenu.svelte';
	import { normalizeBranchName } from '$lib/utils/branch';
	import { openExternalUrl } from '$lib/utils/url';
	import { onDestroy } from 'svelte';
	import type { Persisted } from '$lib/persisted/persisted';
	import type { BaseBranch, Branch } from '$lib/vbranches/types';

	export let base: BaseBranch | undefined | null;
	export let branch: Branch;
	export let prUrl: string | undefined;
	export let isUnapplied = false;
	export let hasIntegratedCommits = false;
	export let isLaneCollapsed: Persisted<boolean>;

	function updateContextMenu(copyablePrUrl: string) {
		if (popupMenu) popupMenu.$destroy();
		return new ViewPrContextMenu({
			target: document.body,
			props: { prUrl: copyablePrUrl }
		});
	}

	$: popupMenu = updateContextMenu(prUrl || '');

	onDestroy(() => {
		if (popupMenu) {
			popupMenu.$destroy();
		}
	});
</script>

{#if !branch.upstream}
	{#if !branch.active}
		<Tag
			icon="virtual-branch-small"
			color="light"
			help="These changes are stashed away from your working directory."
			reversedDirection
			verticalOrientation={$isLaneCollapsed}>unapplied</Tag
		>
	{:else if hasIntegratedCommits}
		<Tag
			icon="pr-small"
			color="success"
			help="These changes have been integrated upstream, update your workspace to make this lane disappear."
			reversedDirection
			verticalOrientation={$isLaneCollapsed}>Integrated</Tag
		>
	{:else}
		<Tag
			icon="virtual-branch-small"
			color="light"
			help="These changes are in your working directory."
			reversedDirection
			verticalOrientation={$isLaneCollapsed}>Virtual</Tag
		>
	{/if}
	{#if !isUnapplied && !$isLaneCollapsed}
		<Tag
			shrinkable
			disabled
			help="Branch name that will be used when pushing. You can change it from the lane menu."
			verticalOrientation={$isLaneCollapsed}
		>
			origin/{branch.upstreamName ? branch.upstreamName : normalizeBranchName(branch.name)}</Tag
		>
	{/if}
{:else}
	<Tag
		color="dark"
		icon="remote-branch-small"
		help="At least some of your changes have been pushed"
		verticalOrientation={$isLaneCollapsed}
		reversedDirection>Remote</Tag
	>
	<Tag
		icon="open-link"
		color="ghost"
		border
		clickable
		shrinkable
		verticalOrientation={$isLaneCollapsed}
		on:click={(e) => {
			const url = base?.branchUrl(branch.upstream?.name);
			if (url) openExternalUrl(url);
			e.preventDefault();
			e.stopPropagation();
		}}
	>
		{$isLaneCollapsed ? 'View branch' : `origin/${branch.upstream?.name}`}
	</Tag>
	{#if prUrl}
		<Tag
			icon="pr-small"
			color="ghost"
			border
			clickable
			verticalOrientation={$isLaneCollapsed}
			on:click={(e) => {
				const url = prUrl;
				if (url) openExternalUrl(url);
				e.preventDefault();
				e.stopPropagation();
			}}
			on:contextmenu={(e) => {
				e.preventDefault();
				popupMenu.openByMouse(e, undefined);
			}}
		>
			View PR
		</Tag>
	{/if}
{/if}
