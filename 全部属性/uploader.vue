<!--
  @description: 文件上传 - 移动端 + 工具端共用

-->
<template>
	<div>
		<div v-if="type==='00'" ref="listDats" style="position: relative">
			<div v-if="device" style="padding-bottom: 10px;">
				<!--移动端显示-->
				<div v-if="list.length<maxCount" @click="uploadAttachment" class="avatar avatar-uploader pointer">
					<i class="el-icon-plus avatar-uploader-icon"></i>
				</div>
				<div>
					<div>
						<!--图片展示-->
						<div v-for="(item,index) in list" :key="index" class="avatar avatar-img pointer">
							<div>
								<div class="avatar-img-move">
									<span @click="deleteFile(item,index)">
										<i class="el-icon-delete"></i>
									</span>
								</div>
							</div>
							<div @click="js_native_file_show(item)">
								<img v-if="hasFile===0" :src="item.fileUrl" width="100%">
								<div v-else class="wrap" style="line-height:100px;padding: 10px">
									<span style="color: #333333">{{item.fileName}}</span>
								</div>
							</div>
						</div>
					</div>
				</div>
			</div>
			<!--客户端-工具端-->
			<div v-else>
				<div v-if="list.length<maxCount" style="position: relative" class="pointer avatar avatar-uploader pointer">
					<input class="pointer avatar" :accept="accept" ref="uploder" @change="submit" style="border: none;position: absolute;top: 0;left: 0;opacity: 0"
					 type="file">
					<i class="el-icon-plus avatar-uploader-icon pointer"></i>
				</div>
				<div v-else>
					<div>
						<!--图片展示-->
						<div v-for="(item,index) in fileList" :key="index" class="avatar avatar-img">
							<div class="avatar avatar-img-mask">
								<!--<a v-if="hasFile===1" :href="item.fileUrl">
                    <span
                      class="pointer"
                    >
                    <i class="el-icon-zoom-in"></i>
                  </span>
                  </a>-->
								<span class="pointer" @click="pictureCardPreview(item,index)">
									<i class="el-icon-zoom-in"></i>
								</span>
								<span class="pointer" @click="deleteFile(item,index)">
									<i class="el-icon-delete"></i>
								</span>
							</div>
							<img v-if="hasFile===0" :src="item.fileUrl" width="100%">
							<div v-else class="wrap" style="line-height:100px;padding: 10px">
								<span style="color: #333333">{{item.fileName}}</span>
							</div>
						</div>
					</div>
				</div>
			</div>
		</div>
		<!--文件列表上传-->
		<div v-if="type==='01'">
			<div v-if="list.length<maxCount">
				<van-button v-if="device" plain @click="uploadAttachment" type="info" class="border_radius sizeMini pointer" size="small">上传</van-button>
				<div v-else style="position: relative">
					<input class="pointer" :accept="accept" ref="uploder" @change="submit" style="border: none;position: absolute;top: 0;left: 0;opacity: 0;z-index: 1;width: 60px;"
					 type="file">
					<van-button plain type="info" class="border_radius sizeMini pointer" size="small">上传</van-button>
				</div>
			</div>
			<ul class="fileList" style="margin-top: 10px;">
				<li v-for="(item,index) in fileList" :key="index">
					<span>{{item.fileName}}</span> <span @click="deleteFile(item,index)" class="fr delete">删除</span>
				</li>
			</ul>
		</div>
		<!--只展示文件列表-->
		<div v-if="type==='02'">
			<ul class="fileList">
				<li v-for="(item,index) in fileList" :key="index">
					<a v-if="isDowload" :href="item.fileUrl" :download="item.fileName">{{item.fileName}}</a>
					<div v-else>
						<span v-if="device" @click="js_native_file_show(item)">{{item.fileName}}</span>
						<span v-else @click="pictureCardPreview(item,index)">{{item.fileName}}</span>
					</div>
				</li>
			</ul>
		</div>
		<!--大图查看-pc-工具端-->
		<el-dialog :visible.sync="dialogVisible">
			<img width="100%" :src="dialogImageUrl" alt="">
		</el-dialog>
	</div>
</template>

