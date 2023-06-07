<script lang="ts">
    export let nft;
    export let price;
    import {getContext, onMount} from 'svelte';

    const basket = getContext('basket');
    let metadata;
    console.log($basket);

    let isInBasket = false;

    onMount(async () => {
        metadata = await getMetadata();
    });

    const getMetadata = async () => {
        const metadataUrl = nft.metadata.startsWith('ipfs://')
            ? `https://dweb.link/ipfs/${nft.metadata.slice(7)}`
            : nft.metadata;
        const response = await fetch(metadataUrl);
        return response.json();
    };

    const addToBasketHandler = () => {
        const newBasket = $basket;
        newBasket[nft.id] = nft;
        basket.set(newBasket);
        isInBasket = true;
        console.log($basket);
    };

    const removeFromBasketHandler = () => {
        const newBasket = $basket;
        delete newBasket[nft.id];
        basket.set(newBasket);
        isInBasket = false;
        console.log($basket);
    };
</script>

<div class="nft-container">
    {#if metadata?.image}
        <img src={metadata?.image} alt="Svelte logo"/>
    {/if}
    <!--    <img src={metadata?.image} alt="Svelte logo"/>-->
    <div>{metadata?.name}</div>
    <div>NFT Price: {price}</div>
    <div>
        <button type="button">Buy</button>
        <button type="button"
                on:click={isInBasket ? removeFromBasketHandler: addToBasketHandler}>{isInBasket ? 'Remove from Cart' : 'Add to Cart'}</button>
    </div>
</div>

<style lang="scss">
  .nft-container {
    border: 5px solid #cccccc;
    margin: 5px;

    img {
      width: 20%;
    }
  }
</style>
