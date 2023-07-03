<script lang="ts">
    import CloseIcon from "$lib/images/close_icon.svelte";

    export let nft;
    export let showModal;
    export let marketplaceContract;
    import {getContext, onMount} from 'svelte';
    import {ethers} from 'ethers';
    import ShoppingCart from '$lib/images/shopping-cart.svelte';
    import imagePlaceholder from '$lib/images/image-placeholder.svg';

    const basket = getContext('basket');
    const account = getContext('account');
    const notificationMessage = getContext('notificationMessage');
    const price = getContext('price');
    let metadata;
    let windowWidth;

    let isInBasket = false;

    $: $basket && nft && $basket[nft.id] ? isInBasket = true : isInBasket = false;
    $: usdPrice = $price && nft && parseFloat((nft.price * $price).toFixed(2));
    // $: windowWidth && console.log(windowWidth);

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

    const removeMessage = (timeout = 2000) => {
        setTimeout(() => {
            const messages = $notificationMessage;
            messages.shift();
            notificationMessage.set(messages);
        }, timeout);
    };

    const addToBasketHandler = () => {
        if (!$account) {
            const message = ['Please connect your wallet', 'error'];
            if (!$notificationMessage.find(i => i === message)) {
                const newMessages = $notificationMessage;
                newMessages.push(message);
                notificationMessage.set(newMessages);
                removeMessage();
            }
            return;
        }
        $notificationMessage.push(['Added to basket', 'success']);
        notificationMessage.set($notificationMessage);
        removeMessage();
        const newBasket = $basket;
        newBasket[nft.id] = nft;
        basket.set(newBasket);
        localStorage.setItem('basket', JSON.stringify(newBasket));
        isInBasket = true;
    };

    const removeFromBasketHandler = () => {
        const newBasket = $basket;
        delete newBasket[nft.id];
        basket.set(newBasket);
        localStorage.setItem('basket', JSON.stringify(newBasket));
        isInBasket = false;
        $notificationMessage.push(['Removed from basket', 'error']);
        notificationMessage.set($notificationMessage);
        removeMessage(2000);
    };

    const buyHandler = async () => {
        if (!$account) {
            const message = ['Please connect your wallet', 'error'];
            if (!$notificationMessage.find(i => i === message)) {
                const newMessages = $notificationMessage;
                newMessages.push(message);
                notificationMessage.set(newMessages);
                removeMessage();
            }
            return;
        }
        await marketplaceContract.purchaseItem(nft.purchaseId, {
            value: ethers.utils.parseEther(nft.price.toString()),
            from: $account
        });
    };
</script>

<svelte:window bind:innerWidth={windowWidth}/>

<div class="group m-1 border-4 border-solid bg-[#293249] w-1/4 rounded-xl h-full basis-full sm:basis-1/3
            md:basis-1/4 lg:basis-1/5 xl:basis-1/6 nft-container {isInBasket && 'ml-0 mb-4'}
            {isInBasket && !showModal && 'border-4 border-[#4075a6]'} {showModal && 'w-full border-2 border-white'}">
    <div class="rounded-t-xl text-white card-content {showModal && 'flex'}">
        <div class="overflow-hidden rounded-t-md {showModal && 'w-1/4'}">
            <img class="w-full box-border max-w-full rounded-t-l transition ease-in-out duration-300 group-hover:scale-110"
                 src={metadata?.image ? metadata.image : imagePlaceholder}
                 alt="{metadata?.image ? 'Svelte Logo' : 'placeholder'}"/>
        </div>
        <div class="card-info {showModal ? 'm-0 text-base sm:text-xl w-3/4' : 'mt-3'}">
            <div class="h-6 overflow-hidden">{metadata?.name}</div>
            <div class="h-6 overflow-hidden">{nft.price} ETH ({usdPrice} USD)</div>
        </div>

        {#if showModal}
            <button on:click={removeFromBasketHandler} class="p-1.5 m-1.5 close-icon-button">
                <CloseIcon/>
            </button>
        {/if}
    </div>
    {#if !showModal}
        <div class="opacity-0 transition-all duration-200 ease-in-out flex items-center justify-evenly rounded-b-xl text-xl h-8 buy-container">
            <button type="button" on:click={buyHandler}
                    class="bg-[#2081e2] hover:bg-[#2e8eee] w-4/5 h-full border-r border-white rounded-bl-xl">Buy now
            </button>
            <button type="button"
                    class="bg-[#2081e2] hover:bg-[#2e8eee] w-1/5 h-full flex items-center justify-center rounded-br-xl"
                    on:click={isInBasket ? removeFromBasketHandler: addToBasketHandler}>
                <ShoppingCart --fillColor={isInBasket ? 'red' :'white'}/>
            </button>
        </div>
    {/if}
</div>

<style lang="postcss">

    .card-info {
        div {
            padding-left: 0.5rem;
            padding-bottom: 0.5rem;
        }
    }

    .buy-container {
        align-items: center;

        * {
            cursor: pointer;
        }

        button {
            color: white;
        }
    }

    .nft-container:hover .buy-container {
        opacity: 1;
    }

    .close-icon-button {
        border: none;
        background: #d7c9c9 none;
        border-radius: 50%;
        fill: #de0f46;
        height: 2rem;
        right: 1rem;
        visibility: hidden;
    }

    .nft-container:hover .close-icon-button {
        visibility: visible;
    }
</style>