<script>
	import axios from 'axios'
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
			// 文件类型
			accept: {
				type: String,
				default: '*'
			},
			// 00 上传pdf 图片 01 以列表形式展示上传 02 只展示文件列表
			type: {
				type: String,
				default: '00'
			},
			// 文件列表名称
			name: {
				type: String,
				default: ''
			},
			// 大小
			previewSize: {
				type: String,
				default: '120px'
			},
			// 最大上传 M
			maxSize: {
				type: Number,
				default: 0
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
			// 是否展示可以下载效果
			isDowload: {
				type: Boolean,
				default: false
			},
			// 附件类型数组
			fileType: {
				type: Array,
				default () {
					return null
				}
			},
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
			// 校验上传是否超过限制
			oversize: {
				type: Function,
				default: noop
			},
			// 上传成功
			onSuccess: {
				type: Function,
				default: noop
			},
			// 上传之前校验
			beforeRead: {
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
				isIOS: '', // 是否为ios
				dialogVisible: false, // 弹框
				dialogImageUrl: '', // 大图查看
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
			this.isDeviceLoad()
			this.isIOSLoad()
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
			 * 安卓 - ios
			 * */
			// 上传附件
			uploadAttachment() {
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
				if (this.isIOS) {
					window.webkit.messageHandlers.js_native_file_upload.postMessage(data)
				} else {
					window.android.js_native_file_upload(data)
				}
			},
			// 文件查看
			js_native_file_show(row) {
				let data = {
					fileUrl: row.fileUrl,
					hasFile: 1
				}
				data = JSON.stringify(data)
				if (this.isIOS) {
					window.webkit.messageHandlers.js_native_file_show.postMessage(data)
				} else {
					window.android.js_native_file_show(data)
				}
			},
			// 上传文件 安卓-ios 文件类型 文件数据
			native_js_file_fetch(data, base64) {
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
				if (formData.fileType[0] === 'pdf' || formData.fileType[0] === 'PDF' || formData.fileType[0] === 'doc' || formData.fileType[
						0] === 'docx') {
					if (this.isIOS) {
						img = `data:application/${formData.fileType};base64,${formData.fileData}`
					} else {
						img = `data:application/${formData.fileType};base64,${base64}`
					}
				} else if (formData.fileType[0] === 'jpeg' || formData.fileType[0] === 'JPEG' || formData.fileType[0] === 'jpg' ||
					formData.fileType[0] === 'JPG' || formData.fileType[0] === 'png' || formData.fileType[0] === 'PNG') {
					if (this.isIOS) {
						img = `data:image/${formData.fileType};base64,${formData.fileData}`
					} else {
						img = `data:image/${formData.fileType};base64,${base64}`
					}
				}
				files = this.dataURLtoFile(img, formData.fileName)
				let params = new FormData()
				params.append('file', files)
				for (let item in formData.webFlag) {
					params.append(item, formData.webFlag[item])
				}
				axios.post(formData.webFlag.action, params).then(res => {
					if (res.data.header.returnCode === '0') {
						Toast.clear()
						Toast.success('上传成功')
						this.onSuccess(formData.webFlag.name, res.data.data)
					}
				})
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
			},
			/**
			 * pc-工具端
			 * */
			// 上传 -浏览器
			submit(file) {
				file = this.$refs.uploder.files[0]
				// 文件大小
				if (this.maxSize && file.size > this.maxSize) {
					this.oversize()
					return false
				}
				let formData = new FormData()
				formData.append('file', file)
				for (let item in this.data) {
					formData.append(item, this.data[item])
				}
				Toast.loading({
					message: '上传中...',
					forbidClick: true,
					duration: 0,
					loadingType: 'spinner'
				})
				axios.post(this.action, formData).then(res => {
					if (res.data.header.returnCode === '0') {
						Toast.clear()
						Toast.success('上传成功')
						this.listCange()
						this.onSuccess(this.name, JSON.parse(JSON.stringify(res.data.data)))
					}
				})
			},
			// 大图查看-pc-工具端
			pictureCardPreview(file) {
				/* if(file.fileName.indexOf('pdf')){

				}else{ */
				this.dialogImageUrl = !file.url ? file.fileUrl : file.url
				this.dialogVisible = true
				// }
			},
			/**
			 * 公用
			 * */
			// 当文件改变时
			listCange() {
				if (this.fileList.length > 0) {
					this.fileList.splice(0, this.fileList.length)
				}
				this.fileList.push(...this.list)
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
	.fileList {
		font-size: 14px;

		li {
			cursor: pointer;
			border-radius: 5px;
			margin-top: 10px;
			padding: 2px 10px;
			background: @bg_lightBlue;
			line-height: 24px;

			&:first-of-type {
				margin-top: 0;
			}

			.delete {
				color: red;
				cursor: pointer;
			}
		}
	}
</style>
