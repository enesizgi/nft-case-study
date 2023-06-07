<script>
  import Card from "./Card.svelte";
  import {getContext} from "svelte";

  const marketplaceContract = getContext("marketplaceContract");
  const nfts = getContext("nfts");

</script>

<svelte:head>
    <title>Home</title>
    <meta name="description" content="Svelte demo app"/>
</svelte:head>

<section>
    <div class="nft-display-container">
        {#each $nfts as nft (nft.id)}
            <Card nft={nft} showModal={false} marketplaceContract={$marketplaceContract}/>
        {/each}
    </div>

</section>

<style>
    .nft-display-container {
        display: flex;
        justify-content: center;
        align-items: center;
        flex-wrap: wrap;
    }

    section {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        flex: 0.6;
    }
</style>
