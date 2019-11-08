<!--
  @description: 文件上传-ios 安卓交互
-->
<template>
	<div>
		<!--点击触发上传事件-->
		<div v-if="list.length<maxCount" @click="uploadAttachment" class="avatar avatar-uploader pointer">
			<i class="el-icon-plus avatar-uploader-icon"></i>
		</div>
		<!--图片展示-->
		<div>
			<div v-for="(item,index) in list" :key="index" class="avatar avatar-img pointer">
				<div>
					<!-- 删除按钮 -->
					<div class="avatar-img-move">
						<span @click="deleteFile(item,index)">
							<i class="el-icon-delete"></i>
						</span>
					</div>
				</div>
				<div>
					<!-- 图片 -->
					<img :src="item.fileUrl" width="100%">
				</div>
			</div>
		</div>
	</div>
</template>

<script>
	import axios from 'axios'
	// vant-ui
	import {
		Toast
	} from 'vant'

	function noop() {}
	export default {
		name: 'uploader',
		props: {
			// 上传地址
			action: {
				type: String,
				default: ''
			},
			// 文件列表名称
			name: {
				type: String,
				default: ''
			},
			// 样式大小
			previewSize: {
				type: String,
				default: '120px'
			},
			// 上传类型 0 图片 1 文件
			hasFile: {
				type: Number,
				default: 0
			},
			// 上传个数
			maxCount: {
				type: Number,
				default: 1
			},
			// 文件 图片列表
			fileList: {
				type: Array,
				default () {
					return []
				}
			},
			// 上传文件时传的参数
			data: {
				type: Object,
				default () {
					return {}
				}
			},
			// 上传成功
			onSuccess: {
				type: Function,
				default: noop
			},
			// 删除
			beforeDelete: {
				type: Function,
				default: noop
			}
		},
		data() {
			return {
				device: '', // 手机端/pc端
				isIOS: null, // 是否为ios
				list: []
			}
		},
		watch: {
			fileList(val) {
				console.log(this.list)
				this.list = [...val]
			}
		},
		created() {
			this.isIOSLoad()
			this.isDeviceLoad()
			// 安卓-ios
			window.native_js_file_fetch = this.native_js_file_fetch
		},
		activated() {
			// 安卓-ios
			window.native_js_file_fetch = this.native_js_file_fetch
		},
		methods: {
			// 判断是工具端还是移动端
			isDeviceLoad() {
				let flag = navigator.userAgent.match(
					/(phone|pad|pod|iPhone|iPod|ios|iPad|Android|Mobile|BlackBerry|IEMobile|MQQBrowser|JUC|Fennec|wOSBrowser|BrowserNG|WebOS|Symbian|Windows Phone)/i
				)
				if (flag) {
					// 移动端
					this.device = true
				} else {
					// 客户端
					this.device = false
				}
			},
			// 判断是IOS还是Android
			isIOSLoad() {
				var u = navigator.userAgent,
					app = navigator.appVersion
				var isAndroid = u.indexOf('Android') > -1 || u.indexOf('Linux') > -1 // Android
				var isIOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/) // ios
				if (isAndroid) {
					this.isIOS = false
				}
				if (isIOS) {
					this.isIOS = true
				}
			},
			/**
			 * * Android - ios
			 * */
			// 上传附件
			uploadAttachment() {
				// 我传给原生的内容
				const data = JSON.stringify({
					webFlag: {
						...this.data,
						name: this.name,
						action: this.action
					}, // 前端标识符
					hasFile: this.hasFile, // 是否需要上传附件
					fileType: this.fileType, // 附件类型数组
					fileName: null, // 附件名
					fileData: null // 图片或者附件数据的base64编码
				})
				// 区分调用方法
				if (this.isIOS) {
					window.webkit.messageHandlers.js_native_file_upload.postMessage(data)
				} else {
					window.android.js_native_file_upload(data)
				}
			},
			// 调用成功后返回的数据是之前传给原生的内容，并且把上传的文件内容传给我
			// 接收数据
			// 上传文件 安卓-ios 文件类型 文件数据
			native_js_file_fetch(data, base64) {
				// vant-ui
				Toast.loading({
					message: '上传中...',
					forbidClick: true,
					duration: 0,
					loadingType: 'spinner'
				})
				let img = ''
				let formData = ''
				let files = ''
				formData = JSON.parse(data)
				// 判断上传的文件类型
				if (formData.fileType[0] === 'pdf' || formData.fileType[0] === 'PDF' || formData.fileType[0] === 'doc' || formData.fileType[
						0] === 'docx') {
					// 判断是ios或安卓区分调用方法
					if (this.isIOS) {
						img = `data:application/${formData.fileType};base64,${formData.fileData}`
					} else {
						img = `data:application/${formData.fileType};base64,${base64}`
					}
				} else if (formData.fileType[0] === 'jpeg' || formData.fileType[0] === 'JPEG' || formData.fileType[0] === 'jpg' ||
					formData.fileType[0] === 'JPG' || formData.fileType[0] === 'png' || formData.fileType[0] === 'PNG') {
					// 判断是ios或安卓区分调用方法
					if (this.isIOS) {
						img = `data:image/${formData.fileType};base64,${formData.fileData}`
					} else {
						img = `data:image/${formData.fileType};base64,${base64}`
					}
				}
				// base64 转 file文件
				files = this.dataURLtoFile(img, formData.fileName)

				// 上传到服务端的交互
				let params = new FormData()
				// 创建实例
				params.append('file', files)
				// 插入上传所需的参数
				for (let item in formData.webFlag) {
					params.append(item, formData.webFlag[item])
				}
				axios.post(formData.webFlag.action, params).then(res => {
					if (res.data.header.returnCode === '0') {
						Toast.clear()
						Toast.success('上传成功')
						// 成功回调
						this.onSuccess(formData.webFlag.name, res.data.data)
					}
				})
			},
			//  删除前
			deleteFile(file, index) {
				const type = this.beforeDelete(file, this.fileList)
				if (type) {
					this.fileList.splice(index, 1)
				}
			},
			// 删除成功
			deleteRemove() {
				this.listCange()
			},
			// 当文件改变时
			listCange() {
				if (this.fileList.length > 0) {
					this.fileList.splice(0, this.fileList.length)
				}
				this.fileList.push(...this.list)
			},
			/**
			 * base64转文件对象
			 * @param ''
			 * @returns string
			 */
			dataURLtoFile(dataurl, filename) {
				var arr = dataurl.split(',')
				var mime = arr[0].match(/:(.*?);/)[1]
				var bstr = atob(arr[1])
				var n = bstr.length
				var u8arr = new Uint8Array(n)
				while (n--) {
					u8arr[n] = bstr.charCodeAt(n)
				}
				// 转换成file对象
				return new File([u8arr], filename, {
					type: mime
				})
				// 转换成成blob对象
				// return new Blob([u8arr],{type:mime});
			}
		}
	}
