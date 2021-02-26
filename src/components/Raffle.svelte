<script>
    import WrongNetwork from './WrongNetwork.svelte';
    import { ethStore, chainId, web3, selectedAccount, connected } from 'svelte-web3';
    import { tweened } from 'svelte/motion';
    import Button from './Button.svelte';

    let wrongNetwork = false;

	// Loading Spinners
	let connectWalletLoading = false;
	let transactionLoading = false;

    let price = 10000000;
    let numberOfTickets = 1;

    import abi from '../_abi.js';
    export let contractAddress = '0x27dc128236C4383DbA81259EF0Ae621Abd06e4cd';
	const contractABI = abi;

    let contractBalance = tweened(0.0);
    let ticketLeft = tweened(0);
    let winners = tweened(0);

    $: checkAccount = $selectedAccount || '0x0000000000000000000000000000000000000069'
    $: balance = $connected ? $web3.eth.getBalance(checkAccount) : '';

    const ticketsRemaining = async(e) => {
		let contract = new $web3.eth.Contract(contractABI, contractAddress);
		return contract.methods.getTicketsRemaining().call().then(function(res) {
			return res;
		});
	}
    const getWinners = async(e) => {
		let contract = new $web3.eth.Contract(contractABI, contractAddress);
		return contract.methods.getWinners().call().then(function(res) {
			return res;
		});
	}
    
    const enableBrowser = async () => {
		connectWalletLoading = true;
		await ethStore.setBrowserProvider();
		if($web3.eth && await $web3.eth.net.getId() !== 777) {
            console.log("Wrong chain - Change MetaMask to cheapETH (cheapeth.org)");
			wrongNetwork = true;
			return;
        }
        
        let bal = await $web3.eth.getBalance(contractAddress);
        contractBalance.set(parseFloat($web3.utils.fromWei(bal.toString(), 'ether')));
        console.log(await $web3.eth.getBalance(contractAddress))

        let contract = new $web3.eth.Contract(contractABI, contractAddress, { from: $selectedAccount });

        contract.events.JoinEvent({
            filter: {myIndexedParam: [20,23], myOtherIndexedParam: '0x123456789...'}, // Using an array means OR: e.g. 20 or 23
            fromBlock: 0
        }, function(error, event){ console.log(event); })
        .on('data', async function (event){
            console.log(event); // same results as the optional callback above
            let bal = await $web3.eth.getBalance(contractAddress);
            contractBalance.set(parseFloat($web3.utils.fromWei(bal.toString(), 'ether')));
        })
        .on('changed', function(event){
            // remove event from local database
        })
        .on('error', console.error);
    }

        $: if (checkAccount) {
            set();
        }

	async function set() {
		let newticketLeft = $connected ? await ticketsRemaining() : 0;
		newticketLeft = parseInt(newticketLeft);
		ticketLeft.set($connected ? newticketLeft !== ticketLeft ? newticketLeft : ticketLeft : 0);

		let newwinners = $connected ? await getWinners() : 0;
		newwinners = parseInt(newwinners);
		winners.set($connected ? newwinners !== winners ? newwinners : winners : 0);
	}

    const handleSubmit = async (e) => {
        transactionLoading = true; // Start spinner
        let contract = new $web3.eth.Contract(contractABI, contractAddress, { from: $selectedAccount });
        let val = price * numberOfTickets;
        console.log(val)
        console.log(val.toString())
        return new Promise((resolve, reject) => {
			contract.methods.joinRaffle().send({
				gasPrice: $web3.utils.toHex($web3.utils.toWei('1', 'gwei')),
				gasLimit: $web3.utils.toHex(115000),
				from: $selectedAccount,
				to: contractAddress,
				value: $web3.utils.toHex($web3.utils.toWei(val.toString(), 'gwei'))
			}).on('confirmation', async (confirmationNumber) => {
				if(confirmationNumber === 0) {
					transactionLoading = false; // Stop spinner
					console.log("The transaction has confirmed!");
				}
				transactionLoading = false; // Stop spinner
			}).on('receipt', function(e){
                console.log(e)
            }).on('error', (error) => {
				console.log("The transaction failed!");
				console.log(error);
				transactionLoading = false; // Stop spinner
			});
		})
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
                {$contractBalance.toFixed(2)} cTH
            {:else}
                please connect your wallet...
            {/if}
        </h2>
    </div>
    <div class="flex bg-gradient-to-r from-indigo-400 via-purple-500 w-full to-indigo-600 text-white py-4 justify-center mt-4">
        <h2 class="italic font-extrabold text-xl" >
            {#if $connected}
                {$ticketLeft.toFixed(0)} Tickets Remaining
            {:else}
                please connect your wallet...
            {/if}
        </h2>
    </div>
    <form on:submit|preventDefault={handleSubmit} class="flex mt-5 flex-col items-center justify-center" >
        <div class="font-bold">
            <span>I want to buy</span>
            <input bind:value={numberOfTickets} type="number" placeholder="100" min="1" step="1" class="mt-1 p-3 pl-5 text-lg appearance-none placeholder-black font-bold focus:ring-indigo-500 leading-none text-center bg-pink-300 focus:border-indigo-500 w-40 shadow-sm border-gray-300 rounded-md">
            <span>raffle tickets</span>
        </div>
        <span> 0.01cTH per ticket</span>
        <div class="submit mt-3">
            {#if $connected}
                <Button on:click={handleSubmit} loading={transactionLoading}> Enter Raffle </Button>
            {:else}
                <Button on:click={enableBrowser} loading={connectWalletLoading}>
                    Connect Wallet
                </Button>
            {/if}
        </div>
    </form>
    <h3 class="font-bold text-xl mt-4">Latest Winners</h3>
    <div class="winner font-bold">
        {$winners}
        <span>0x0f8C4EC2f0D68602E79571199a79F52F70ae75a4</span><span class="mx-4">➡</span><span>291cTH 🎉</span>
    </div>
</div>
