<script lang="ts">
	import { onMount, afterUpdate } from "svelte"
	import { page } from "$app/stores"
	import { fly } from "svelte/transition"

	let socket: any
	let username: string | null
	let profile_pictures: string | null
	let roles: string | null

	let messages: any[] = []
	let pinned: string = ""
	let connected: string = ""

	// Get message sender profile picture

	async function getProfile(user: string) {
		const res = await fetch("https://kick.com/api/v2/channels/" + user)
		const data = await res.json()

		const { profile_pic } = data.user

		return profile_pic
	}

	function messageFilter(input: any) {
		let message: string = input.content
		let badges: string[] = input.sender.identity.badges

		// Add badges

		let emojiPattern = /\[emote:(\d+):[^\]]+\]/g

		//@ts-ignore
		let messageNew = message.replace(emojiPattern, (match: string, number: string) => {
			if (number) {
				return "<img src='https://files.kick.com/emotes/" + number + "/fullsize' alt='Emoji'>"
			}
		})

		return messageNew
	}

	// Main function with all functionality

	async function startSocket() {
		// Connecting to the websocket server
		socket = new WebSocket(
			"wss://ws-us2.pusher.com/app/eb1d5f283081a78b932c?protocol=7&client=js&version=7.6.0&flash=false"
		)

		// Functionality
		socket.onmessage = async (e: any) => {
			let msg = JSON.parse(e.data)

			let eventType = msg.event

			if (eventType == "pusher_internal:subscription_succeeded") {
				connected = "1"
			}

			if (eventType == "App\\Events\\ChatMessageEvent") {
				let picture: string = await getProfile(JSON.parse(msg.data).sender.slug)

				if (!picture) {
					picture = "/user.png"
				}

				messages.push({
					eventType: eventType,
					message: JSON.parse(msg.data),
					pfp: picture
				})

				messages = messages
			}

			if (eventType == "App\\Events\\MessageDeletedEvent") {
				msg = JSON.parse(msg.data)

				for (let i = 0; i < messages.length; i++) {
					if (messages[i].message.id == msg.message.id) {
						messages.splice(i, 1)
						messages = messages
						break
					}
				}
			}

			if (eventType == "App\\Events\\PinnedMessageCreatedEvent") {
				msg = JSON.parse(msg.data)

				pinned = msg.message.sender.username + ">" + msg.message.content
			}

			if (eventType == "App\\Events\\PinnedMessageDeletedEvent") {
				pinned = ""
			}
		}
	}

	// Connecting to the channel

	async function connect() {
		messages = []
		pinned = ""

		connected = "0"

		const res = await fetch("https://kick.com/api/v2/channels/" + username?.toLowerCase())

		const data = await res.json()

		const { id } = data.chatroom

		socket.send(
			JSON.stringify({
				event: "pusher:subscribe",
				data: {
					auth: "",
					channel: "chatrooms." + id + ".v2"
				}
			})
		)
	}

	// Do stuff on component mount

	onMount(async () => {
		startSocket()

		username = $page.url.searchParams.get("username")
		profile_pictures = $page.url.searchParams.get("pictures")
		roles = $page.url.searchParams.get("roles")

		socket.onopen = () => {
			if (username) {
				connect()
			}
		}
	})

	// Always scroll to bottom when anything happens

	afterUpdate(() => {
		document.documentElement.scrollTop = document.documentElement.scrollHeight
	})
</script>

{#if connected == "0"}
	<h1>Connecting...</h1>
{:else}
	<h1>{pinned}</h1>
	<div class="messages">
		{#each messages as message}
			<div class="message" in:fly={{ y: 20 }}>
				{#if profile_pictures != "0"}
					<img class="avatar" src={message.pfp} alt="Profile" />
				{/if}
				{#if message.message.sender.identity.badges[0] && roles != "0"}
					<span class="badge rainbow">
						[{message.message.sender.identity.badges[0].text}]
					</span>
				{/if}
				<div class="sender">
					<span style:color={message.message.sender.identity.color}>
						{message.message.sender.username}<b>:</b>
					</span>
				</div>
				<div class="msg">
					{@html messageFilter(message.message)}
				</div>
			</div>
		{/each}
	</div>
{/if}
