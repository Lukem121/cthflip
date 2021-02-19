<script>
	export let name;
	import { ethStore, web3, selectedAccount, connected, chainId } from 'svelte-web3';
	import { onMount } from 'svelte';
	import { tweened } from 'svelte/motion';

	const contractABI = [{"inputs":[],"name":"flip","outputs":[],"stateMutability":"payable","type":"function"},{"inputs":[],"stateMutability":"payable","type":"constructor"},{"inputs":[],"name":"betAmount","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"currentBattler","outputs":[{"internalType":"address payable","name":"","type":"address"}],"stateMutability":"view","type":"function"},{"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"losses","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"streak","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"wins","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"}];
	const contractAddress = '0xe8e164e1CfB5Cc618111651C860BB54EAfC360A2';

	let currentBattler = ''; 
	let totalWins = tweened(0); 
	let totalLosses = tweened(0);
	let currentStreak = tweened(0);

	const enableBrowser = async () => {
		await ethStore.setBrowserProvider();

		$web3.eth.subscribe('newBlockHeaders', function(error, result) {
			console.log("New Block");
			set();
		})
	}

	$: checkAccount = $selectedAccount || '0x0000000000000000000000000000000000000000';
	/*$: currentBattler.set($connected ? battler() : '0x0000000000000000000000000000000000000000');
	$: totalWins.set($connected ? wins() : '0');
	$: currentStreak.set($connected ? streak() : '0');*/

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

					await set();

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

	onMount(async () => {

	});


	$: if (checkAccount) {
		set();
		if($web3.eth && $chainId !== "0x309") {
			console.log("Wrong chain - Change MetaMask to cheapETH (cheapeth.org)");
		}
	}

	async function set() {
		let newLosses = $connected ? await losses() : 0;
		let newWins = $connected ? await wins() : 0;
		let newStreak = $connected ? await streak() : 0;
		let newBattler = $connected ? await battler() : 0;

		newLosses = parseInt(newLosses);
		newWins = parseInt(newWins);
		newStreak = parseInt(newStreak);

		currentBattler = $connected ? newBattler !== currentBattler ? newBattler : currentBattler : 0;
		totalLosses.set($connected ? newLosses !== $totalLosses ? newLosses : $totalLosses : 0);
		totalWins.set($connected ? newWins !== $totalWins ? newWins : $totalWins : 0);
		currentStreak.set($connected ? newStreak !== $currentStreak ? newStreak : $currentStreak : 0);
	}
	
	enableBrowser();
</script>

<style>
</style>

{#if $connected && $web3.eth.getBalance(checkAccount)}
	<p>Account: {checkAccount}</p>
	<p>Balance: 
		{#await $web3.eth.getBalance(checkAccount)}
			<span>...</span>
		{:then value}
			{Number($web3.utils.fromWei(value, 'ether')).toFixed(2)}
		{/await}
	</p>

	<p>Current Winner: {currentBattler}</p>

	<p>My Total Wins: {$totalWins.toFixed(0)}</p>

	<p>My Total Losses: {$totalLosses.toFixed(0)}</p>

	<p>My Current Streak: {$currentStreak.toFixed(0)}</p>
	

	<button on:click="{flipCoin}">Flip a Coin</button>
{/if}