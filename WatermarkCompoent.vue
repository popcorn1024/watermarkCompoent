<template>
	<div></div>
</template>

<script>
export default {
	name: 'Watermark',
	props: {
		// 水印显示文本
		watermarkText: {
			type: String,
			default: '爆米花配可乐',
		},
		
		// 是否允许修改
		allowChange: {
			type: Boolean,
			default: false,
		},
		
		// 销毁组件时是否去除水印
		destroyWatermark: {
			type: Boolean,
			default: true,
		},
		
		canvasWidth: {
			type: Number,
			default: 120,
		},
		
		canvasHeight: {
			type: Number,
			default: 120,
		},
		
		fontColor: {
			type: String,
			default: 'rgba(0, 0, 0, 0.1)',
		},
		
		fontStyle: {
			type: String,
			default: '12px microsoft yahei',
		},
	},
	data() {
		return {
			markElement: {},
		}
	},
	
	watch: {
		watermarkText() {
			this.$nextTick(() => {
				this.removeMarkElement()
			})
		},
	},
	
	mounted() {
		this.$nextTick(() => {
			this.init()
			if (!this.allowChange) {
				// 设置水印节点修改的DOM事件
				this.monitorNode()
			}
		})
	},
	
	methods: {
		init() {
			this.setStyle()
		},
		
		// 生成canvas画布
		drawMark() {
			let canvas = document.createElement('canvas')
			canvas.id = 'canvas'
			canvas.setAttribute('width', `${this.canvasWidth}px`)
			canvas.setAttribute('height', `${this.canvasHeight}px`)
			this.markElement = document.createElement('div')
			let ctx = canvas.getContext('2d')
			ctx.font = this.fontStyle
			ctx.fillStyle = this.fontColor
			ctx.rotate((Math.PI / 180) * 30)
			ctx.fillText(this.watermarkText || '爆米花配可乐', 20, 0)
			return canvas
		},
		
		// 绘制样式
		setStyle() {
			let base64Url = this.drawMark().toDataURL('image/png')
			// 添加样式属性
			// pointer-events: none; 元素永远不会成为鼠标事件的target
			this.markElement.setAttribute(
				'style',
				`
         position: fixed;
         top: 0;
         left: 0;
         width: 100%;
         height: 100%;
         z-index: 999;
         pointer-events: none;
         background-image: url(${base64Url})
         `,
			)
			this.markElement.id = 'watermark'
			document.body.appendChild(this.markElement)
		},
		
		// dom改变后执行回调
		domChangeCallback(mutations, observer) {
			if (mutations[0].target.id === 'watermark') {
				this.removeMarkElement()
			}
			// id 改变
			if (mutations[0].attributeName === 'id') {
				this.removeMarkElement()
				this.init()
			}
			// 节点删除
			if (mutations[0].removedNodes[0] && mutations[0].removedNodes[0]['id'] === 'watermark') {
				this.init()
			}
		},
		
		// 监听节点
		monitorNode() {
			let body = document.getElementsByTagName('body')[0]
			let options = {
				childList: true,
				attributes: true,
				characterData: true,
				subtree: true,
				attributeOldValue: true,
				characterDataOldValue: true,
			}
			let observes = new MutationObserver(this.domChangeCallback)
			observes.observe(body, options)
		},
		
		removeMarkElement() {
			document.body.removeChild(this.markElement)
		},
	},
	
	destroyed() {
		// 组件销毁时移除水印节点
		if (this.destroyWatermark) {
			this.removeMarkElement()
		}
	},
}
</script>

<style lang="less" scoped></style>
