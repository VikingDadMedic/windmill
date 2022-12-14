<script lang="ts">
	import { goto } from '$app/navigation'
	import { page } from '$app/stores'
	import { UserService, WorkspaceService } from '$lib/gen'
	import { logoutWithRedirect } from '$lib/logout'
	import { superadmin, userStore, usersWorkspaceStore, workspaceStore } from '$lib/stores'
	import { getUserExt, refreshSuperadmin } from '$lib/user'
	import { sendUserToast } from '$lib/utils'
	import { SvelteToast } from '@zerodevx/svelte-toast'
	import { onMount } from 'svelte'
	import github from 'svelte-highlight/styles/github'

	// Default toast options
	const toastOptions = {
		duration: 4000, // duration of progress bar tween to the `next` value
		initial: 1, // initial progress bar value
		next: 0, // next progress value
		pausable: false, // pause progress bar tween on mouse hover
		dismissable: true, // allow dismiss with close button
		reversed: false, // insert new toast to bottom of stack
		intro: { x: 256 }, // toast intro fly animation settings
		theme: {} // css var overrides
	}

	const monacoEditorUnhandledErrors = [
		'Model not found',
		'Connection is disposed.',
		'Connection got disposed.',
		'Stopping the server timed out',
		'Canceled'
	]

	async function loadUser() {
		try {
			$usersWorkspaceStore = await WorkspaceService.listUserWorkspaces()
			await refreshSuperadmin()

			if ($workspaceStore) {
				if ($userStore) {
					console.log(`Welcome back ${$userStore.username} to ${$workspaceStore}`)
				} else if ($superadmin) {
					console.log(
						`You are a superadmin, you can go wherever you please, even at ${$workspaceStore}`
					)
				} else {
					$userStore = await getUserExt($workspaceStore)
					if (!userStore) {
						throw Error('Not logged in')
					}
				}
			} else {
				if (!$page.url.pathname.startsWith('/user/')) {
					goto(`/user/workspaces?rd=${encodeURIComponent($page.url.pathname + $page.url.search)}`)
				}
				let user = await UserService.globalWhoami()
				console.log(`Welcome back ${user.email}`)
			}
		} catch (e) {
			console.error(e)
			logoutWithRedirect($page.url.pathname + $page.url.search)
		}
	}

	onMount(() => {
		loadUser()

		window.onunhandledrejection = (event: PromiseRejectionEvent) => {
			event.preventDefault()

			if (event.reason?.message) {
				const { message, body, status } = event.reason

				// Unhandled errors from Monaco Editor don't logout the user
				if (monacoEditorUnhandledErrors.includes(message)) {
					return
				}
				if (message == 'Client not running') {
					sendUserToast(
						'Unrecoverable error for the smart assistant. Refresh the page to get the full experience again (This issue is WIP and will get fixed)',
						true
					)
					return
				}

				if (status == '401') {
					const pathName = $page.url.pathname
					if (pathName != '/user/login') {
						logoutWithRedirect(pathName + $page.url.search)
					}
				} else {
					if (body) {
						sendUserToast(`${body}`, true)
					} else {
						sendUserToast(`${message}`, true)
					}
				}
			} else {
				console.log('Caught unhandled promise rejection without message', event)
			}
		}
	})
</script>

<svelte:head>
	{@html github}
</svelte:head>

<slot />
<SvelteToast options={toastOptions} />

<style>
	:root {
		--toastBackground: #eff6ff;
		--toastBarBackground: #eff6ff;
		--toastColor: #123456;
	}
</style>