</script>

<style scoped lang="less">
	@bg_lightBlue: #DDEFFB;

	/**
      * 文件上传--start
   **/
	.avatar {
		width: 120px;
		height: 120px;
		display: inline-block;
		line-height: 120px;
		-webkit-border-radius: 6px;
		-moz-border-radius: 6px;
		border-radius: 6px;
		overflow: hidden;
		transition: 0.3s;
		border: 1px dashed #d9d9d9;
		background: #F2F4F6;
	}

	.avatar:hover {
		border-color: #409EFF;
	}

	.avatar-uploader {
		.avatar-uploader-icon {
			font-size: 28px;
			color: #8c939d;
			width: 120px;
			height: 120px;
			line-height: 120px;
			text-align: center;
		}
	}

	.avatar-img {
		position: relative;

		.avatar-img-mask {
			width: 120px;
			height: 120px;
			opacity: 0;
			transition: 0.5s;
			position: absolute;
			top: 0;
			left: 0;
			text-align: center;

			span {
				padding: 0 10px;
				color: #409EFF;
			}
		}

		.avatar-img-move {
			color: #fff;
			position: absolute;
			right: 1px;
			padding: 2px 4px;
			background: rgba(0, 0, 0, 0.3);
			bottom: 1px;
			line-height: 20px;
		}

		&:hover .avatar-img-mask {
			opacity: 1;
			background: rgba(0, 0, 0, .5);
		}

		.img {
			witth: 120px !important;
		}
	}

	/**
      * 文件上传--end
   **/
</style>
