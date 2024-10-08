<script>
	import '../app.scss'
	import { onMount } from 'svelte'
	import { fly } from 'svelte/transition'
	import { fade } from 'svelte/transition'
	import { page } from '$app/stores'
	import { goto } from '$app/navigation'
	import { parseData } from '$lib/js/parseData.js'
	import { session } from '$lib/stores/session.js'
	import { settings } from '$lib/stores/settings.js'
	import { oldAssignments } from '$lib/stores/oldAssignments.js'
	import Spinner from '$lib/components/Spinner.svelte'
	

	export let data

	let spinning = false

	onMount(async () => {
		if ($settings.theme === 'dark') {
			$settings.theme = 'night'
		}
		if (data.user) {
			console.log('load')
			await load()
		}
	})

	async function load() {
		const res = await fetch('/data')
		if (!res.ok) {
			console.log('fetch data not ok: ', res.status)
			goto('/login')
			return
		}
		const json = await res.json()
		console.log(json)
		let { student, periods, currentPeriod } = json
		$session = {
			student,
			periods,
			currentPeriod,
			selectedPeriod: currentPeriod,
			selected: periods[currentPeriod],
			gradebook: periods[currentPeriod]
		}
		parseData($session, $oldAssignments)
		console.log($session)
		$oldAssignments = $oldAssignments
	}

	$: if ($session.periods) {
		$session.selected = $session.periods[$session.selectedPeriod]
	}

	async function refresh() {
		spinning = true
		await load()
		spinning = false
	}
</script>

<svelte:head>
	<meta name="color-scheme" content={$settings.theme} />
	<link rel="stylesheet" href={`/themes/${$settings.theme}.css`} />
</svelte:head>

{#if data.user}
	{#if $session.gradebook && $session.student}
		<nav
			in:fade={{ duration: 200, delay: 200 }}
			out:fade={{ duration: 200 }}
			data-sveltekit-prefetch
		>
			<img alt="profile" src={'data:image/jpeg;base64,' + $session.student.Photo} />
			<a class:active={$page.url.pathname === '/'} href="/">
				<i class="bi bi-house" />
			</a>
			<a class:active={$page.url.pathname === '/grades'} href="/grades">
				<i class="bi bi-list-ol" />
			</a>
			<a class:active={$page.url.pathname === '/assignments'} href="/assignments">
				<i class="bi bi-pen" />
			</a>
			<a class:active={$page.url.pathname === '/assignments'} href="/assignments">
				Attendence
			</a>
			<button class={'refresh' + (spinning ? ' spinning' : '')} on:click={refresh}>
				<i class="bi bi-arrow-repeat" />
			</button>
			<a class:active={$page.url.pathname === '/settings'} href="/settings">
				<i class="bi bi-gear" />
			</a>
		</nav>
		<main in:fade={{ duration: 200, delay: 200 }} out:fade={{ duration: 200 }}>
			{#key data.url}
				<div
					class="transition-container"
					in:fly={{ y: -5, duration: 200, delay: 200 }}
					out:fly|local={{ y: 5, duration: 200 }}
				>
					<slot />
				</div>
			{/key}
		</main>
	{:else}
		<div class="loading-container" out:fade={{ duration: 200 }}>
			<div class="loading">
				<Spinner />
				<div class="spinner-label">Fetching student info...</div>
			</div>
		</div>
	{/if}
{:else}
	<div
		class="login-container"
		in:fade={{ duration: 200, delay: 200 }}
		out:fade={{ duration: 200 }}
	>
		<slot />
	</div>
{/if}

<style lang="scss">
	nav {
		@include box;
		display: flex;
		flex-direction: column;
		align-items: center;
		width: min-content;
		padding: $spacing-small;
	}

	main {
		width: 100%;
		height: 100%;
		margin-left: $spacing;
	}

	.transition-container {
		width: 100%;
		height: 100%;
	}

	.loading-container {
		position: absolute;
		top: 0;
		left: 0;
		display: flex;
		width: 100%;
		height: 100%;
	}

	.loading {
		width: max-content;
		margin: auto;
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.spinner-label {
		margin-top: $spacing-small;
	}

	img {
		width: 50px;
		height: 50px;
		object-fit: cover;
		object-position: 0 0;
		border-radius: 50px;
	}

	a:first-of-type {
		margin-top: 50px;
	}

	a:last-of-type {
		margin-bottom: 12.5px;
	}

	.refresh {
		margin-top: auto;
		&.spinning i {
			animation: spin 1s ease-in-out infinite;
		}
	}

	a,
	button {
		display: flex;
		align-items: center;
		justify-content: center;
		text-decoration: none;
		height: 50px;
		width: 50px;
		margin-top: 30px;
		text-align: center;
		border-radius: 50%;
		color: var(--font-color);
		background: transparent;
		&:hover {
			background: var(--bg-color-1-5);
		}
		&:active {
			background: var(--bg-color-1);
			i {
				transform: scale(0.9);
			}
		}
		&.active {
			background: var(--bg-color-1);
		}
	}

	i {
		display: inline-block;
		height: 25px;
		font-size: 25px;
		line-height: 25px;
		color: var(--accent-color);
	}

	@keyframes spin {
		from {
			transform: rotate(0deg);
		}
		to {
			transform: rotate(360deg);
		}
	}

	@media (max-width: $breakpoint-phone) {
		nav {
			flex-direction: row;
			justify-content: space-between;
			width: 100%;
		}

		a {
			margin: 0 !important;
		}

		img {
			display: none;
		}

		main {
			margin-left: 0;
			margin-bottom: $spacing;
			height: calc(100% - 2 * $spacing - 50px);
		}
	}
</style>
