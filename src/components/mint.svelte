<script>
	import { ethers } from 'ethers';
	import { createEventDispatcher } from 'svelte';
	import myEpicNFT from '../utils/MyEpicNFT.json';
	$: mining = false;
	$: networkName = null;
	$: openSeaAddress = null;
	const { ethereum } = window;
	const contractAddress = '0x8A0ba361403Dd83a824a329678F9CAE1023b954a';
	let provider = null;
	let signer = null;
	let myEpicNFTContract = null;
	let dispatch = createEventDispatcher();
	if (ethereum) {
		provider = new ethers.providers.Web3Provider(ethereum);
		signer = provider.getSigner();
		myEpicNFTContract = new ethers.Contract(contractAddress, myEpicNFT.abi, signer);
		const getNetworkName = async () => {
			const network = await provider.getNetwork();
			networkName = network.name;
		};
		myEpicNFTContract.on('NewEpicNFTMinted', (from, tokenId) => {
			openSeaAddress = `https://testnets.opensea.io/assets/${contractAddress}/${tokenId.toNumber()}`;
			dispatch('newEpicNFTMinted', { from, tokenId });
		});
		getNetworkName();
	}

	const mint = async () => {
		try {
			mining = true;
			const nftTxn = await myEpicNFTContract.makeAnEpicNFT();
			await nftTxn.wait();
			mining = false;
		} catch (error) {}
	};
</script>

{#if networkName != 'rinkeby'}
	<h2>USE RINKEBY TEST NETWORK</h2>
{:else if !mining}
	{#if openSeaAddress}
		<h3>Minted!</h3>
		<p>It may take a few minutes for the image to show on the link below</p>
		<p>
			<a href={openSeaAddress}>OpenSea NFT</a>
		</p>
		<p>👆 👀 PEEP YOUR NEW ART! 👀 👆</p>
		<p>OR</p>
		<p>Mint another NFT 👇</p>
	{/if}
	<p>
		<button on:click={mint}>Click to mint ⚒️ an NFT!</button>
	</p>
{:else}
	<p>🤖 is currently minting ⚒️...</p>
{/if}

<style>
	h2,
	h3 {
		text-align: center;
	}
	p {
		text-align: center;
	}
	.mining {
		background-color: darkgoldenrod;
	}
	.wave {
		background-color: var(--purple);
		margin-top: 16px;
		padding: 8px;
		box-shadow: var(--level-10);
		border-radius: 8px;
	}
	.tna {
		color: var(--lightBlack);
		display: flex;
		justify-content: space-between;
	}
	.msg {
		margin-top: 1rem;
		text-align: center;
		font-size: var(--h4);
	}
</style>
