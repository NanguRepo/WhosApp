<script lang="ts">
	import axios from 'axios';
	$: inputText = '';
	$: username = '111';
	$: enableSend = connectionStatus == 'connected' && inputText;
	$: connectionStatus = '';
	$: connectionName = '';
	let checkIndex: number = 0;
	const matchmake = async () => {
		const response = await axios.post('https://api.nangu.dev/v2/whosapp/matchmaking', {
			user: username
		});
		connectionStatus = response.data.status;
		if (connectionStatus == 'waiting') {
			checkConnection();
		}
		if (connectionStatus == 'connected') {
			connectionName = response.data.connectionName;
			getMessages();
		}
	};
	const checkConnection = async () => {
		const response = await axios.post('https://api.nangu.dev/v2/whosapp/checkConnection', {
			user: username
		});
		if (response.data.status == 'connected') {
			connectionStatus = 'connected';
			connectionName = response.data.id;
			getMessages();
		} else {
			checkIndex++;
			if (checkIndex < 15) {
				setTimeout(async () => {
					checkConnection();
				}, 3000);
			} else {
				connectionStatus = 'waitingTimedOut';
				const response = await axios.post('https://api.nangu.dev/v2/whosapp/removeAvailability', {
					user: username
				});
			}
		}
	};
	const sendMessage = async () => {
		await axios.post('https://api.nangu.dev/v2/whosapp/send', {
			id: connectionName,
			user: username,
			message: inputText
		});
		inputText = '';
	};
	$: chatLog = [];
	$: prevChatLog = [];
	const getMessages = async () => {
		const response = await axios.post('https://api.nangu.dev/v2/whosapp/messages', {
			id: connectionName
		});
		if (response.data != prevChatLog) {
			prevChatLog = chatLog;
			chatLog = response.data;
		}
		setTimeout(async () => {
			getMessages();
		}, 1000);
	};
</script>

<body class="dark:bg-neutral-800 dark:text-white">
	<div class="flex flex-col gap-2 p-2 h-screen">
		<div class="flex flex-col h-full rounded gap-2 overflow-scroll">
			{#if connectionStatus == 'connected'}
				{#each chatLog as message}
					<div class="w-full flex flex-row {message.sender == username ? 'justify-end' : ''}">
						<div class="bg-neutral-700 px-2 py-2 rounded w-fit max-w-[55ch]">
							<p class="font-bold">{message.sender}</p>
							<p class="break-words">{message.message}</p>
						</div>
					</div>
				{/each}
			{:else}
				<div class="flex flex-col w-full h-full items-center justify-center">
					<p class="dark:text-neutral-300">enter username</p>
					<form>
						<input bind:value={username} class="w-[25ch]" />
						<button on:click={matchmake} disabled={username.length < 3}>go</button>
					</form>
					{#if connectionStatus == 'waiting'}
						<p class="dark:text-neutral-500">waiting for end user... ({checkIndex})</p>
					{:else if connectionStatus == 'waitingTimedOut'}
						<p class="error-text">waiting timed out.</p>
					{:else if connectionStatus == 'usernameTaken'}
						<p class="error-text">username taken.</p>
					{:else}
						<div class="h-[2.3ch]" />
					{/if}
				</div>
			{/if}
		</div>
		<form class="flex flex-row h-auto p-2 rounded gap-2">
			<input class="w-full" placeholder="Chat" bind:value={inputText} autofocus />
			<button disabled={!enableSend} on:click={sendMessage}>
				<svg
					xmlns="http://www.w3.org/2000/svg"
					viewBox="0 0 24 24"
					fill="currentColor"
					class="w-6 h-6"
				>
					<path
						d="M3.478 2.405a.75.75 0 00-.926.94l2.432 7.905H13.5a.75.75 0 010 1.5H4.984l-2.432 7.905a.75.75 0 00.926.94 60.519 60.519 0 0018.445-8.986.75.75 0 000-1.218A60.517 60.517 0 003.478 2.405z"
					/>
				</svg>
			</button>
		</form>
	</div>
</body>
