<script lang="ts">
    export let nft;
    import {getContext, onMount} from 'svelte';

    const basket = getContext('basket');
    const account = getContext('account');
    const price = getContext('price');
    let metadata;
    console.log($basket, $account);

    let isInBasket = false;

    $: $basket && nft && $basket[nft.id] ? isInBasket = true : isInBasket = false;

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
        if (!$account) {
            console.log($account)
            console.log('Please connect your wallet');
            return;
        }
        const newBasket = $basket;
        newBasket[nft.id] = nft;
        basket.set(newBasket);
        localStorage.setItem('basket', JSON.stringify(newBasket));
        isInBasket = true;
        console.log($basket);
    };

    const removeFromBasketHandler = () => {
        const newBasket = $basket;
        delete newBasket[nft.id];
        basket.set(newBasket);
        localStorage.setItem('basket', JSON.stringify(newBasket));
        isInBasket = false;
        console.log($basket);
    };
</script>

<div class="nft-container {isInBasket ? 'basket-nft-container' : ''}">
    {#if metadata?.image}
        <img src={metadata?.image} alt="Svelte logo"/>
    {/if}
    <!--    <img src={metadata?.image} alt="Svelte logo"/>-->
    <div>{metadata?.name}</div>
    <div>NFT Price: {$price}</div>
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
    width: 25%;

    img {
      max-width: 100%;
    }
  }

  .basket-nft-container {
    border: 5px solid #4075a6;
    box-shadow: 0 0 10px #ff0000;
  }
</style>
