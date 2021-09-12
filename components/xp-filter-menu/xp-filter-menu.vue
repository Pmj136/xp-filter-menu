<template>
	<view class="data-filter" ref="wrap">
		<scroller scroll-direction="horizontal" class="scroll-view">
			<view v-for="(item,index) in menus" ref="item" :key="item.key" :class="[{
					'is-active':portVisible && activeMenuKey===item.key,
					'border-active':isBorderHighlight(item.key),
					'text-active':!!selectedTxt[item.key]
				},'menu-item']" @tap.stop="clickMenuItem(item.key,index)">
				<view class="menu-item-container"
					:style="{width:item.width || '180rpx',marginRight:index===menus.length-1?'0':'10px'}">
					<text class="title">{{selectedTxt[item.key] || item.title}}</text>
					<view class="arrow"></view>
				</view>
				<view class="empty-view" :style="{width:item.width || '180rpx'}"></view>
			</view>
		</scroller>
		<view v-if="portVisible" :style="maskStyle" class="mask" @tap.stop="closeViewPort"></view>
		<view v-if="portVisible" ref="content" :style="viewPortStyle" class="view-port" @tap.stop>
			<scroller>
				<view class="content-wrap">
					<view v-for="item in contentList" :key="item.id"
						:class="[{'is-selected':currSelectedList.indexOf(item.id) !== -1},'content-item']"
						@tap.stop="doSelect(item.id)">
						<text>{{item.name}}</text>
					</view>
				</view>
			</scroller>
			<view class="action-wrap">
				<text class="action action-reset" @tap.stop="reset">重置</text>
				<text class="action action-confirm" @tap.stop="confirm">确定</text>
			</view>
		</view>
	</view>
</template>

<script>
	const dom = uni.requireNativePlugin('dom')
	const animation = uni.requireNativePlugin('animation')
	export default {
		props: {
			menus: {
				type: Array,
				default: () => []
			},
			options: {
				type: Object,
				default: () => {}
			}
		},
		data() {
			return {
				wrapHeight: 0,
				portVisible: false,
				activeMenuKey: null,
				refs: [],
				selected: {},
				selectedTxt: {},
				confirmFlag: {}
			}
		},
		computed: {
			maskStyle() {
				return {
					top: this.wrapHeight + 'px'
				}
			},
			viewPortStyle() {
				const style = {
					top: this.wrapHeight + 'px',
				}
				if (this.contentList.length > 12) {
					style.height = '450rpx'
				}
				return style
			},
			contentList() {
				if (this.activeMenuKey == null) return []
				return this.options[this.activeMenuKey]
			},
			currSelectedList() {
				if (this.activeMenuKey == null) return []
				return this.selected[this.activeMenuKey]
			}
		},
		created() {
			for (const v of this.menus) {
				this.selected[v.key] = []
				this.confirmFlag[v.key] = false
			}
		},
		mounted() {
			this.$nextTick(() => {
				dom.getComponentRect(this.$refs.wrap, option => {
					this.wrapHeight = option.size.height
				})
				this.refs = this.$refs.item
			})
		},
		methods: {
			clickMenuItem(key, i) {
				//重复点击关闭
				if (this.activeMenuKey === key && this.portVisible) {
					this.closeViewPort()
					return
				}
				//切换
				if (this.activeMenuKey != null && this.activeMenuKey !== key) {
					this.portVisible = false
					this._clearSelected()
				}
				this.activeMenuKey = key
				this.$nextTick(() => {
					dom.scrollToElement(this.refs[i], {
						offset: -130
					})
					this.portVisible = true
					this.$nextTick(() => {
						animation.transition(this.$refs.content, {
							styles: {
								transform: 'rotateX(0)'
							},
							duration: 140, //ms
							timingFunction: 'ease',
							delay: 0 //ms
						})
					})
				})
			},
			closeViewPort() {
				animation.transition(this.$refs.content, {
					styles: {
						transform: 'rotateX(90deg)'
					},
					duration: 140, //ms
					timingFunction: 'ease',
					delay: 0 //ms
				}, () => {
					this._clearSelected()
					this.activeMenuKey = null
					this.portVisible = false
				})
			},
			doSelect(pk) {
				const arr = this.selected[this.activeMenuKey]
				const index = arr.indexOf(pk)
				if (index !== -1) {
					//移除
					arr.splice(index, 1)
				} else {
					arr.push(pk)
				}
				this.$forceUpdate()
			},
			reset() {
				this.selected[this.activeMenuKey] = []
				this.selectedTxt[this.activeMenuKey] = undefined
				this._emitResult()
			},
			confirm() {
				this.selectedTxt[this.activeMenuKey] = this._getSelectedTxt()
				this.confirmFlag[this.activeMenuKey] = true
				this._emitResult()
			},
			isSelected(v) {
				if (this.activeMenuKey == null) return false
				return this.selected[this.activeMenuKey].indexOf(v) !== -1
			},
			isBorderHighlight(key) {
				return (!!this.selectedTxt[key] && !this.portVisible) ||
					(this.activeMenuKey !== key && !!this.selectedTxt[key])
			},
			_clearSelected() {
				if (!this.confirmFlag[this.activeMenuKey])
					this.selected[this.activeMenuKey] = []
			},
			_emitResult() {
				this.$emit("change", this.selected)
				this.closeViewPort()
			},
			_getSelectedTxt() {
				const key = this.activeMenuKey
				if (key === undefined) return undefined
				const arr = this.selected[key]
				const res = this.options[key].filter(v => arr.indexOf(v.id) !== -1)
				let str = ""
				for (const v of res) {
					str += v.name + "/"
				}
				return str.replace(/^(.*)\/$/, "$1")
			}
		}
	}
