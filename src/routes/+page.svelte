<script lang="ts">
    import { onMount } from "svelte";

    import toast, { Toaster } from "svelte-french-toast";
    import { writable } from "svelte/store";
    import { greet } from "$lib/greet";
    import { browser } from "$app/environment";

    function log(string: string) {
        toast(string, {
            icon: "📝",
            style: "background-color: #31363f;color: #bbc2cf;",
        });
    }

    function errlog(string: string) {
        toast.error(string, {
            style: "background-color: #31363f;color: #bbc2cf;",
        });
    }

    let hasNFCSupport = false;
    var DEBUG = 1;

    onMount(async () => {
        hasNFCSupport = "NDEFReader" in window;

        if (!hasNFCSupport) {
            errlog(
                "NFC is not availabe on your device/browser. You will not be able to use all the features.",
            );
        }
    });

    const readNDEF = async () => {
        try {
            const ndef = new NDEFReader();
            await ndef.scan();

            log("> Scan started");

            ndef.onreadingerror = (event) => {
                errlog("Error can't read this Tag: " + event);
            };

            ndef.onreading = (event) => {
                log("NDEF message read: " + event.message.records);
                log("There is "+event.message.records.length+" records");
                log("event serail number: "+event.serialNumber);
                log("isTrusted: "+event.isTrusted);
                log("Time Stamp: "+event.timeStamp)
                const decoder = new TextDecoder();
                for (const record of event.message.records) {
                    log("Record type:  " + record.recordType);
                    log("MIME type:    " + record.mediaType);
                    if (record.data) {
                        const arr = new Uint8Array(record.data.buffer);
                        log("=== data ===\n" + arr.join(",") );
                    }
                }
            };
        } catch (error) {
            if (!hasNFCSupport) {
                errlog("NFC is not availabe on your device/browser");
            } else {
                errlog("Error: " + error);
            }
        }
    };

    const writeNDEF = async () => {
        try {
            const ndef = new NDEFReader();
            await ndef.scan();
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
        } catch (error) {
            if (!hasNFCSupport) {
                errlog("NFC is not availabe on your device/browser");
            } else {
                errlog("Error: " + error);
            }
        }
    };

    if (browser) {
        const prefersDarkMode = window.matchMedia(
            "(prefers-color-scheme: dark)",
        );

        const theme = writable(prefersDarkMode.matches ? "dark" : "light");

        prefersDarkMode.addEventListener("change", (e) => {
            theme.set(e.matches ? "dark" : "light");
        });

        theme.subscribe((newTheme) => {
            if (DEBUG >= 2) {
                log("Swapped to theme: " + newTheme);
            }
        });
    }
</script>

<svelte:head>
    <title>NFC App</title>
    <meta name="description" content="NFC APP for a school project" />
</svelte:head>

<section>
    <button
        class={hasNFCSupport ? "" : "disabled"}
        type="button"
        on:click={readNDEF}>Toast</button
    >
    <Toaster />
</section>

<style>
    :global(body) {
        background-color: #282c34; /* Light background color */
        color: #bbc2cf; /* Light text color */
        display: flex;
        align-items: center;
        justify-content: center;
        height: 100vh; /* Ensure the full height of the viewport is covered */
        margin: 0; /* Remove default margin to prevent unwanted spacing */
    }
    button {
        text-align: center;
        border: none;
        color: rgb(41, 38, 38);
        padding: 15px 32px;
        text-decoration: none;
        display: inline-block;
        font-size: 20px;
        border-radius: 1em;
        box-shadow:
            0 8px 16px 0 rgba(0, 0, 0, 0.2),
            0 6px 20px 0 rgba(0, 0, 0, 0.19);
    }

    button.disabled {
        opacity: 0.2;
        cursor: not-allowed;
    }
</style>
