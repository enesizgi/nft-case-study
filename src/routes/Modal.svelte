<script>
  export let showModal; // boolean
  import CloseIcon from "$lib/images/close_icon.svelte";

  let dialog; // HTMLDialogElement

  $: if (dialog && showModal) dialog.showModal();
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<dialog
        bind:this={dialog}
        on:close={() => (showModal = false)}
        on:click|self={() => dialog.close()}
        class="sm:w-1/2 sm:left-1/2 md:w-1/3 md:left-2/3 lg:w-1/4 lg:left-3/4 xl:w-1/5 xl:left-[80%] bg-[#131A2A]"
>
    <div on:click|stopPropagation>
        <button on:click={() => dialog.close()} class="m-1 p-1">
            <CloseIcon/>
        </button>
        <slot name="header"/>
        <hr/>
        <slot/>
        <hr/>
    </div>
</dialog>

<style lang="scss">
  dialog {
    height: 100%;
    padding: 0;

    div > button {
      border: none;
      background: #d7c9c9 none;
      border-radius: 50%;
      fill: #de0f46;
    }
  }

  dialog::backdrop {
    background: rgba(0, 0, 0, 0.3);
  }

  dialog > div {
    padding: 1em;
  }

  dialog[open] {
    animation: zoom 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  }

  @keyframes zoom {
    from {
      transform: scale(0.95);
    }
    to {
      transform: scale(1);
    }
  }

  dialog[open]::backdrop {
    animation: fade 0.2s ease-out;
  }

  @keyframes fade {
    from {
      opacity: 0;
    }
    to {
      opacity: 1;
    }
  }

  button {
    display: block;
  }
</style>
