<script lang="ts">
	import { onMount } from "svelte";

	import toast, { Toaster } from "svelte-french-toast";

	function log(string: string) {
		toast(string, {
	icon: '📝',
});
	}

	onMount(async () => {
		if (!("NDEFReader" in window)) {
			toast.error("NFC is not availabe on your device/browser");
		}
	});

	const handleClick = async () => {
		// toast.success(window.NDEFMessage.toString());
		try {
			log("NFC Handle Started");
			const ndef = new NDEFReader();

			await ndef.scan();
			log("> Scan started");

			ndef.addEventListener("readingerror", () => {
				log(
					"Argh! Cannot read data from the NFC tag. Try another one?",
				);
			});

		} catch (error) {
			toast.error(""+error);
		}
	};

	const writeTag = async () => {
		if (!("NDEFReader" in window)) {
		}

		const ndef = new NDEFReader();
		try {
			await ndef.write({
				records: [
					{
						recordType: "url",
						data: "google.fr",
					},
				],
			});

			// This workaround prevents the phone from reading the NFC tag immediately after the write operation.
			const abortController = new AbortController();
			await ndef.scan({ signal: abortController.signal });
		} catch (err) {}
	};
</script>

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Svelte demo app" />
</svelte:head>

<section>
	<Toaster />
	<button type="button" on:click={handleClick}>Toast</button>
</section>

<style>
	section {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		flex: 0.6;
	}

	h1 {
		width: 100%;
	}

	.welcome {
		display: block;
		position: relative;
		width: 100%;
		height: 0;
		padding: 0 0 calc(100% * 495 / 2048) 0;
	}

	.welcome img {
		position: absolute;
		width: 100%;
		height: 100%;
		top: 0;
		display: block;
	}
</style>
