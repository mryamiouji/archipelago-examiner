<template>
	<h1>Archipelago Examiner</h1>
	<a href="https://github.com/mryamiouji/archipelago-examiner" target="_blank">Github</a>

	<div class="flexcol">
		<input type="text" placeholder="game" v-model="game" />
		<input type="text" placeholder="host" v-model="host" />
		<input type="password" placeholder="password" v-model="password" />
		<input type="text" placeholder="player name (case sensitive)" v-model="playerName" />
		<button v-if="!apConnectedState" @click="connect">Connect</button>
		<button v-if="apConnectedState" @click="disconnect">Disconnect</button>

		<div v-if="apConnectedState">
			<h2>Connected to Archipelago</h2>
			<p>Game: {{ game }}</p>
			<p>Host: {{ host }}</p>
			<p>Player Name: {{ playerName }}</p>

			<div>
				<h3>Connection Package</h3>
				<pre>{{ apConnectionPackage }}</pre>
			</div>

			<div class="flexrow">
				<div>
					<h3>Items</h3>
					<ul>
						<li v-for="(name, id) in apItemList" :key="id" style="list-style-type: none">{{ id }}: {{ name }}</li>
					</ul>
				</div>

				<div>
					<h3>Locations</h3>
					<ul>
						<li v-for="(name, id) in apLocationList" :key="id" style="list-style-type: none">{{ id }}: {{ name }}</li>
					</ul>
				</div>
			</div>
		</div>

		<div v-if="!apConnectedState">
			<h2>Not connected to Archipelago</h2>
		</div>
	</div>
</template>

<script setup>
import { ref } from 'vue';
import { Client } from 'archipelago.js';

const host = ref('localhost:38281');
const version = ref({
	major: 0,
	minor: 5,
	build: 1
});
const game = ref('');
const playerName = ref('');
const password = ref('');

const apConnectedState = ref(false);

const client = new Client();

const apItemList = ref([]);
const apLocationList = ref([]);

const apConnectionPackage = ref([]);

const connect = () => {
	client
		.login(host.value, playerName.value, game.value, {
			password: '',
			version: version.value,
			tags: ['Tracker']
		})
		.then((configs) => {
			apConnectedState.value = true;

			console.info('AP configs', configs);
			apConnectionPackage.value = configs;

			console.info('AP client package', client.package.findPackage(client.game));
			apItemList.value = client.package.findPackage(client.game).reverseItemTable;
			apLocationList.value = client.package.findPackage(client.game).reverseLocationTable;

			console.log('Locations received', client.room.checkedLocations);

			client.room.on('locationsChecked', (locations) => {
				console.log('Locations checked:', locations);
			});

			client.items.on('itemsReceived', (items) => {
				console.log('Items received:', items);
			});
		})
		.catch((error) => {
			apConnectedState.value = false;
			console.error('Failed to connect:', error);
		});
};

const disconnect = () => {
	client.disconnect();
	apConnectedState.value = false;
	console.log('Disconnected');
};
</script>
