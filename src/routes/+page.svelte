<script lang="ts">
	import ChevronDoubleDown from "$lib/components/icons/ChevronDoubleDown.svelte";
	import ChevronDoubleUp from "$lib/components/icons/ChevronDoubleUp.svelte";
	import ArrowPathRoundedSquare from "$lib/components/icons/ArrowPathRoundedSquare.svelte";
	import Cog6Tooth from "$lib/components/icons/Cog6Tooth.svelte";
	import { ProgressRadial, type ToastSettings } from "@skeletonlabs/skeleton";
	import { localStorageStore } from "@skeletonlabs/skeleton";
	import { getToastStore } from "@skeletonlabs/skeleton";
	import { get, type Writable } from "svelte/store";
	import { onDestroy, onMount } from "svelte";

	const toastStore = getToastStore();

	const resetToast: ToastSettings = {
		message: "Are you sure you want to reset?",
		background: "bg-error-500",
		classes: "text-white-900",
		hoverable: true,
		action: {
			label: "Yes",
			response: () => {
				currentCalories.set(0);
				addedCalories = 0;
				lastReset.set(new Date());
			},
		},
	};

	const currentCalories: Writable<number> = localStorageStore(
		"currentCalories",
		0,
	);
	const maxCalories: Writable<number> = localStorageStore(
		"maxCalories",
		2000,
	);
	const lastReset: Writable<Date> = localStorageStore(
		"lastReset",
		new Date(),
	);

	let addedCalories = 0;

	let lastResetDate = new Date();
	$: timeSinceLastReset = new Date().getTime() - lastResetDate.getTime();
	$: hoursSinceLastReset = Math.floor(timeSinceLastReset / 1000 / 60 / 60);
	$: minutesSinceLastReset = Math.floor(
		(timeSinceLastReset / 1000 / 60 / 60 - hoursSinceLastReset) * 60,
	);
	$: secondsSinceLastReset = Math.floor(
		((timeSinceLastReset / 1000 / 60 / 60 - hoursSinceLastReset) * 60 -
			minutesSinceLastReset) *
			60,
	);

	let updateDateInterval: NodeJS.Timeout;
	onMount(() => {
		if (localStorage.getItem("lastReset") == null) {
			lastReset.set(new Date());
		}

		updateDateInterval = setInterval(() => {
			lastResetDate = new Date(get(lastReset));
		}, 100);
	});
	onDestroy(() => {
		clearInterval(updateDateInterval);
	});
</script>

<div class="flex flex-col items-center justify-center gap-4">
	<ProgressRadial
		value={($currentCalories / $maxCalories) * 100}
		width="w-72"
		stroke={30}
	>
		{$currentCalories}
		/
		{$maxCalories}
	</ProgressRadial>
	<span class="text-tertiary-200">
		{addedCalories != 0 ? addedCalories : ""}
	</span>
</div>

<div class="flex flex-col gap-4">
	<div class="flex flex-row gap-4">
		<button
			type="button"
			class="btn variant-filled-secondary w-full"
			on:click={() => {
				currentCalories.update((n) => n - 50);
				addedCalories -= 50;
			}}
			disabled={$currentCalories < 50}
		>
			<span><ChevronDoubleDown /></span>
			<span>50</span>
		</button>
		<button
			type="button"
			class="btn variant-filled-primary w-full"
			on:click={() => {
				currentCalories.update((n) => n + 50);
				addedCalories += 50;
			}}
			disabled={$currentCalories + 50 > $maxCalories}
		>
			<span><ChevronDoubleUp /></span>
			<span>50</span>
		</button>
	</div>
	<div class="flex flex-col">
		<button
			type="button"
			class="btn variant-filled-error"
			on:click={() => {
				toastStore.trigger(resetToast);
			}}
		>
			<span><ArrowPathRoundedSquare /></span>
			<span>Reset</span>
		</button>
		<span class="text-error-300 text-center">
			{hoursSinceLastReset}h {minutesSinceLastReset}m{" "}
			{secondsSinceLastReset}s
		</span>
	</div>
</div>