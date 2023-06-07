<script>
  import Card from "./Card.svelte";
  import {getContext} from 'svelte';

  const price = getContext('price');

  const nfts = [
    {
      id: 0,
      metadata: "ipfs://bafkreibvhe564s5jrbrzhokp55bcmkvw7n5qakop5g65uduwhnfkv45plm",
      price: 1,
    },
    {
      id: 1,
      metadata: "ipfs://bafkreiagl5bvxrg4yzi33zqnh2bmnpof6a6jaejfo4h2qag2m3chv4w3ze",
      price: 1,
    },
    {
      id: 2,
      metadata: "ipfs://bafkreicddswjo54ga5bm3yeojcaeqwckqetyfwhtdrpij35henyq2iwujq",
      price: 1,
    },
    {
      id: 3,
      metadata: "ipfs://bafkreifazmgz2depv5p6sew6zbh6orft4us5sjpcmwshm3rw3alx4rol7i",
      price: 1,
    },
    {
      id: 4,
      metadata: "ipfs://bafkreidek6cumjehbjf4nvbzl6yktcrkthov42vafls2baxsqj35afp34u",
      price: 1,
    },
    {
      id: 5,
      metadata: "ipfs://bafkreiezlvqyau6kjziuksnbhg2gnoeblga557g64yz2ep2unfkehazega",
      price: 1,
    },
    {
      id: 6,
      metadata: "ipfs://bafkreihw6xgjhq6bnihzmnvpeqzv6hsla6qe3mnohh7gamgegg5gb5t3w4",
      price: 1,
    },
    {
      id: 7,
      metadata: "ipfs://bafkreih7a7ctsfjdkhhznm4zrxfgnksuubtf3padnvdqgp537a3gtmy754",
      price: 1,
    },
    {
      id: 8,
      metadata: "ipfs://bafkreiemakwynm3seqlxub4m5qapiavab3pzv54vfn4fiyfjbgzxlrpky4",
      price: 1,
    },
    {
      id: 9,
      metadata: "ipfs://bafkreic53igmvxfyne5bb3fl6u7ytawtqqbitoymuc5c4hv3hpcyaiai3m",
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
            <Card nft={nft} showModal={false}/>
        {/each}
    </div>

</section>

<style>
    .nft-display-container {
        display: flex;
        justify-content: center;
        align-items: center;
        flex-wrap: wrap;
    }

    section {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        flex: 0.6;
    }
</style>
