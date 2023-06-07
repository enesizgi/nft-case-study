<script>
  import Card from "./Card.svelte";
  import {getContext} from 'svelte';

  const price = getContext('price');

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
            <Card nft={nft}/>
        {/each}
    </div>

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
</style>
