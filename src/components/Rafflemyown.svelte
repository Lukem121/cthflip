<script>
    import WrongNetwork from './WrongNetwork.svelte';
    import Button from './Button.svelte';
    import { ethStore, chainId, web3, selectedAccount, connected } from 'svelte-web3';


    import abi from '../_abi.js';
    let contractAddress = '0xCf234C91b8e592C320ae7f34099fC9128d5D94f6'; 

    // Display modal if user is on wrong network
    let wrongNetwork = false; // A1 201.489 A2 300.023

	// Loading Spinners
	let connectWalletLoading = false;
	let transactionLoading = false;

    let price = 10000000;
    let numberOfTickets = 100;

    // UI Varibles
    let ticketRemaing = 0;
    let contractBalance = 0;
    let numberOfPlayers = 0;
    let timeRemaining = 0;
    let raffleWinners = [];

    // Enables the connection to the MetaMask extention
    const enableBrowser = async () => {
        connectWalletLoading = true;

		await ethStore.setBrowserProvider();
        // If user is on wrong network check
		if($web3.eth && await $web3.eth.net.getId() !== 777) {
            console.log("Wrong chain - Change MetaMask to cheapETH (cheapeth.org)");
			wrongNetwork = true;
        }

        connectWalletLoading = true;

        await updateValues();

        // Updates any values that may have changed in the new block
        $web3.eth.subscribe('newBlockHeaders', async function(error, result) {
			console.log("New Block");
			await updateValues()
		})        
    }

    const joinRaffle = async () => {
        let contract = new $web3.eth.Contract(abi, contractAddress, { from: $selectedAccount });
        await contract.methods.joinRaffle().send({
            from: $selectedAccount,
            gasPrice: $web3.utils.toHex($web3.utils.toWei('1', 'gwei')),
            gasLimit: $web3.utils.toHex(115000),
            value: $web3.utils.toWei((price * numberOfTickets).toString(), 'gwei')
        })
        .then( (receipt) => {
            console.log(receipt);
        });
    }

    const updateValues = async () => {
        ticketRemaing = await getTickets();
        contractBalance = await getContractBalance();
        timeRemaining = await getTimeRemaining();
        numberOfPlayers = await getNumberOfPlayers();
        raffleWinners = await getRaffleWinners();
        if(raffleWinners.length > 10){
            raffleWinners = raffleWinners.slice(Math.max(raffleWinners.length - 10, 1));
        }
        raffleWinners = raffleWinners.slice().reverse();

    }

    const getNumberOfPlayers = async(e) => {
		let contract = new $web3.eth.Contract(abi, contractAddress);
		return contract.methods.numberOfPlayers().call().then(function(res) {
			return res;
		});
	}
    const getTimeRemaining = async(e) => {
		let contract = new $web3.eth.Contract(abi, contractAddress);
		return contract.methods.getTimeRemaining().call().then(function(res) {
			return res;
		});
	}
    const getTickets = async(e) => {
		let contract = new $web3.eth.Contract(abi, contractAddress);
		return contract.methods.getTickets().call().then(function(res) {
			return res;
		});
	}
    const getContractBalance = async(e) => {
		let contract = new $web3.eth.Contract(abi, contractAddress);
		return contract.methods.getBal().call().then(function(res) {
			return res;
		});
	}
    const getRaffleWinners = async(e) => {
		let contract = new $web3.eth.Contract(abi, contractAddress);
		return contract.methods.getWinners().call().then(function(res) {
			return res;
		});
	}


</script>

{#if wrongNetwork}
	<WrongNetwork />
{/if}
<div class="flex flex-col items-center justify-center mt-6">
    <img class="w-56" src="/moneycat.png" alt="">
    <h2 class="font-bold text-4xl mt-2" >Raffle Jackpot</h2>
    <div class="flex bg-gradient-to-r from-indigo-400 via-purple-500 w-full to-indigo-600 text-white py-4 justify-center mt-4">
        <h2 class="italic font-extrabold text-4xl text-yellow-300" >
            {#if $connected}
                {#await updateValues()}
                    <p>...waiting</p>
                {:then}
                    <p>{$web3.utils.fromWei((contractBalance).toString(), 'ether')} cTH</p>
                {:catch error}
                    <p style="color: red">{error.message}</p>
                {/await}
            {:else}
                please connect your wallet...
            {/if}
        </h2>
    </div>
    <div class="flex bg-gradient-to-r from-indigo-400 via-purple-500 w-full to-indigo-600 text-white py-4 justify-center mt-4">
        <h2 class="italic font-extrabold text-xl" >
            {#if $connected}
                {#await updateValues()}
                    <p>...waiting</p>
                {:then}
                    <p>{ticketRemaing} Tickets Sold</p>
                {:catch error}
                    <p style="color: red">{error.message}</p>
                {/await}
            {:else}
                please connect your wallet...
            {/if}
        </h2>
    </div>
    <div class="flex bg-gradient-to-r from-indigo-400 via-purple-500 w-full to-indigo-600 text-white py-4 justify-center mt-4">
        <h2 class="italic font-extrabold text-xl" >
            {#if $connected}
                {#await updateValues()}
                    <p>...waiting</p>
                {:then}
                    <p>
                        {#if timeRemaining > 0}
                            {timeRemaining} Blocks until draw
                        {:else if timeRemaining == 0}
                            Draw is happening
                        {:else}
                            {2 - numberOfPlayers} More players needed
                        {/if}
                    </p>
                {:catch error}
                    <p style="color: red">{error.message}</p>
                {/await}
            {:else}
                please connect your wallet...
            {/if}
        </h2>
    </div>
    <form on:submit|preventDefault={joinRaffle} class="flex mt-5 flex-col items-center justify-center" >
        <div class="font-bold">
            <span>I want to buy</span>
            <input bind:value={numberOfTickets} type="number" required min="1" step="1" class="mt-1 p-3 pl-5 text-lg appearance-none placeholder-black font-bold focus:ring-indigo-500 leading-none text-center bg-pink-300 focus:border-indigo-500 w-40 shadow-sm border-gray-300 rounded-md">
            <span>raffle tickets</span>
        </div>
        <span> 0.01cTH per ticket</span>
        <div class="submit mt-3">
            {#if $connected}
                <Button loading={transactionLoading}> Enter Raffle </Button>
            {:else}
                <Button on:click={enableBrowser} loading={connectWalletLoading}>
                    Connect Wallet
                </Button>
            {/if}
        </div>
    </form>
    <h3 class="font-bold text-xl mt-4">Latest Winners</h3>
    {#if $connected}
        {#await updateValues()}
            <p>...waiting</p>
        {:then}
            {#each raffleWinners as winner }
                <div class="font-bold">
                    <p>
                        {winner}
                        {#if winner.toLowerCase() == $selectedAccount.toLowerCase()}
                            <span class="text-pink-600"> YOU WON!! 🥳</span>
                        {/if}
                    </p>
                </div>
            {/each}
        {:catch error}
            <p style="color: red">{error.message}</p>
        {/await}
    {:else}
        please connect your wallet...
    {/if}
</div>
