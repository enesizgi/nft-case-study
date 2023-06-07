<script>
  import {onMount, getContext} from "svelte";
  import logo from '$lib/images/svelte-logo.svg';
  import Modal from './Modal.svelte';
  import BasketSvg from '$lib/images/shopping-basket.svelte';
  import Card from "./Card.svelte";
  import {ethers} from 'ethers';

  // Retrieve user store from context
  const account = getContext('account');
  const basket = getContext('basket');
  const price = getContext('price');
  $: basketFlat = $basket ? Object.entries($basket) : [];
  $: totalPrice = Object.values($basket).reduce((acc, nft) => acc.add(nft.price), ethers.BigNumber.from(0));

  let networkId = '';
  let chainName = '';
  let showModal = false;

  $: shortAccount = account ? `${$account.slice(0, 5)}...${$account.slice(-3)}` : '';

  onMount(async () => {
    if (window.ethereum) {
      window.ethereum.on('chainChanged', async chainId => {
        networkId = chainId;
      });

      window.ethereum.on('accountsChanged', async accounts => {
        account.set(accounts[0]);
      });
    }
  });

  const getChainName = (chainId) => {
    switch (chainId) {
      case '0x1':
        return 'Ethereum Mainnet';
      case '0x3':
        return 'Ropsten';
      case '0x4':
        return 'Rinkeby';
      case '0x5':
        return 'Goerli';
      case '0x2a':
        return 'Kovan';
      case '0xaa36a7':
        return 'Sepolia'
      default:
        return 'Unknown';
    }
  };

  $: chainName = networkId ? getChainName(networkId) : '';
  const getChainIdOfAccount = async () => {
    return window.ethereum.request({method: 'eth_chainId'});
  };

  const getAccounts = async () => {
    const accounts = await window.ethereum.request({method: 'eth_requestAccounts'});
    return accounts[0];
  };

  const connectButtonHandler = async () => {
    if (!window.ethereum) {
      console.log('Metamask not installed');
      return;
    }
    try {
      networkId = await getChainIdOfAccount();
      account.set(await getAccounts());
    } catch (e) {
      networkId = '';
      account.set('');
      console.error(e);
    }
    console.log(networkId, $account, 'connectButtonHandler');
  };

  const purchaseHandler = () => {
    console.log('purchaseHandler');
  };

</script>

<header>
    <div class="corner">
        <a href="https://kit.svelte.dev">
            <img src={logo} alt="SvelteKit"/>
        </a>
    </div>

    <nav>
        <svg viewBox="0 0 2 3" aria-hidden="true">
            <path d="M0,0 L1,2 C1.5,3 1.5,3 2,3 L2,0 Z"/>
        </svg>
        <div class="basket">
            <button type="button" on:click={() => showModal = true}>
                <BasketSvg on:click={() => {
                          console.log('click');
                        }}/>
            </button>
        </div>
        <ul>
            <li>
                <!-- svelte-ignore a11y-invalid-attribute-->
                <a href="#">Buy</a>
            </li>
            <li>
                <!-- svelte-ignore a11y-invalid-attribute-->
                <a href="#">Rent</a>
            </li>
            <li>
                <!-- svelte-ignore a11y-invalid-attribute-->
                <a href="#">Contact</a>
            </li>
        </ul>
        <button type="button" class="connect"
                on:click={connectButtonHandler}>{$account ? shortAccount : 'Connect Wallet'}</button>

        <svg viewBox="0 0 2 3" aria-hidden="true">
            <path d="M0,0 L0,3 C0.5,3 0.5,3 1,2 L2,0 Z"/>
        </svg>
    </nav>
    <div>Chain: {chainName}</div>
</header>

<Modal bind:showModal>
    {#if $basket}
        {#each basketFlat as [key, value]}
            <Card nft={value}/>
        {/each}
        <div>Total price: {totalPrice}</div>
        <button type="button" on:click={purchaseHandler}>Purchase</button>
    {/if}
</Modal>

<style lang="scss">
  button {
    background: none;
    border: none;
    cursor: pointer;
    font: inherit;
    line-height: normal;
    padding: 0;
    -webkit-appearance: none;
    -moz-appearance: none;
  }

  header {
    display: flex;
    justify-content: space-between;
  }

  .corner {
    width: 3em;
    height: 3em;

    a {
      display: flex;
      align-items: center;
      justify-content: center;
      width: 100%;
      height: 100%;
    }

    img {
      width: 2em;
      height: 2em;
      object-fit: contain;
    }
  }

  nav {
    display: flex;
    justify-content: center;
    --background: rgba(255, 255, 255, 0.7);

    ul, .basket, .connect {

      li {
        position: relative;
        height: 100%;
      }

      position: relative;
      padding: 0;
      margin: 0;
      height: 3em;
      display: flex;
      justify-content: center;
      align-items: center;
      list-style: none;
      background: var(--background);
      background-size: contain;
    }

    a, .basket, .connect {
      display: flex;
      height: 100%;
      align-items: center;
      padding: 0 0.5rem;
      color: var(--color-text);
      font-weight: 700;
      font-size: 0.8rem;
      text-transform: uppercase;
      letter-spacing: 0.1em;
      text-decoration: none;
      transition: color 0.2s linear;
    }
  }

  svg {
    width: 2em;
    height: 3em;
    display: block;
  }

  path {
    fill: var(--background);
  }

  a:hover, .connect:hover {
    color: var(--color-theme-1);
  }

  .basket:hover {
    fill: #4075a6;
  }
</style>
