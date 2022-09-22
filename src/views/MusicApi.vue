<template>

  <el-row>
    <el-table
        :data="tableData.slice((pageData.pageCurrent - 1) * pageData.pageSize, pageData.pageCurrent * pageData.pageSize)"
        style="width: 100%"
    >
      <el-table-column type="index" label="序" width="50" />
      <el-table-column prop="FileName" align="left">
        <template #header>
          <el-input
              v-model="search.search_keyword"
              placeholder="关键字..."
              class=""
              @keyup.enter.native="GetMusicList"
              style="padding: 2px 0 0 2px;">
            <template #prepend>
              <el-select v-model="search.search_type" placeholder="Select" style="width: 115px">
                <el-option label="歌名" value="1" />
                <el-option label="歌手" value="2" />
                <el-option label="专辑" value="3" />
              </el-select>
            </template>
            <template #append>
              <el-button type="default" @click="GetMusicList">
                <el-icon><Search /></el-icon>
              </el-button>
            </template>
          </el-input>
        </template>
      </el-table-column>
      <el-table-column :width="100" prop="SingerName" label="歌手" />
      <el-table-column :width="200" prop="FileHash" label="下载">
        <template #default="scope">
          <div>
            <el-button size="small" v-if="tableData[scope.$index].FileHash.length>0" @click="GetMusicDetail(tableData[scope.$index].FileHash, tableData[scope.$index].albumid)">标清</el-button>
            <el-button size="small" v-if="tableData[scope.$index].HQFileHash.length>0" @click="GetMusicDetail(tableData[scope.$index].HQFileHash, tableData[scope.$index].albumid)">高清</el-button>
            <el-button size="small" v-if="tableData[scope.$index].SQFileHash.length>0" @click="GetMusicDetail(tableData[scope.$index].SQFileHash, tableData[scope.$index].albumid)">无损</el-button>
          </div>
        </template>
      </el-table-column>
    </el-table>

    　　<el-pagination
      :page-size="pageData.pageSize"
      :page-sizes="[10, 20, 50, 100,200]"
      :total="pageData.pageTotal"
      v-model:page-size="pageData.pageSize"
      @current-change="currentChange"
      @prev-click="prevClick"
      @next-click="nextClick"
      @size-change="sizeChange"
      background
      layout="total, sizes, prev, pager, next, jumper"
      style="padding-top: 8px"/>

  </el-row>
  <el-dialog v-model="dialogVisible" title="单曲下载" style="text-align: center;width: 300px" justify="center" type="flex" draggable>
    <el-card :body-style="{ padding: '0px' }">
      <img :src="MusicDetail.img" style="max-width: 250px" alt=""/>
      <div style="padding: 14px">
        <div>歌曲:{{ MusicDetail.song_name }}</div>
        <div class="musicDetail">歌手:{{ MusicDetail.author_name }}</div>
        <el-input v-model="MusicDetail.play_url" ref="myUrl">
          <template #append>
            <el-button type="default" @click="copyUrl">
              <el-icon><CopyDocument /></el-icon>
            </el-button>
          </template>
        </el-input>
      </div>
    </el-card>

  </el-dialog>
</template>


<script>
import {ref, reactive} from 'vue'
import axios from 'axios'
import { Download,Search,CopyDocument } from '@element-plus/icons-vue'
import { ElMessage } from 'element-plus'

export default {
  SQFileHash: "MusicApi",
  methods:{
    copyUrl(){
      this.$refs.myUrl.select();
      document.execCommand("Copy");
      ElMessage({
        message: '复制成功',
        type: 'success',
      })
    }

  },
  setup(){
    const search = reactive({
      search_type: '',
      search_keyword: '',
    })

    let dialogVisible = ref(false);

    const GetMusicList = () => {
      axios.get(`http://47.106.91.103:15059/music/get_music_list/`+search.search_keyword+`/`+search.search_type)
          .then(res=>{
            tableData.value = res.data;
            pageData.pageTotal = res.data.length;
          })
    }

    const GetMusicDetail = (hash, albumid) => {
      let data = [hash +'_'+ albumid]
      axios.post(`http://47.106.91.103:15059/music/get_music_by_hash/`, data)
          .then(res=>{
            MusicDetail.value = res.data[0];
            dialogVisible.value = true;
          })
    }

    const DownloadMusic = (url) => {
      console.log(url);
      const a = document.createElement('a')
      // 这里是将url转成blob地址，
      fetch(url).then(res => res.blob()).then(blob => { // 将链接地址字符内容转变成blob地址
        a.href = URL.createObjectURL(blob)
        a.download = '' // 下载文件的名字
        // a.download = url.split('/')[url.split('/').length -1] //  // 下载文件的名字
        document.body.appendChild(a)
        a.click()
      })
    }


    const test = (obj) => {
      console.log(tableData.value[obj].FileHash.length);
    }
    let tableData = ref([{"AlbumName":"星辰不坠落", "Duration": "2:38", "FileHash": "0B8F99A8D0B3599DCE1BB98CCF4E01A4", "FileName": "蓝心羽 - 星辰不坠落", "HQFileHash": "320C56D8284B743EF09B926249170676","SQFileHash": "DF441CE6DE8F3050CBAD90B989A3C849", "SingerName" :"蓝心羽","albumid":"46590460"}]);
    let MusicDetail = ref({"author_name": "蓝心羽",
      "img": "http://imge.kugou.com/stdmusic/20211021/20211021173947143953.jpg",
      "play_url": "https://webfs.ali.kugou.com/202209180943/8d3aaf90dd765c0c0dcb8592308237ab/KGTX/CLTX001/0b8f99a8d0b3599dce1bb98ccf4e01a4.mp3",
      "song_name": "星辰不坠落",
    })

    let pageData = reactive({
      pageCurrent: 1,
      pageSize: 10,
      pageTotal: 1,
    })

    const currentChange = (page) =>{
      pageData.pageCurrent = page;
    }
    const prevClick = () =>{
      pageData.pageCurrent -= 1;
    }
    const nextClick = () =>{
      pageData.pageCurrent += 1;
    }
    const sizeChange = (size) =>{
      pageData.pageSize = size;
    }
    return {test, search,tableData,GetMusicList,GetMusicDetail,DownloadMusic,dialogVisible,MusicDetail, Download,Search,CopyDocument, pageData,currentChange,prevClick,nextClick,sizeChange}

  }

}
</script>

<style scoped>

</style>



