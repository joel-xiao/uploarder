<!--
  @description: 使用组件方法
-->
<template>
  <div>
    <!--content-->
    <div class="content">
      <uploader
        name="qualificationsFileList"
        :oversize="oversize"
        :maxSize="5242880"
        accept="image/*"
        :action="uploadFile"
        :data="qualificationsData"
        :onSuccess="afterRead"
        :beforeDelete="deleteFile"
        :fileList="qualificationsFileList"
        :maxCount="1"
        :beforeRead="beforeRead"
      ></uploader>
    </div>
  </div>
</template>

<script>
import uploader from './uploader'//引入组件
import {randomData} from '@/assets/js/util/func'
import {getFileList, uploadFile} from '@/api/common/common.js'

import {deleteFile} from '@/api/declare/index'
export default {
  name: 'qualificationInfo',
  components: {
    uploader
  },
  data () {
    return {
      uploadFile: uploadFile, // 上传地址
      qualificationsFileList: [
		  /* 格式必须以这种格式---如果需要更改请看uploarder组件*/
		  {
			  fileId:'',//文件Id
			  fileName:'',//文件名称
			  fileUrl:''//文件地址
		  }
	  ], // 文件列表
      qualificationsData: {
		  // 上传时的参数 不是必填，如果需要必须是json格式
	  }
    }
  },
  created () {
  },
  mounted () {
  },
  methods: {
    // 获取文件时赋值
    getData () {
      getFileList().then(res => {
          this.qualificationsFileList = res.data
      })
    },
    // 上传成功
	/* 
	 @name: 文件列表名称
	 @file:上传成功的文件
	 @必须赋值-此方法不可缺少
	 */
    afterRead (name, file) {
      this[name].push(file)
    },
    // 删除
	/*
	 @fileList: 文件列表
	 @file:要删除的文件
	 @必须 return 一个 promise 对象
	 */
    deleteFile (file, fileList) {
      const data = {
        fileId: file.fileId
      }
      return new Promise(function (resolve, reject) {
        deleteFile(data).then(res => {
          if (res.header.returnCode === '0') {
            resolve()
          } else {
          }
        })
      })
    },
    /* 图片上传 -end */
  }
}
</script>

<style scoped lang="less">
</style>
