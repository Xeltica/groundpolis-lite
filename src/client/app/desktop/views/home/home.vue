<template>
<mk-ui v-hotkey.global="keymap" v-if="$store.getters.isSignedIn || $route.name != 'index'">
	<div class="wqsofvpm">
		<div class="main">
			<div class="left">
				<div class="icon" @click="goToTop">
					<img svg-inline src="../../assets/header-icon.svg"/>
				</div>
				<ul>
					<li>
						<router-link to="/" :class="{ active: $route.name == 'index' }">
							<fa :icon="$store.state.i ? 'home' : 'arrow-left'" fixed-width/>
							<span>{{ $t($store.state.i ? '@.timeline' : '@.back-to-top') }}</span>
						</router-link>
					</li>
					<li>
						<router-link to="/explore" :class="{ active: $route.name == 'explore' }">
							<fa icon="hashtag" fixed-width/>
							<span>{{ $t('@.explore') }}</span>
						</router-link>
					</li>
					<li v-if="$store.state.i">
						<router-link to="/i/notifications" :class="{ active: $route.name == 'notifications' }">
							<fa :icon="['far', 'bell']" fixed-width/>
							<span>{{ $t('@.notifications') }}</span>
						</router-link>
					</li>
					<li v-if="$store.state.i">
						<router-link to="/i/favorites" :class="{ active: $route.name == 'favorites' }">
							<fa icon="star" fixed-width/>
							<span>{{ $t('@.favorites') }}</span>
						</router-link>
					</li>
					<li v-if="$store.state.i">
						<router-link to="/i/lists" :class="{ active: $route.name == 'lists' }">
							<fa icon="list" fixed-width/>
							<span>{{ $t('@.lists') }}</span>
						</router-link>
					</li>
					<li v-if="$store.state.i">
						<router-link to="/i/drive" :class="{ active: $route.name == 'drive' }">
							<fa icon="cloud" fixed-width/>
							<span>{{ $t('@.drive') }}</span>
						</router-link>
					</li>
					<li v-if="($store.state.i && ($store.state.i.isLocked || $store.state.i.carefulBot))">
						<router-link to="/i/follow-requests" :class="{ active: $route.name == 'follow-requests' }">
							<fa icon="user-check" fixed-width/>
							<span>{{ $t('@.follow-requests') }}</span>
						</router-link>
					</li>
					<li v-if="$store.state.i">
						<router-link :to="`/@${ $store.state.i.username }`" :class="{ active: $route.name == 'user' }">
							<mk-avatar class="avatar" :user="$store.state.i"/>
							<span>{{ $t('@.profile') }}</span>
						</router-link>
					</li>
					<li v-if="$store.state.i">
						<router-link to="/i/settings" :class="{ active: $route.name == 'settings' }">
							<fa icon="cog" fixed-width/>
							<span>{{ $t('@.settings') }}</span>
						</router-link>
					</li>
					<li v-if="$store.state.i && ($store.state.i.isAdmin || $store.state.i.isModerator)">
						<a href="/admin">
							<fa icon="terminal" fixed-width/>
							<span>{{ $t('@.admin') }}</span>
						</a>
					</li>
				</ul>
				<ui-button primary class="post-button" v-if="$store.state.i && !$store.state.settings.showPostFormOnTopOfTl">
					<fa icon="pencil-alt" />
					<span>{{ $t('@.post') }}</span>
				</ui-button>
			</div>
			<div class="right">
				<component v-for="widget in home" :is="`mkw-${widget.name}`" :key="widget.id" :ref="widget.id" :widget="widget" platform="desktop"/>
			</div>
			<div class="main">
				<router-view ref="content"></router-view>
			</div>
		</div>
	</div>
</mk-ui>
<x-welcome v-else/>
</template>

<script lang="ts">
import Vue from 'vue';
import i18n from '../../../i18n';
import XWelcome from '../pages/welcome.vue';

