<template>
	<span class="font-weight-bold">{{ track.imei }}: </span>{{ track.tramas[0].DATETIME }}
</template>

<script>
	export default {
		name: "TrackEach",
		props: {
			track: {
				type: Object,
				//required: true
			}
		}
	}
</script>