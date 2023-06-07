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
>
    <div on:click|stopPropagation>
        <button on:click={() => dialog.close()}>
            <CloseIcon/>
            <!--            <img src={CloseIcon} alt="closeModalIcon" width="20px" height="20px"/>-->
        </button>
        <slot name="header"/>
        <hr/>
        <slot/>
        <hr/>
    </div>
</dialog>

<style lang="scss">
  dialog {
    left: 70%;
    height: 100%;
    max-width: 32em;
    border-radius: 0.2em;
    border: none;
    padding: 0;

    div > button {
      border: none;
      background: #d7c9c9 none;
      border-radius: 50%;
      fill: #de0f46;
      padding: 0.3em;
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
