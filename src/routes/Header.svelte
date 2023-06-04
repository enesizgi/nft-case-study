<script>
  import {onMount} from "svelte";
  import {page} from '$app/stores';
  import logo from '$lib/images/svelte-logo.svg';
  import github from '$lib/images/github.svg';

  let networkId = '';
  let chainName = '';
  let account = '';

  $: shortAccount = account ? `${account.slice(0, 5)}...${account.slice(-3)}` : '';
  $: chainName = networkId ? getChainName(networkId) : '';

  onMount(async () => {
    if (window.ethereum) {
      window.ethereum.on('chainChanged', async chainId => {
        networkId = chainId;
      });

      window.ethereum.on('accountsChanged', async accounts => {
        account = accounts[0];
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
      account = await getAccounts();
    } catch (e) {
      networkId = '';
      account = '';
      console.error(e);
    }
    console.log(networkId, account, 'connectButtonHandler');
  };

</script>

<header>
	<div class="corner">
		<a href="https://kit.svelte.dev">
			<img src={logo} alt="SvelteKit" />
		</a>
	</div>

	<nav>
		<svg viewBox="0 0 2 3" aria-hidden="true">
			<path d="M0,0 L1,2 C1.5,3 1.5,3 2,3 L2,0 Z" />
		</svg>
		<ul>
			<li aria-current={$page.url.pathname === '/' ? 'page' : undefined}>
				<a href="/">Home</a>
			</li>
			<li aria-current={$page.url.pathname === '/about' ? 'page' : undefined}>
				<a href="/about">About</a>
			</li>
			<li aria-current={$page.url.pathname.startsWith('/sverdle') ? 'page' : undefined}>
				<a href="/sverdle">Sverdle</a>
			</li>
		</ul>
		<svg viewBox="0 0 2 3" aria-hidden="true">
			<path d="M0,0 L0,3 C0.5,3 0.5,3 1,2 L2,0 Z" />
		</svg>
	</nav>

    <!--	<div class="corner">-->
    <!--		<a href="https://github.com/sveltejs/kit">-->
    <!--			<img src={github} alt="GitHub" />-->
    <!--		</a>-->
    <!--	</div>-->
    <div>Chain: {chainName}</div>
    <button type="button" on:click={connectButtonHandler}>{account ? shortAccount : 'Connect Wallet'}</button>
</header>

<style>
	header {
		display: flex;
		justify-content: space-between;
	}

	.corner {
		width: 3em;
		height: 3em;
	}

	.corner a {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 100%;
		height: 100%;
	}

	.corner img {
		width: 2em;
		height: 2em;
		object-fit: contain;
	}

	nav {
		display: flex;
		justify-content: center;
		--background: rgba(255, 255, 255, 0.7);
	}

	svg {
		width: 2em;
		height: 3em;
		display: block;
	}

	path {
		fill: var(--background);
	}

	ul {
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

	li {
		position: relative;
		height: 100%;
	}

	li[aria-current='page']::before {
		--size: 6px;
		content: '';
		width: 0;
		height: 0;
		position: absolute;
		top: 0;
		left: calc(50% - var(--size));
		border: var(--size) solid transparent;
		border-top: var(--size) solid var(--color-theme-1);
	}

	nav a {
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

	a:hover {
		color: var(--color-theme-1);
	}
</style>
