<script>
  // import Counter from './Counter.svelte';
  // import welcome from '$lib/images/svelte-welcome.webp';
  // import welcome_fallback from '$lib/images/svelte-welcome.png';
  import Card from "./Card.svelte";
  import {setContext, onMount} from 'svelte';
  import {writable} from 'svelte/store';

  const basket = writable({});
  setContext('basket', basket);

  let price;

  onMount(async () => {
    const response = await fetch('https://api.coingecko.com/api/v3/simple/price?ids=ethereum&vs_currencies=usd');
    const data = await response.json();
    price = data.ethereum.usd;
  })

  const nfts = [
    {
      id: 0,
      image: 'https://www.sveltejs.cn/svelte-logo-horizontal.svg',
      metadata: "ipfs://bafkreibvhe564s5jrbrzhokp55bcmkvw7n5qakop5g65uduwhnfkv45plm",
      price: 1,
    },
    {
      id: 1,
      image: 'https://www.sveltejs.cn/svelte-logo-horizontal.svg',
      metadata: "ipfs://bafkreiagl5bvxrg4yzi33zqnh2bmnpof6a6jaejfo4h2qag2m3chv4w3ze",
      price: 1,
    },
  ];
</script>

<svelte:head>
    <title>Home</title>
    <meta name="description" content="Svelte demo app"/>
</svelte:head>

<section>
    <div class="nft-display-container">
        {#each nfts as nft}
            <Card nft={nft} price={price}/>
        {/each}
    </div>
    <!--	<h1>-->
    <!--		<span class="welcome">-->
    <!--			<picture>-->
    <!--				<source srcset={welcome} type="image/webp" />-->
    <!--				<img src={welcome_fallback} alt="Welcome" />-->
    <!--			</picture>-->
    <!--		</span>-->

    <!--		to your new<br />SvelteKit app-->
    <!--	</h1>-->

    <!--	<h2>-->
    <!--		try editing <strong>src/routes/+page.svelte</strong>-->
    <!--	</h2>-->

    <!--	<Counter />-->


</section>

<style>
    .nft-display-container {
        display: flex;
        justify-content: center;
        align-items: center;
    }

    section {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        flex: 0.6;
    }

    h1 {
        width: 100%;
    }

    .welcome {
        display: block;
        position: relative;
        width: 100%;
        height: 0;
        padding: 0 0 calc(100% * 495 / 2048) 0;
    }

    .welcome img {
        position: absolute;
        width: 100%;
        height: 100%;
        top: 0;
        display: block;
    }
</style>
