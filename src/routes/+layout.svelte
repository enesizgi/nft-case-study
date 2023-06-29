<script>
  import "../app.css";
  import Header from './Header.svelte';
  import './styles.css';
  import {writable} from "svelte/store";
  import {setContext, onMount} from "svelte";

  import * as MarketplaceAddressSepolia from '$lib/contractsData/sepolia/Marketplace-address.json';
  import * as MarketplaceAbiSepolia from '$lib/contractsData/sepolia/Marketplace.json';

  const CONTRACTS = writable({
    ['0xaa36a7']: {
      MARKETPLACE: {
        address: MarketplaceAddressSepolia.address,
        abi: MarketplaceAbiSepolia.abi
      },
    }
  });
  setContext('CONTRACTS', CONTRACTS);

  const CHAIN_PARAMS = writable({
    '0xaa36a7': {
      chainId: '0xaa36a7',
      rpcUrls: ['https://rpc.ankr.com/eth_sepolia'],
      chainName: 'Sepolia',
      nativeCurrency: {
        name: 'Ether',
        symbol: 'ETH', // 2-6 characters long
        decimals: 18
      },
      blockExplorerUrls: ['https://sepolia.etherscan.io']
    }
  });
  setContext('CHAIN_PARAMS', CHAIN_PARAMS);

  const account = writable('');
  setContext('account', account);

  const price = writable(0);
  setContext('price', price);

  const notificationMessage = writable([]);
  setContext('notificationMessage', notificationMessage);

  const basket = writable({});
  setContext('basket', basket);
  if (typeof localStorage !== "undefined") {
    const localBasket = localStorage.getItem('basket');
    basket.set(localBasket ? JSON.parse(localBasket) : {});
  }

  onMount(async () => {
    const response = await fetch('https://api.coingecko.com/api/v3/simple/price?ids=ethereum&vs_currencies=usd');
    const data = await response.json();
    price.set(data.ethereum.usd);
  })

  import {ethers} from 'ethers';

  const getProviderOrSignerFn = (userId, chainId) => {
    if (userId && chainId) {
      const provider = new ethers.providers.Web3Provider(window.ethereum);
      return provider.getSigner();
    }
    return new ethers.providers.JsonRpcProvider($CHAIN_PARAMS['0xaa36a7'].rpcUrls[0]);
  };

  const getMarketplaceContractFn = (userId, chainId) => {
    let marketplaceContract;
    if (chainId && $CONTRACTS['0xaa36a7']) marketplaceContract = $CONTRACTS['0xaa36a7'].MARKETPLACE;
    else marketplaceContract = $CONTRACTS['0xaa36a7'].MARKETPLACE;
    const providerOrSigner = getProviderOrSignerFn(userId, chainId);
    return new ethers.Contract(marketplaceContract.address, marketplaceContract.abi, providerOrSigner);
  };

  const marketplaceContract = writable();
  setContext('marketplaceContract', marketplaceContract);
  $: $CHAIN_PARAMS && $CONTRACTS && marketplaceContract.set(getMarketplaceContractFn($account, '0xaa36a7'));

  let nfts = writable([
    {
      id: 0,
      metadata: "ipfs://bafkreibvhe564s5jrbrzhokp55bcmkvw7n5qakop5g65uduwhnfkv45plm",
      price: 1,
      purchaseId: 11
    },
    {
      id: 1,
      metadata: "ipfs://bafkreiagl5bvxrg4yzi33zqnh2bmnpof6a6jaejfo4h2qag2m3chv4w3ze",
      price: 1,
      purchaseId: 10
    },
    {
      id: 2,
      metadata: "ipfs://bafkreicddswjo54ga5bm3yeojcaeqwckqetyfwhtdrpij35henyq2iwujq",
      price: 1,
      purchaseId: 9
    },
    {
      id: 3,
      metadata: "ipfs://bafkreifazmgz2depv5p6sew6zbh6orft4us5sjpcmwshm3rw3alx4rol7i",
      price: 1,
      purchaseId: 8
    },
    {
      id: 4,
      metadata: "ipfs://bafkreidek6cumjehbjf4nvbzl6yktcrkthov42vafls2baxsqj35afp34u",
      price: 1,
      purchaseId: 7
    },
    // {
    //   id: 5,
    //   metadata: "ipfs://bafkreiezlvqyau6kjziuksnbhg2gnoeblga557g64yz2ep2unfkehazega",
    //   price: 1,
    //   purchaseId: 6
    // },
    // {
    //   id: 6,
    //   metadata: "ipfs://bafkreihw6xgjhq6bnihzmnvpeqzv6hsla6qe3mnohh7gamgegg5gb5t3w4",
    //   price: 2,
    //   purchaseId: 5
    // },
    // {
    //   id: 7,
    //   metadata: "ipfs://bafkreih7a7ctsfjdkhhznm4zrxfgnksuubtf3padnvdqgp537a3gtmy754",
    //   price: 1,
    //   purchaseId: 4
    // },
    // {
    //   id: 12,
    //   metadata: "ipfs://bafkreiemakwynm3seqlxub4m5qapiavab3pzv54vfn4fiyfjbgzxlrpky4",
    //   price: 1,
    //   purchaseId: 3
    // },
    // {
    //   id: 9,
    //   metadata: "ipfs://bafkreic53igmvxfyne5bb3fl6u7ytawtqqbitoymuc5c4hv3hpcyaiai3m",
    //   price: 1,
    //   purchaseId: 1
    // },
  ]);
  setContext('nfts', nfts);

  $: nfts.set($nfts.filter(nft => {
    if (!nft.purchaseId) return false;
    if (!$marketplaceContract) return false;
    return $marketplaceContract.items(nft.purchaseId).then(item => {
      return item.sold === false && item.canceled === false;
    });
  }));

</script>

<div class="app">
    <Header/>

    <main>
        <slot/>
    </main>

    <footer>
        {#if $notificationMessage.length}
            <div class="notification" style="color: {$notificationMessage[0][1] === 'error' ? 'red': 'green'};">{$notificationMessage[0][0]}</div>
        {/if}
        <p>Supported Networks: Sepolia</p>
        <p>NFT Marketplace</p>
        <!--        <p>visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to learn SvelteKit</p>-->
    </footer>
</div>

<style>
    .app {
        display: flex;
        flex-direction: column;
        min-height: 100vh;
    }

    main {
        flex: 1;
        display: flex;
        flex-direction: column;
        padding: 1rem;
        width: 100%;
        max-width: 64rem;
        margin: 0 auto;
        box-sizing: border-box;
    }

    footer {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        padding: 12px;
    }

    @media (min-width: 480px) {
        footer {
            padding: 12px 0;
        }
    }

    .notification {
        color: red;
    }
</style>
