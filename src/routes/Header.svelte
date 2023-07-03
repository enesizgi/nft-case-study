<script>
  import {onMount, getContext} from "svelte";
  import logo from '$lib/images/svelte-logo.svg';
  import Modal from './Modal.svelte';
  import BasketSvg from '$lib/images/shopping-basket.svelte';
  import Card from "./Card.svelte";
  import {ethers} from 'ethers';

  const CHAIN_PARAMS = getContext('CHAIN_PARAMS');
  // Retrieve user store from context
  const account = getContext('account');
  const basket = getContext('basket');
  const price = getContext('price');
  const notificationMessage = getContext('notificationMessage');
  const marketplaceContract = getContext('marketplaceContract');
  $: basketFlat = $basket ? Object.entries($basket) : [];
  $: totalPrice = Object.values($basket).reduce((acc, nft) => acc.add(nft.price), ethers.BigNumber.from(0));
  $: totalPriceUSD = totalPrice && $price && parseFloat((totalPrice * $price).toString())

  let width = 0;

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

  const changeNetwork = async networkId => {
    try {
      await window.ethereum.request({
        method: 'wallet_switchEthereumChain',
        params: [{chainId: networkId}] // chainId must be in hexadecimal numbers
      });
      return true;
    } catch (error) {
      // This error code indicates that the chain has not been added to MetaMask
      // if it is not, then install it into the user MetaMask
      if (error.code === 4902) {
        try {
          await window.ethereum.request({
            method: 'wallet_addEthereumChain',
            params: [CHAIN_PARAMS[networkId]]
          });
          return true;
        } catch (addError) {
          console.error(addError);
          return false;
        }
      }
      console.error(error);
      return false;
    }
  };

  const connectButtonHandler = async () => {
    if (!window.ethereum) {
      console.log('Metamask not installed');
      return;
    }
    try {
      networkId = await getChainIdOfAccount();
      if (!(networkId in $CHAIN_PARAMS)) {
        const success = await changeNetwork('0xaa36a7');
        if (!success) {
          networkId = '';
          return;
        }
      }

      account.set(await getAccounts());
    } catch (e) {
      networkId = '';
      account.set('');
      console.error(e);
    }
  };

  const purchaseHandler = async () => {
    if (!$account) {
      const message = ['Please connect your wallet first.', 'error'];
      if (!$notificationMessage.find(i => i === message)) {
        const newMessages = $notificationMessage;
        newMessages.push(message);
        notificationMessage.set(newMessages);
        setTimeout(() => {
          const messages = $notificationMessage;
          messages.shift();
          notificationMessage.set(messages);
        }, 2000);
        return;
      }
    }
    await $marketplaceContract.purchaseItem_multiple(Object.values($basket).map(i => i.purchaseId), {
      value: ethers.utils.parseEther(totalPrice.toString())
    });
  };

  const disconnectHandler = () => {
    account.set('');
    networkId = '';
  };

</script>

<svelte:window bind:innerWidth={width}/>

<header>

    <div class="basket">
        <button type="button" on:click={() => showModal = true}>
            <BasketSvg/>
        </button>
    </div>
    <nav>
        {#if width >= 480}
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
        {/if}
        <button type="button" class="connect"
                on:click={connectButtonHandler}>{$account ? shortAccount : 'Connect Wallet'}</button>
        {#if $account}
            <button type="button" class="disconnect" on:click={disconnectHandler}>Disconnect</button>
        {/if}
        <div class="corner">
            <!-- svelte-ignore a11y-invalid-attribute-->
            <a href="#">
                <img src={logo} alt="SvelteKit"/>
            </a>
        </div>
    </nav>

</header>

<Modal bind:showModal>
    {#if $basket}
        {#each basketFlat as [key, value] (key)}
            <Card nft={value} {showModal}/>
        {/each}
        <div class="text-white m-1 price-container">
            <div>Total: {totalPrice} ETH ({totalPriceUSD} USD)</div>
            <button type="button" on:click={purchaseHandler}>Purchase</button>
        </div>
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
    background-color: #2b305d;

    .basket {
      display: flex;
      margin-left: 0.5rem;
    }
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

    ul, .basket, .connect, .disconnect {

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

    a, .basket, .connect, .disconnect {
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

  .disconnect:hover {
    color: red;
  }

  .price-container {
    display: flex;
    justify-content: space-between;
    align-items: center;

    button {
      background: #4075a6 none;
      color: white;
      border-radius: 0.5rem;
      padding: 1rem;
    }
  }
</style>
