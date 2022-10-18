<script setup>
import ColorThief from "colorthief/dist/color-thief"
const address = new URLSearchParams(window.location.search).get("address")
const nfts = await fetch(
	`https://host.airland.io:5443/nfts/getNFTByAddress?walletAddress=${address}&status=1`
).then((r) => r.json())
</script>

<template>
	<div
		class="large-nft-container"
		:style="{ '--main-bg-color': background, '--main-text-color': text }"
	>
		<div class="square glass"></div>
		<button v-if="current > 0" class="nft-arrow" @click="back">
			<i class="fa fa-chevron-left" />
		</button>
		<section class="center--h">
			<div class="center--v">
				<transition mode="out-in">
					<div class="nft-img-container" :key="nft.id">
						<div class="nft-name glass" v-show="hover">
							{{ nft.name }}
						</div>
						<img
							@mouseenter="hover = true"
							@mouseleave="hover = false"
							class="nft-img"
							ref="img"
							:src="nft.image_url"
							:alt="nft.description"
							crossorigin="anonymous"
						/>
						<div class="nft-artist glass" v-show="hover">
							{{ artist?.value }}
						</div>
					</div>
				</transition>
			</div>
		</section>
		<button
			v-if="current < total - 1"
			class="nft-arrow nft-next"
			@click="forward"
		>
			<i class="fa fa-chevron-right" />
		</button>
	</div>
</template>

<script>
const colorThief = new ColorThief()
export default {
	name: "NFTLarge",
	data() {
		return {
			current: 0,
			color: [255, 255, 255],
			hover: false,
		}
	},
	created() {
		window.addEventListener("keydown", this.onKeydown)
	},
	mounted() {
		this.getColor()
	},
	computed: {
		total() {
			return this.nfts?.data.assets.filter((x) => x.image_url).length
		},
		nft() {
			return this.nfts?.data.assets.filter((x) => x.image_url)[this.current]
		},
		background() {
			return `rgba(${this.color[0]},${this.color[1]},${this.color[2]})`
		},
		text() {
			const luminance =
				0.299 * this.color[0] + 0.587 * this.color[1] + 0.114 * this.color[2]
			return luminance > 50 ? " black" : " white"
		},
		artist() {
			return this.nft.traits.find((x) => x.trait_type === "artist")
		},
	},
	methods: {
		back() {
			if (this.current === 0) return
			this.current = this.current - 1
			setTimeout(() => this.getColor(), 550)
		},
		forward() {
			if (this.current === this.total - 1) return
			this.current = this.current + 1
			setTimeout(() => this.getColor(), 550)
		},
		getColor() {
			const img = this.$refs.img

			if (img?.complete) {
				this.color = colorThief.getColor(img)
			} else {
				img.addEventListener("load", () => {
					this.color = colorThief.getColor(img)
				})
			}
		},
		onKeydown(e) {
			switch (e.key) {
				case "ArrowLeft":
					this.back()
					break
				case "ArrowRight":
					this.forward()
					break
			}
		},
	},
}
</script>

<style scoped>
.glass {
	background: rgba(255, 255, 255, 0.2);
	border-radius: 8px;
	box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1);
	backdrop-filter: blur(5px);
	-webkit-backdrop-filter: blur(5px);
}

.large-nft-container {
	background: var(--main-bg-color);
	height: 100vh;
	width: 100vw;
}

.square {
	width: calc(var(--app-padding) * 6);
	height: calc(var(--app-padding) * 6);
	position: absolute;
	top: var(--app-padding);
	left: var(--app-padding);
	line-height: 0;
	font-size: 2em;
}

.nft-arrow {
	z-index: 1;
	position: absolute;
	top: 50%;
	padding: var(--app-padding);
	margin: var(--app-margin);
}

.nft-next {
	right: 0%;
}

.center--h {
	display: flex;
	justify-content: center;
	align-items: center;
	height: 100%;
}

.center--v {
	display: flex;
	flex-direction: row;
}

.nft-img-container {
	position: relative;
	display: inline;
	box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1);
	border-radius: 10px;
}

.nft-img {
	width: 45vw;
	max-height: 85vh;
	border-radius: 10px;
	border: 4px solid var(--main-text-color);
}

button {
	background: transparent;
	border: none;
	font-size: 2em;
	color: var(--main-text-color);
	opacity: 0.2;
	backdrop-filter: blur(5px);
	-webkit-backdrop-filter: blur(5px);
	cursor: pointer;
}

.nft-name {
	position: absolute;
	top: var(--app-padding);
	left: var(--app-padding);
	color: var(--main-text-color);
	font-size: 2em;
	padding: var(--app-padding);
	padding: var(--app-padding);
	margin: var(--app-padding);
}

.nft-artist {
	position: absolute;
	bottom: var(--app-padding);
	left: var(--app-padding);
	color: var(--main-text-color);
	font-size: 2em;
	padding: var(--app-padding);
	margin: var(--app-padding);
}

.v-enter-active,
.v-leave-active {
	transition: opacity 0.5s ease;
}

.v-enter-from,
.v-leave-to {
	opacity: 0;
}
</style>
