<script lang="ts">
  import { onMount } from 'svelte';
  import {slide} from 'svelte/transition';

  // Copied items is the contents that we'll be displaying
  let copiedItems: string[] = [];
  // Buffer records last clipboard item
  let buffer: string = '';

  // Clipboard status describes our permission level on getting access to the clipboard
  let clipboardStatus: PermissionStatus = {
    state: 'denied',
  } as PermissionStatus;
  // Can we read the clipboard?
  let canReadClipboard = false;

  // If the clipboardStatus is granted, then we naturally can read clipboard
  $: {
    canReadClipboard = clipboardStatus.state == 'granted' ? true : false;
  }
  $: {
    // If something changes so that we can read clipboard, start the process!
    // Memory leak if someone disables read clipboard access then enables it, but we don't talk about that
    // if (canReadClipboard) {
    //   read();
    // }
  }

  // Sets clipboard status by querying permissions
  // Is run at the beginning of the program
  const getClipboardStatus = async (): Promise<void> => {
    const clipboardPermissionName: PermissionName = 'clipboard-write' as PermissionName;
    clipboardStatus = await navigator.permissions.query({
      name: clipboardPermissionName,
    });
  };

  // Read cycle every 300ms.
  const read = async () => {
    // Always check to make sure we can read clipboard
    if (canReadClipboard) {
      // Read clipboard and store in buffer
      try {
        buffer = await navigator.clipboard.readText();
        // If our last item is not identical,
        if (copiedItems[copiedItems.length - 1] != buffer) {
          // Update the buffer
          copiedItems.push(buffer);
          copiedItems = copiedItems;
        }
      } catch (e) {}
      // Repeat
      setTimeout(read, 300);
    }
  };

  onMount(async () => {
    // Initialize clipboard status();
    await getClipboardStatus();
    clipboardStatus.addEventListener('change', getClipboardStatus);

    read();
  });
</script>

<main>
  {#if canReadClipboard}
    <!-- Make sure clipboard hooks into `main`, as stated here! -->
    <p>Clipboard Inserter</p>
    {#each copiedItems as copiedItem}
      <p transition:slide>{copiedItem}</p>
    {/each}
  {:else}
    Please accept prompt to read clipboard
  {/if}
</main>

<style lang="scss">
  :global(body) {
    margin: 0;
    padding: 0;
  }
  main {
    // Required
    padding: 0;
    height: 100vh;
    width: 100vw;

    // Theme
    background-color: hsl(226, 23%, 11%);
    color: #bcbdd0;

    // CSS scroll snap!
    scroll-snap-type: y proximity;
    overflow: auto;

    // Display grid so we can get gap
    display: grid;
    gap: 1rem;

    // Every `p` inside main
    & > p {
      // Required
      margin: 0;
      padding: 1rem;

      // Theme
      background-color: #282d3f;
      color: #bcbdd0;

      // Scroll snap at end, may change later
      scroll-snap-align: end;
    }
  }
</style>
