<script>
	import Tailwindcss from './Tailwindcss.svelte';
	import { Tabs, TabList, TabPanel, Tab } from './tabs.js';
	import Flipper from './components/Flipper.svelte';
</script>

<Tailwindcss />
<main class="pt-10">
	<div class="flex justify-center"><h1 class="text-transparent bg-clip-text bg-gradient-to-r from-indigo-400 via-purple-500 to-indigo-600 text-5xl md:text-7xl ">Flip me off</h1></div>
	<div class="flex justify-center mt-1">
		<h2><a class="text-blue-700" href="https://cheapeth.org/">cheapETH's</a>
			number one coin flip battle ground!
		</h2>
	</div>

	<!-- Tabs -->
	<Tabs>
		<TabList>
			<Tab>0.01 cTH</Tab>
			<Tab>0.1 cTH</Tab>
			<Tab>1 cTH</Tab>
		</TabList>
	
		<!-- 0.01cTH-->
		<TabPanel>
			<Flipper price={'10'} contractAddress={'0x986855785F6aBe92a6aFfAaaAB5481247503B0c0'}/>
		</TabPanel>
		
		<!-- 0.1cTH -->
		<TabPanel>
			<Flipper price={'100'} contractAddress={'0x38cD5c965fE59904443Deab67C7015B260FA479f'}/>
		</TabPanel>
		
		<!-- 1cTH -->
		<TabPanel>
			<Flipper price={'1000'} contractAddress={'0xF153CF203410205eba4c77F98768F564Cdd827F8'}/>
		</TabPanel>
	</Tabs>
	
	<div class="donate">
		<center>Support the devs 💕</center>
		<center><p class="text-xs">0x02C2fCAfCe36B4AAdB39625866Bc6B1699d83043</p></center>
		<center class="mb-5"><p class="text-xs">0x0084CabF156C0ea09ED16c089f5AFba5dFAFF5e3</p></center>
	</div>
	<img class="absolute left-0 bottom-0 w-72 md:w-auto -z-10" src="flip.gif" alt="">

</main>

<style>
	h1{
		font-family: 'Ultra', serif;
	}

	.donate {
		position: fixed;
		left: 50%;
		bottom: 15px;
		transform: translate(-50%, -50%);
		margin: 0 auto;
	}
</style>
