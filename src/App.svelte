<script>
	import Tailwindcss from './Tailwindcss.svelte';
	import { ethStore, web3, selectedAccount, connected } from 'svelte-web3';
	import { onMount } from 'svelte';

	const contractABI = [{"inputs":[],"name":"flip","outputs":[],"stateMutability":"payable","type":"function"},{"inputs":[],"stateMutability":"payable","type":"constructor"},{"inputs":[],"name":"betAmount","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"currentBattler","outputs":[{"internalType":"address payable","name":"","type":"address"}],"stateMutability":"view","type":"function"},{"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"streak","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"wins","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"}];
	const contractAddress = '0xA0BdC5691d808D3F58A938503Da997be77C85822';

	const enableBrowser = async () => {
		await ethStore.setBrowserProvider();

		$web3.eth.subscribe('newBlockHeaders', function(error, result) {
			currentBattler = $connected ? battler() : '';
			totalWins = $connected ? wins() : '0';
			currentStreak = $connected ? streak() : '0';
		})
	}

	let currentBattler, totalWins, currentStreak;

	$: checkAccount = $selectedAccount || '0x0000000000000000000000000000000000000000';
	$: currentBattler = $connected ? battler() : '';
	$: totalWins = $connected ? wins() : '0';
	$: currentStreak = $connected ? streak() : '0';

	const flipCoin = async(e) => {
		let contract = new $web3.eth.Contract(contractABI, contractAddress, { from: $selectedAccount });

		let gasEstimate = await contract.methods.flip().estimateGas({
			from: $selectedAccount,
			to: contractAddress,
			value: $web3.utils.toHex($web3.utils.toWei('10', 'finney'))
		}).then(function (res) {
			return res;
		});

		return new Promise((resolve, reject) => {
			contract.methods.flip().send({
				gasPrice: $web3.utils.toHex($web3.utils.toWei('1', 'gwei')),
				gasLimit: $web3.utils.toHex(gasEstimate),
				from: $selectedAccount,
				to: contractAddress,
				value: $web3.utils.toHex($web3.utils.toWei('10', 'finney'))
			}).on('confirmation', async (confirmationNumber) => {
				if(confirmationNumber === 0) {
					console.log("The transaction has confirmed!");
					console.log("Now check if winner...");

					currentBattler = $connected ? await battler() : '';
					totalWins = $connected ? await wins() : '0';
					currentStreak = $connected ? await streak() : '0';

					console.log(currentBattler);
					console.log($selectedAccount);

					if(currentBattler.toLowerCase() === $selectedAccount.toLowerCase()) {
						console.log("You are the winner, congratulations!");
					} else {
						console.log("You lost :(");
					} 
				}
			}).on('error', (error) => {
				console.log("The transaction failed!");
			});
		})
	}

	const battler = async(e) => {
		let contract = new $web3.eth.Contract(contractABI, contractAddress);
		return contract.methods.currentBattler().call().then(function(res) {
			return res;
		});
	}

	const wins = async(e) => {
		let contract = new $web3.eth.Contract(contractABI, contractAddress);
		return contract.methods.wins($selectedAccount).call().then(function(res) {
			return res;
		});
	}

	const streak = async(e) => {
		let contract = new $web3.eth.Contract(contractABI, contractAddress);
		return contract.methods.streak($selectedAccount).call().then(function(res) {
			return res;
		});
	}

	onMount(async () => {

	});


	$: if (checkAccount) {
		currentBattler = $connected ? battler() : '';
		totalWins = $connected ? wins() : '0';
		currentStreak = $connected ? streak() : '0';
	}
	
</script>

<Tailwindcss />
<main class="pt-10">
	<div class="flex justify-center"><h1 class="text-transparent bg-clip-text bg-gradient-to-r from-indigo-400 via-purple-500 to-indigo-600 text-5xl md:text-7xl ">Flip me off</h1></div>
	<div class="flex justify-center mt-1">
		<h2><a class="text-blue-700" href="https://cheapeth.org/">cheapETH's</a>
			number one coin flip battle ground!
		</h2>
	</div>
	<div class="champ pt-6 pb-5">
		<div class="flex justify-center">
			<img class="w-10" src="crown.svg" alt="">
		</div>
		<div class="flex justify-center">
			<h2 class="font-bold text-lg" >Current Champion
				{#if $connected}
					{#if currentBattler.toLowerCase === $selectedAccount.toLowerCase }
						<span>(it's you)</span>
					{/if}
				{/if}
			</h2>
		</div>
		<div class="flex justify-center">
			<h2>
				{#if $connected}
					{#await currentBattler}
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
					{#await $web3.eth.getBalance(checkAccount)}
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
				{#if $connected}
					{#await currentStreak}
						<span>...</span>
					{:then value}
						<span class="font-bold">Your Winning Streak: {value}</span>
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
				{#if $connected}
					{#await totalWins}
						<span>...</span>
					{:then value}
						<span class="font-bold">Your Total Wins: {value}</span>
					{/await}
				{:else}
					<span>please connect your wallet...</span>
				{/if}
				🎉
			</h2>
		</div>
	</div>
	<div class="flex justify-center">
		<center><h2 class="font-bold" >Challange the current champion to a coin toss! <br> Win and become the new champion.</h2></center>
	</div>
	<div class="flex justify-center mt-3">
		<center><h2 class="font-bold" >Once you have become the champion every time you <br> win against your foes, you keep their cTH.</h2></center>
	</div>
	<div class="flex justify-center mt-6">
		<center><h2 class="" >Each flip is fixed to 0.01 cTH</h2></center>
	</div>
	<div class="flex justify-center mt-3 mb-4">
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
	<img class="absolute left-0 bottom-0 w-72 md:w-auto -z-10" src="flip.gif" alt="">

</main>

<style>
	h1{
		font-family: 'Ultra', serif;
	}
</style>
