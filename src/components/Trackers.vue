<template>
	<div class="card-body">
		<h3>Track List</h3>
		<hr>
		<ul class="list-group">
			<li class="list-group-item" v-for="(track, imei) in trackers" :key="imei"><span class="font-weight-bold">{{ track.imei }}: </span>{{ track.tramas[0].DATETIME }}</li>
		</ul>
	</div>
</template>

<script>
	import io from 'socket.io-client';
	export default {
		data() {
			return {
				history_limit: 7,
				connected: false,
				user: '',
				message: '',
				trackers: [],
				socket : io('yeci.online:3006')
			}
		},
		methods: {
			sendMessage(e) {
				e.preventDefault();
				this.socket.emit('trama', this.last);
				this.message = '';
			}
		},
		mounted() {
			/* eslint-disable */
			this.socket.on('connect', () => {
				this.connected = true;
				this.socket.emit('login', {user: 'server'});
			});
			this.socket.on('tracker_list', (tracker_list) => {
				this.trackers = [];
				for (imei in tracker_list) {
					this.trackers.push({imei: imei, tramas: tracker_list[imei]});
				}
			});
			this.socket.on('trama', (data) => {
				let track = this.trackers.find(obj => {
					return obj.imei == data.IMEI
				});
				if (track) {
					track.tramas.unshift(data); // Add start
					if (track.tramas.length >= this.history_limit) {
						track.tramas.pop(); // Remove end
					}
				} else {
					this.trackers.push({imei: data.IMEI, tramas: [data]});
				}
				console.log(this.trackers);
					
			});
			this.socket.on('disconect', () => {
				this.connected = false;
			});
		}
	}
</script>