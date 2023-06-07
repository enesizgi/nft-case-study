<script lang="ts">
    export let nft;
    export let showModal;
    import {getContext, onMount} from 'svelte';
    import ShoppingCart from '$lib/images/shopping-cart.svelte';

    const basket = getContext('basket');
    const account = getContext('account');
    const price = getContext('price');
    let metadata;
    console.log($basket, $account);

    let isInBasket = false;

    $: $basket && nft && $basket[nft.id] ? isInBasket = true : isInBasket = false;
    $: usdPrice = $price && nft && parseFloat((nft.price * $price).toFixed(2));

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

<div class="nft-container {isInBasket && 'basket-nft-container'} {showModal && 'isModal'}">
    <div class="card-content">
        {#if metadata?.image}
            <img src={metadata?.image} alt="Svelte logo"/>
        {/if}
        <!--    <img src={metadata?.image} alt="Svelte logo"/>-->
        <div class="card-info">
            <div>{metadata?.name}</div>
            <div>Price: {nft.price} ETH ({usdPrice} USD)</div>
        </div>
    </div>
    {#if !showModal}
        <div class="buy-container">
            <button type="button">Buy</button>
            <button type="button"
                    on:click={isInBasket ? removeFromBasketHandler: addToBasketHandler}>
                <ShoppingCart --fillColor={isInBasket ? 'red' :'white'}/>
            </button>
        </div>
    {/if}
</div>

<style lang="scss">
  .nft-container {
    border: 5px solid #cccccc;
    margin: 5px;
    width: 25%;
    flex: 16%;

    img {
      max-width: 100%;
      padding: 1rem;
      box-sizing: border-box;
    }
  }

  .basket-nft-container {
    border: 5px solid #4075a6;
    box-shadow: 0 0 10px #ff0000;
    margin-left: 0;
    margin-bottom: 1rem;
  }

  .card-info {
    div {
      padding-left: 0.5rem;
      padding-bottom: 0.5rem;
    }
  }

  .buy-container {
    transition: all 0.2s linear;
    visibility: hidden;
    display: flex;
    align-items: center;
    padding-left: 0.5rem;
    background: #4075a6;

    * {
      cursor: pointer;
    }

    *:hover {
      transform: scale(1.1);
    }

    button {
      flex: 60%;
      background: none;
      border: none;
      color: white;
      margin-right: 0.5rem;
    }

    button:first-child {
      font-size: 1.5rem;
      font-weight: bold;
    }
  }

  .nft-container:hover .buy-container {
    transition: all 0.5s ease-in-out;

    visibility: visible;
  }

  .isModal {
    width: 100%;

    .card-content {
      display: flex;
    }

    .card-info {
      padding-top: 1rem;
    }

    img {
      width: 20%;
    }
  }
</style>
