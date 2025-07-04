<script lang="ts">
	import Input from '$lib/components/ui/input/input.svelte';
	import Button from '$lib/components/ui/button/button.svelte';
	import { fly } from 'svelte/transition';
	import type { ChatMessage } from '$lib/chat';
	import { enhance } from '$app/forms';
	import { GatewaySocket } from '$lib/chat.js';
	import { page } from '$app/state';
	import pfp from '$lib/assets/gh-small.png';
	import anon from './assets/anon.jpg';
	import { cubicOut } from 'svelte/easing';
	let { data } = $props();
	let messages = $state(data.messages);
	let sending = $state(false);

	$effect(() => {
		if (page.params.uuid === undefined) {
			return;
		}
		new GatewaySocket(page.params.uuid, (msgs) => {
			msgs.forEach((msg) => {
				if (!messages.find((m: ChatMessage) => m.id === msg.id)) {
					if (msg.author === null) {
						sending = false;
					}
					messages.unshift(msg);
				}
			});
		});
	});
</script>

<div class="flex h-full w-full flex-col items-center justify-center">
	<div class="grid h-4/5 w-5/6 grid-rows-[1fr_3rem] border bg-neutral-900 p-2">
		<div class="flex h-full min-h-[0] flex-col-reverse overflow-x-hidden overflow-y-scroll">
			{#if messages.length == 0}
				<div class="pb-[1rem] text-center text-gray-400 select-none">
					Begin chatting by typing a message
				</div>
			{/if}
			{#each messages as message (message.id)}
				<div transition:fly|global={{ y: 50, duration: 300, easing: cubicOut }}>
					<div
						class="flex p-4 text-wrap break-all {message.author == null ? 'flex-row-reverse' : ''}"
					>
						<img
							class="h-[2.5rem] w-[2.5rem] rounded-full"
							src={message.author == null ? anon : pfp}
							alt="Profile"
						/>
						<p class="p-2">{message.content}</p>
					</div>
					<hr class="w-[99%] self-center bg-white opacity-15" />
				</div>
			{/each}
		</div>
		<div>
			<form
				class="flex h-full w-full items-center space-x-2"
				data-sveltekit-preload-data="tap"
				method="POST"
				use:enhance={(evt) => {
					// Reset chat bot (Needed for new chat redirects)
					evt.formElement.reset();
					sending = true;

					return async ({ update }) => {
						await update();
						// Re-Focus text field after submission
						document.getElementById('text-input')?.focus();
					};
				}}
			>
				<Input
					name="msg"
					required
					id="text-input"
					class="bg-primary text-primary-foreground h-full"
					autocomplete="off"
					type="text"
					placeholder="Type Here"
				/>
				<Button disabled={sending} variant="secondary" class="h-full" type="submit">Send</Button>
			</form>
		</div>
	</div>
	<div class="pt-4">
		Chat sessions are Anonymous between you and <a href="https://github.com/Enoz"
			><i class="text-amber-600">Enoz</i></a
		>. IPs are logged.
	</div>
</div>
