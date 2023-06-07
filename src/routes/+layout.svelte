<script>
  import Header from './Header.svelte';
  import './styles.css';
  import {writable} from "svelte/store";
  import {setContext, onMount} from "svelte";

  const account = writable('');
  setContext('account', account);

  const price = writable(0);
  setContext('price', price);

  const basket = writable({});
  console.log('basket', $basket)
  setContext('basket', basket);
  if (typeof localStorage !== "undefined") {
    const localBasket = localStorage.getItem('basket');
    basket.set(localBasket ? JSON.parse(localBasket) : {});
    console.log('localStorage', localStorage, $basket);
    console.log($basket);
  }

  onMount(async () => {
    const response = await fetch('https://api.coingecko.com/api/v3/simple/price?ids=ethereum&vs_currencies=usd');
    const data = await response.json();
    price.set(data.ethereum.usd);
  })

</script>

<div class="app">
    <Header/>

    <main>
        <slot/>
    </main>

    <footer>
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
</style>