export default Vue.extend({
	i18n: i18n('desktop/views/components/home.vue'),

	components: {
		XWelcome
	},

	data() {
		return {
			connection: null,
			widgetAdderSelected: null,
			trash: [],
			view: null
		};
	},

	computed: {
		home(): any[] {
				return [{
					name: 'broadcast',
					place: 'right',
					data: {}
				}, {
					name: 'profile',
					place: 'right',
					data: {design: 1}
				}, {
					name: 'calendar',
					place: 'right',
					data: {}
				}, {
					name: 'nav',
					place: 'right',
					data: {}
				}, {
					name: 'version',
					place: 'right',
					data: {}
				}];
		},
		keymap(): any {
			return {
				't': this.focus
			};
		}
	},

	created() {
		if (!this.$store.getters.isSignedIn) return;
	},

	mounted() {
		this.connection = this.$root.stream.useSharedConnection('main');
	},

	beforeDestroy() {
		this.connection.dispose();
	},

	methods: {
		onTlLoaded() {
			this.$emit('loaded');
		},
		
		goToTop() {
			window.scrollTo({
				top: 0,
				behavior: 'smooth'
			});
		},
	}
});
</script>

<style lang="stylus" scoped>
.wqsofvpm
	display block

	&[data-customize]
		padding-top 48px
		background-image url('/assets/desktop/grid.svg')

		> .main > .main
			> a
				display block
				margin-bottom 8px
				text-align center

			> div
				cursor not-allowed !important

				> *
					pointer-events none

	&:not([data-customize])
		> .main > *:not(.main):empty
			display none

	> .customize
		position fixed
		z-index 1000
		top 0
		left 0
		width 100%
		height 48px
		color var(--text)
		background var(--desktopHeaderBg)
		box-shadow 0 1px 1px rgba(#000, 0.075)

		> a
			display block
			position absolute
			z-index 1001
			top 0
			right 0
			padding 0 16px
			line-height 48px
			text-decoration none
			color var(--primaryForeground)
			background var(--primary)
			transition background 0.1s ease

			&:hover
				background var(--primaryLighten10)

			&:active
				background var(--primaryDarken10)
				transition background 0s ease

			> [data-icon]
				margin-right 8px

		> div
			display flex
			margin 0 auto
			max-width 1220px - 32px

			> div
				width 50%

				&.adder
					> p
						display inline
						line-height 48px

				&.trash
					border-left solid 1px var(--faceDivider)

					> div
						width 100%
						height 100%

					> p
						position absolute
						top 0
						left 0
						width 100%
						line-height 48px
						margin 0
						text-align center
						pointer-events none

	> .main
		justify-content center
		margin 0 auto

		> *
			.customize-container
				cursor move
				border-radius 6px

				&:hover
					box-shadow 0 0 8px rgba(64, 120, 200, 0.3)

				> *
					pointer-events none

		> .main
			margin 32px 344px

		> *:not(.main)
			width 280px
			padding 16px 0 16px 0

			> *:not(:last-child)
				margin-bottom 16px

		> .left
			position fixed
			background var(--secondary)
			left 72px
			top 32px
			width 256px
			bottom 32px
			overflow auto
			overflow-x hidden

			> ul
				list-style none 
				margin 0
				padding 0
				
				li>a
					display flex
					align-items center
					padding-left 8px
					margin 8px
					height 48px
					color var(--text)
					font-size 18px
					transition all 0.2s ease
					
					&:hover
						color var(--primary)

					.avatar
						display block
						float left
						min-width 32px
						max-width 32px
						min-height 32px
						max-height 32px
						margin 8px 16px 8px 8px
						border-radius 4px

					svg
						display block
						width 48px
						margin-right 8px
						font-size 28px
						text-align center

					&.active
						color var(--primary)
						transform translateX(16px)
			.post-button
				font-size 18px

			> .icon
				margin-left 32px
				display block
				width 48px
				text-align left
				cursor pointer
				opacity 0.5

				> svg
					width 32px
					height 48px
					vertical-align top
					fill var(--desktopHeaderFg)

		> .right
			position fixed
			right 72px
			top 16px
			width 256px

@media (max-width 1200px)
	.wqsofvpm
		> .main
			> .right
				display none

			> .left
				width 96px
				left 16px
				> ul > li > a
					> span
						display none

			> .main
				margin-left 128px 
				margin-right 16px 

</style>