</script>

<style scoped lang="scss">
	$fill-color:#ff5500; //填充文字后颜色
	$action-reset-color:#FF5500;
	$action-confirm-color:#0077AA;

	.data-filter {
		background-color: #fff;
		padding: 15rpx 30rpx 0 30rpx;
		.scroll-view {
			width: 690rpx;
			flex-direction: row;

			.menu-item {
				.menu-item-container {
					flex-direction: row;
					background-color: #eee;
					padding: 7.5px 9.5px;
					align-items: center;
					border-top-left-radius: 5px;
					border-top-right-radius: 5px;
					border-bottom-left-radius: 5px;
					border-bottom-right-radius: 5px;
					border-width: 1px;
					border-color: transparent;

					.title {
						font-size: 14px;
						flex: 1;
						lines: 1;
						text-overflow: ellipsis;
					}

					.arrow {
						width: 7px;
						height: 7px;
						border-left: 1px solid #999;
						border-bottom: 1px solid #999;
						transform: rotate(-45deg);
					}
				}

				.empty-view {
					height: 15rpx;
				}

			}

			.is-active {
				.menu-item-container {
					border-bottom-left-radius: 0 !important;
					border-bottom-right-radius: 0 !important;

					.arrow {
						transform: rotate(135deg);
					}
				}

				.empty-view {
					background-color: #eee;
				}
			}

			.border-active {
				.menu-item-container {
					border-color: $fill-color !important;
				}
			}

			.text-active {
				.menu-item-container {
					.title {
						color: $fill-color !important;
					}

					.arrow {
						border-color: $fill-color !important;
					}
				}
			}
		}

		.mask {
			position: fixed;
			top: 0;
			bottom: 0;
			left: 0;
			right: 0;
			background-color: #666;
			opacity: 0.5;
		}

		.view-port {
			background-color: #eee;
			position: fixed;
			width: 750rpx;
			border-bottom-left-radius: 10px;
			border-bottom-right-radius: 10px;
			transform: rotateX(90deg);
			transform-origin: center top;

			.content-wrap {
				padding: 6px 30rpx;
				flex-direction: row;
				flex-wrap: wrap;
				justify-content: space-between;

				.content-item {
					margin: 4px 0;
					width: 340rpx;
				}

				.is-selected {
					background-color: #0077AA;
				}
			}

			.action-wrap {
				background-color: #ffffff;
				flex-direction: row;
				justify-content: center;
				padding: 6px 0;

				.action {
					width: 340rpx;
					padding: 8px 0;
					text-align: center;
					color: #ffffff;
				}

				.action-reset {
					background-color: $action-reset-color;
					border-top-left-radius: 30px;
					border-bottom-left-radius: 30px;
				}

				.action-confirm {
					background-color: $action-confirm-color;
					border-top-right-radius: 30px;
					border-bottom-right-radius: 30px;
				}
			}
		}
	}
</style>
