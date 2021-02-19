<script>
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

<style>
</style>

{#if $connected}
	<p>Account: {checkAccount}</p>
	<p>Balance: 
		{#await balance}
			<span>...</span>
		{:then value}
			{value}
		{/await}
	</p>

	<p>Current Winner: 
		{#await currentBattler()}
			<span>...</span>
		{:then value}
			{value}
		{/await}
	</p>

	<button on:click="{flipCoin}">Flip a Coin</button>
{/if}