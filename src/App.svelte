<script>
	import Tailwindcss from './Tailwindcss.svelte';
	import { Tabs, TabList, TabPanel, Tab } from './tabs.js';
	import Flipper from './components/Flipper.svelte';
	import Raffle from './components/Rafflemyown.svelte';
</script>

<Tailwindcss />
<main class="pt-10">
	<div class="flex justify-center"><h1 class="text-transparent bg-clip-text bg-gradient-to-r from-indigo-400 via-purple-500 to-indigo-600 text-5xl md:text-7xl ">Flip me off</h1></div>
	<div class="flex justify-center mt-1">
		<h2><a class="text-blue-700" href="https://cheapeth.org/">cheapETH's</a>
			number one coin flip battle ground!
		</h2>
	</div>

	<div class="">
		<!-- Tabs -->
		<Tabs>
			<TabList>
				<Tab>0.01 cTH</Tab>
				<Tab>0.1 cTH</Tab>
				<Tab>1 cTH</Tab>
				<Tab lable={true} >Raffle</Tab>
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

			<!-- Raffle -->
			<TabPanel>
				<Raffle />
			</TabPanel>
		</Tabs>
	</div>
	
	<img class="hidden md:block fixed left-0 bottom-0 w-72 lg:w-auto z-10" src="flip.gif" alt="">
</main>

<style>
	h1{
		font-family: 'Ultra', serif;
	}
</style>
