<script>
	import WrongNetwork from './WrongNetwork.svelte';
	import Button from './Button.svelte';
	import { ethStore, chainId, web3, selectedAccount, connected } from 'svelte-web3';
	import { tweened } from 'svelte/motion';
	import { onMount } from 'svelte';
	import { scale } from 'svelte/transition';
	import { quintOut } from 'svelte/easing';
    
    export let contractAddress = '0xeDc5BC933d49a85f510CcF0D7440214bc1e6747d';
    export let price = '10';

	// Loading Spinners
	let connectWalletLoading = false;
	let flipLoading = false;

	let wrongNetwork = false;

	let winState = 0;

	const contractABI = [{"inputs":[],"name":"flip","outputs":[],"stateMutability":"payable","type":"function"},{"inputs":[],"stateMutability":"payable","type":"constructor"},{"inputs":[],"name":"betAmount","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"currentBattler","outputs":[{"internalType":"address payable","name":"","type":"address"}],"stateMutability":"view","type":"function"},{"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"losses","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"streak","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"wins","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"}];

	let currentBattler = '0x0000000000000000000000000000000000000069'; 
	let totalWins = tweened(0); 
	let totalLosses = tweened(0);
	let currentStreak = tweened(0);
	let championsBalance = tweened(0.0);

	const enableBrowser = async () => {
		connectWalletLoading = true;
		await ethStore.setBrowserProvider();

		if($web3.eth && await $web3.eth.net.getId() !== 777) {
            console.log("Wrong chain - Change MetaMask to cheapETH (cheapeth.org)");
			wrongNetwork = true;
			return;
        }
		
		await set();
		connectWalletLoading = false;

		$web3.eth.subscribe('newBlockHeaders', async function(error, result) {
			console.log("New Block");
			await set();
		})
	}

	$: checkAccount = $selectedAccount || '0x0000000000000000000000000000000000000069';
	
	const flipCoin = async(e) => {

		winState = 0;

		let contract = new $web3.eth.Contract(contractABI, contractAddress, { from: $selectedAccount });
        console.log(contractAddress);

		let gasEstimate = await contract.methods.flip().estimateGas({
			from: $selectedAccount,
			to: contractAddress,
			value: $web3.utils.toHex($web3.utils.toWei(price, 'finney'))
		}).then(function (res) {
			return res;
		});
        
		flipLoading = true; // Start spinner
		return new Promise((resolve, reject) => {
			contract.methods.flip().send({
				gasPrice: $web3.utils.toHex($web3.utils.toWei('1', 'gwei')),
				gasLimit: $web3.utils.toHex(115000),
				from: $selectedAccount,
				to: contractAddress,
				value: $web3.utils.toHex($web3.utils.toWei(price, 'finney'))
			}).on('confirmation', async (confirmationNumber) => {
				if(confirmationNumber === 0) {
					flipLoading = false; // Stop spinner
					console.log("The transaction has confirmed!");
					console.log("Now check if winner...");

					await set();

					console.log(currentBattler);
					console.log($selectedAccount);

					if(currentBattler.toLowerCase() === $selectedAccount.toLowerCase()) {
						console.log("You are the winner, congratulations!");
						winState = 1;
					} else {
						console.log("You lost :(");
						winState = -1;
					} 
				}
				flipLoading = false; // Stop spinner
			}).on('error', (error) => {
				console.log("The transaction failed!");
				flipLoading = false; // Stop spinner
			});
		})
	}

	const battler = async(e) => {
		let contract = new $web3.eth.Contract(contractABI, contractAddress);
		return contract.methods.currentBattler().call().then(function(res) {
			return res;
		});
	}

	const losses = async(e) => {
		let contract = new $web3.eth.Contract(contractABI, contractAddress);
		return contract.methods.losses($selectedAccount).call().then(function(res) {
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

	$: if (checkAccount) {
		set();
	}

	async function set() {
		let newLosses = $connected ? await losses() : 0;
		let newWins = $connected ? await wins() : 0;
		let newStreak = $connected ? await streak() : 0;
		let newBattler = $connected ? await battler() : 0;
		let newBalance = $connected ? await $web3.eth.getBalance(await newBattler) : 0.0;

		newWins = parseInt(newWins);
		newLosses = parseInt(newLosses);
		newStreak = parseInt(newStreak);

		currentBattler = $connected ? newBattler !== currentBattler ? newBattler : currentBattler : '0x0000000000000000000000000000000000000069';
		totalLosses.set($connected ? newLosses !== $totalLosses ? newLosses : $totalLosses : 0);
		totalWins.set($connected ? newWins !== $totalWins ? newWins : $totalWins : 0);
		currentStreak.set($connected ? newStreak !== $currentStreak ? newStreak : $currentStreak : 0);

		let newBalanceEther = await $web3.utils.fromWei((newBalance).toString(), 'ether');
		championsBalance.set($connected ? parseFloat(newBalanceEther) !== $championsBalance ? parseFloat(newBalanceEther) : $championsBalance : 0.0);
	}
	
	onMount(async () => {
		await set();
	});

</script>
	{#if wrongNetwork}
		<WrongNetwork />
	{/if}
		<div class="flex justify-center mt-6">
			<img class="w-10 h-10" src="crown.svg" alt="">
		</div>
		<div class="flex justify-center">
			<h2 class="font-bold text-lg" >Current Champion
				{#if $connected}
					{#if currentBattler.toLowerCase() === checkAccount.toLowerCase() }
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
			<h2 class="font-bold text-lg">Champion's Balance</h2>
		</div>
		<div class="flex justify-center">
			<h2>
				{#if $connected}
					<span class="font-bold">{$championsBalance.toFixed(2)} cTH</span>
				{:else}
					<span>please connect your wallet...</span>
				{/if}
			</h2>
		</div>
		<!-- Champ streak -->
		<div class="flex bg-gradient-to-r from-indigo-400 via-purple-500 to-indigo-600 text-white py-4 justify-center mt-4">
			<h2 class="font-bold text-lg">
				{#if $connected}
					<span class="font-bold">Your Winning Streak: {$currentStreak.toFixed(0)} 🔥</span>
				{:else}
					<span>please connect your wallet...</span>
				{/if}
			</h2>
		</div>
		<!-- Champ total wins -->
		<div class="flex bg-gradient-to-r from-indigo-400 via-purple-500 to-indigo-600 text-white py-4 justify-center mt-4">
			<h2 class="font-bold text-lg" >
				{#if $connected}
					<span class="font-bold">Your Total Wins: {$totalWins.toFixed(0)} 🎉</span>
					<span class="font-bold">Your Total Losses: {$totalLosses.toFixed(0)} 😢</span>
				{:else}
					<span>please connect your wallet...</span>
				{/if}
				
			</h2>
		</div>
	<!-- Game outcome -->
	<div class="flex justify-center font-bold mt-2">
		Game Outcome
	</div>
	<div class="flex justify-center mt-1">
		<div class="p-6 leading-none text-center bg-pink-300 shadow-inner rounded">

				{#if winState == 1}
					👍
				{:else if winState == -1}
					👎
				{:else}
					👍 | 👎
				{/if}

			{#if winState}
			<div class="outcome mt-3 font-bold leading-none">
				{#if winState == 1}
					You Won!
				{:else if winState == -1}
					You Lost!
				{/if}
			</div>
			{/if}
		</div>
		
	</div>
	<div class="flex justify-center mt-3">
		{#if $connected}
			{#if currentBattler.toLowerCase() === checkAccount.toLowerCase() }
				<Button on:click={flipCoin}  loading={flipLoading} disabled={true}>
					Flip'em
				</Button>
				{:else}
				<Button on:click={flipCoin}  loading={flipLoading}>
					Flip'em
				</Button>
			{/if}
		{:else}
			<Button on:click={enableBrowser} loading={connectWalletLoading}>
				Connect Wallet
			</Button>
		{/if}
	</div>
	<div class="flex justify-center mb-4 mt-3">
		<small class="" >
			{#if $connected}
				{#if currentBattler.toLowerCase() === checkAccount.toLowerCase() }
					<span>You're the current champion! Now just wait and let the money roll in 😎</span>
				{/if}
			{/if}
		</small>
	</div>
	<div class="flex justify-center">
		<center><h2 class="font-bold" >Challange the current champion to a coin toss! <br> Win and become the new champion.</h2></center>
	</div>
	<div class="flex justify-center mt-3">
		<center><h2 class="font-bold" >Once you have become the champion every time you <br> win against your foes, you keep their cTH.</h2></center>
	</div>
	<div class="donate mt-11">
		<center>Support the devs 💕</center>
		<center><p class="text-xs">0x02C2fCAfCe36B4AAdB39625866Bc6B1699d83043</p></center>
		<center class="mb-5"><p class="text-xs">0x0084CabF156C0ea09ED16c089f5AFba5dFAFF5e3</p></center>
	</div>

	


	