<script>
	import Tailwindcss from './Tailwindcss.svelte';

	export let name;
	import { ethStore, web3, selectedAccount, connected } from 'svelte-web3';
	import { onMount } from 'svelte';

	const contractABI = [{"inputs":[],"stateMutability":"payable","type":"constructor"},{"stateMutability":"payable","type":"receive"},{"inputs":[],"name":"betAmount","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"currentBattler","outputs":[{"internalType":"address payable","name":"","type":"address"}],"stateMutability":"view","type":"function"}];
	const contractAddress = '0x3FB09a2b95cb987ac3150B5FcFEEf214071d1986';

	const enableBrowser = () => ethStore.setBrowserProvider();

	$: checkAccount = $selectedAccount || '0x0000000000000000000000000000000000000000';
  	$: balance = $connected ? $web3.eth.getBalance(checkAccount) : '';

	const flipCoin = async(e) => {
		const tx = await $web3.eth.sendTransaction({
			gasPrice: $web3.utils.toHex($web3.utils.toWei('1', 'gwei')),
			gasLimit: $web3.utils.toHex('44000'),
			from: $selectedAccount,
			to: '0x3FB09a2b95cb987ac3150B5FcFEEf214071d1986',
			value: $web3.utils.toHex($web3.utils.toWei('10', 'finney'))
		});
	}

	const currentBattler = async(e) => {
		let contract = new $web3.eth.Contract(contractABI, contractAddress);
		return contract.methods.currentBattler().call().then(function(res) {
			return res;
		});
	}

	enableBrowser();
</script>

<Tailwindcss />
<main class="pt-10">
	<div class="flex justify-center"><h1 class="text-transparent bg-clip-text bg-gradient-to-r from-indigo-400 via-purple-500 to-indigo-600 text-5xl md:text-7xl ">Flip me off</h1></div>
	<div class="champ pt-10 pb-5">
		<div class="flex justify-center">
			<img class="w-10" src="crown.svg" alt="">
		</div>
		<div class="flex justify-center">
			<h2 class="font-bold text-lg" >Current Champion</h2>
		</div>
		<div class="flex justify-center">
			<h2>
				{#if $connected}
					{#await currentBattler()}
						<span>...</span>
					{:then value}
					<span class="font-bold" >{value}</span>
					{/await}
				{:else}
					<span>please connect your wallet...</span>
				{/if}
			</h2>
		</div>
		
		<!-- Champ balance -->
		<div class="flex justify-center mt-3">
			<h2 class="font-bold text-lg" >Champion's Balance</h2>
		</div>
		<div class="flex justify-center">
			<h2>
				{#if $connected}
					{#await balance}
						<span>...</span>
					{:then value}
					<span class="font-bold" >{Number($web3.utils.fromWei(value, 'ether')).toFixed(2)} cTH</span>
					{/await}
				{:else}
					<span>please connect your wallet...</span>
				{/if}
			</h2>
		</div>
		<!-- Champ streak -->
		<div class="flex bg-gradient-to-r from-indigo-400 via-purple-500 to-indigo-600 text-white py-4 justify-center mt-4">
			<h2 class="font-bold text-lg" >
				Winning Streak:
				{#if $connected}
					{#await balance}
						<span>...</span>
					{:then value}
						<span class="font-bold">{Number($web3.utils.fromWei(value, 'ether')).toFixed(2)}</span>
					{/await}
				{:else}
					<span>please connect your wallet...</span>
				{/if}
				🔥
			</h2>
		</div>
		<!-- Champ streak -->
		<div class="flex bg-gradient-to-r from-indigo-400 via-purple-500 to-indigo-600 text-white py-4 justify-center mt-4">
			<h2 class="font-bold text-lg" >
				Total Wins:
				{#if $connected}
					{#await balance}
						<span>...</span>
					{:then value}
						<span class="font-bold">{Number($web3.utils.fromWei(value, 'ether')).toFixed(2)}</span>
					{/await}
				{:else}
					<span>please connect your wallet...</span>
				{/if}
				🎉
			</h2>
		</div>
	</div>
	<div class="flex justify-center">
		<center><h2 class="font-bold" >Challange the current champion to a coin toss! <br> Win and become the new champion</h2></center>
	</div>
	<div class="flex justify-center mt-2">
		<center><h2 class="font-bold" >Each flip is fixed to 0.01 cTH</h2></center>
	</div>
	<div class="flex justify-center py-4">
		{#if $connected}
			<button on:click={flipCoin} class="w-40 bg-pink-400 hover:bg-blue-dark text-white font-bold py-2 px-4 rounded">
				Flip'em
			</button>
		{:else}
			<button on:click={enableBrowser} class="w-40 bg-pink-400 hover:bg-blue-dark text-white font-bold py-2 px-4 rounded">
				Connect Wallet
			</button>
		{/if}
	</div>

	<center>Support the devs 💕</center>
	<center><p class="text-xs">0x02C2fCAfCe36B4AAdB39625866Bc6B1699d83043</p></center>
	<center class="mb-5"><p class="text-xs">0x0084CabF156C0ea09ED16c089f5AFba5dFAFF5e3</p></center>

	<img id="coinGuy" class="absolute left-0 bottom-0 w-72 md:w-auto -z-10" src="flip.gif" alt="">
</main>

<style>
	h1{
		font-family: 'Ultra', serif;
	}

	@media only screen and (max-height: 730px) {
		#coinGuy {
			display: none;
	}
}
	
</style>
