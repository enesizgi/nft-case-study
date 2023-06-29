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

<div class="group m-1 border-4 border-solid border-gray-200 bg-[#293249] w-1/4 rounded-xl nft-container {isInBasket && 'basket-nft-container'} {showModal && 'isModal'}">
    <div class="rounded-t-xl text-white card-content">
        <div class="overflow-hidden rounded-t-md">
            <img class="box-border max-w-full rounded-t-l transition ease-in-out duration-300 group-hover:scale-110"
                 src={metadata?.image ? metadata.image : imagePlaceholder}
                 alt="{metadata?.image ? 'Svelte Logo' : 'placeholder'}"/>
        </div>
        <div class="mt-3 card-info">
            <div>{metadata?.name}</div>
            <div>{nft.price} ETH ({usdPrice} USD)</div>
        </div>

        {#if showModal}
            <button on:click={removeFromBasketHandler} class="close-icon-button">
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

    .nft-container {
        flex: 16%;
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


    .isModal {
        width: 100%;

        .card-content {
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .card-info {
            padding-top: 1rem;
        }

        img {
            width: 20%;
        }
    }

    .close-icon-button {
        border: none;
        background: #d7c9c9 none;
        border-radius: 50%;
        fill: #de0f46;
        height: 2rem;
        position: relative;
        right: 1rem;
        visibility: hidden;
    }

    .nft-container:hover .close-icon-button {
        visibility: visible;
    }
</style>
