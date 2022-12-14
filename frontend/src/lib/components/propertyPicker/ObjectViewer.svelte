<script lang="ts">
	import { truncate } from '$lib/utils'

	import { createEventDispatcher } from 'svelte'
	import { getTypeAsString } from '../flows/utils'
	import { computeKey } from './utils'
	import WarningMessage from './WarningMessage.svelte'

	export let json: Object
	export let level = 0
	export let isLast = true
	export let currentPath: string = ''
	export let pureViewer = false
	export let collapsed = level == 3 || Array.isArray(json)

	const collapsedSymbol = '...'
	let keys: string | any[]
	let isArray: boolean
	let openBracket: string
	let closeBracket: string

	$: {
		keys = getTypeAsString(json) === 'object' ? Object.keys(json) : []
		isArray = Array.isArray(json)
		openBracket = isArray ? '[' : '{'
		closeBracket = isArray ? ']' : '}'
	}

	function collapse() {
		collapsed = !collapsed
	}

	const dispatch = createEventDispatcher()

	function selectProp(key: string) {
		dispatch('select', computeKey(key, isArray, currentPath))
	}
</script>

{#if keys.length > 0}
	<span class:hidden={collapsed}>
		{#if level != 0}<span class="cursor-pointer hover:bg-slate-200 px-1 rounded" on:click={collapse}
				>(-)</span
			>{/if}
		<ul class="w-full">
			{#each keys as key, index}
				<li class="pt-1">
					<button on:click={() => selectProp(key)} class="key rounded px-1 hover:bg-sky-100"
						>{!isArray ? key : index}:</button
					>

					{#if getTypeAsString(json[key]) === 'object'}
						<svelte:self
							json={json[key]}
							level={level + 1}
							isLast={index === keys.length - 1}
							currentPath={computeKey(key, isArray, currentPath)}
							{pureViewer}
							on:select
						/>
					{:else}
						<button
							class="val rounded px-1 hover:bg-sky-100 {getTypeAsString(json[key])}"
							on:click={() => selectProp(key)}
						>
							{#if json[key] === undefined}
								<WarningMessage />
							{:else}
								<span>{truncate(JSON.stringify(json[key]), 40)}</span>
							{/if}
						</button>
					{/if}
				</li>
			{/each}
		</ul>
	</span>
	<span class="cursor-pointer hover:bg-slate-200" class:hidden={!collapsed} on:click={collapse}>
		{openBracket}{collapsedSymbol}{closeBracket}
	</span>
	{#if !isLast && collapsed}
		<span class="text-black">,</span>
	{/if}
{:else}
	<span class="text-black">{openBracket}{closeBracket}</span>
{/if}

<style>
	ul {
		list-style: none;
		padding-left: 1rem;
		border-left: 1px dotted lightgray;
		@apply text-black;
	}

	.val {
		@apply text-black;
	}
	.val.undefined {
		@apply text-red-500;
	}
	.val.null {
		@apply text-red-500;
	}
	.val.string {
		@apply text-lime-600;
	}
	.val.number {
		@apply text-orange-600;
	}
	.val.boolean {
		@apply text-cyan-600;
	}
</style>
