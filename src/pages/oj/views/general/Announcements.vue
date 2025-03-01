<template>
  <Panel shadow :padding="10">
    <div slot="title">
      公告
    </div>
    <div slot="extra">
      <Button type="info" @click="init" :loading="btnLoading">{{$t('m.Refresh')}}</Button>
    </div>

    <div align="center" slot="title" v-if="!listVisible" >
       {{announcement.title}}
    </div>
    <transition-group name="announcement-animate" mode="in-out">
      <div class="no-announcement" v-if="!announcements.length" key="no-announcement">
        <p>{{$t('m.No_Announcements')}}</p>
      </div>
      <template>
        <ul class="announcements-container" key="list">
          <li v-for="announcement in announcements" :key="announcement.title">
            <div class="flex-container">
              <div v-if="announcement.istop" class="title">
                <tag color="red">置顶</tag>
                <a class="entry" @click="goAnnouncement(announcement)">
                <font style="color:rgb(45,140,240)">
                  {{announcement.title}}
                </font>
              </a></div>
              <div v-if="!announcement.istop" class="title">
                <a class="entry" @click="goAnnouncement(announcement)">
                <font style="color:rgb(45,140,240)">
                  {{announcement.title}}
                </font>
              </a></div>
              <div class="date" style="text-align : left;">{{announcement.create_time}}发布</div>
              <div class="date" style="text-align : left;">{{announcement.last_update_time}}更新</div>
              <div class="creator"> {{$t('m.By')}} {{announcement.created_by.username}}</div>
            </div>
          </li>
        </ul>
        <Pagination v-if="!isContest"
                    key="page"
                    :total="total"
                    :page-size="limit"
                    @on-change="getAnnouncementList">
        </Pagination>
      </template>

    </transition-group>

    <el-dialog
    :visible.sync="dialogVisible"
    width="80%"
    style=""
    :before-close="done">
      <template slot="title">
        <h1 style="text-align:center;">{{this.announcement.title}}</h1>
      </template>
      <el-card>
      <div v-katex v-html="this.announcement.content" key="content" class="content-container markdown-body"></div>
      </el-card>
      <span slot="footer" class="dialog-footer">
        <el-button  type="primary" @click="dialogVisible = false">返回</el-button>
      </span>

    </el-dialog>

  </Panel>
</template>

<script>
  import api from '@oj/api'
  import Pagination from '@oj/components/Pagination'

  export default {
    name: 'Announcement',
    components: {
      Pagination
    },
    data () {
      return {
        limit: 10,
        total: 10,
        btnLoading: false,
        announcements: [],
        announcement: '',
        listVisible: true,
        dialogVisible: false
      }
    },
    mounted () {
      this.init()
    },
    methods: {
      getAItime (time) {
        var ttime
        ttime = new Date(time).toJSON()
        ttime = new Date(+new Date(ttime) + 28800000).toISOString().replace(/T/g, ' ').replace(/\.[\d]{3}Z/, ' ').replace(/-/gi, '/')
        ttime = Date.parse(ttime)
        var now = new Date().getTime()
        var d = now - ttime
        var monthC = d / 2592000000
        var weekC = d / 604800000
        var dayC = d / 86400000
        var hourC = d / 3600000
        var minC = d / 60000
        if (monthC >= 1) return parseInt(monthC) + '个月前'
        else if (weekC >= 1) return parseInt(weekC) + '周前'
        else if (dayC >= 1) return parseInt(dayC) + '天前'
        else if (hourC >= 1) return parseInt(hourC) + '小时前'
        else if (minC >= 1) return parseInt(minC) + '分钟前'
        else return '刚刚'
      },
      init () {
        if (this.isContest) {
          this.getContestAnnouncementList()
        } else {
          this.getAnnouncementList()
        }
      },
      getAnnouncementList (page = 1) {
        let params = {
          limit: this.limit,
          offset: (page - 1) * this.limit
        }
        this.btnLoading = true
        api.getAnnouncementList(params).then(res => {
          this.btnLoading = false
          this.announcements = this.sortList(res.data.data.results)
          this.total = res.data.data.total
        }, () => {
          this.btnLoading = false
        })
      },
      getContestAnnouncementList () {
        this.btnLoading = true
        api.getContestAnnouncementList(this.$route.params.contestID).then(res => {
          this.btnLoading = false
          this.announcements = this.sortList(res.data.data)
        }, () => {
          this.btnLoading = false
        })
      },
      sortList (arr) {
        let list = []
        for (let i = 0; i < arr.length; i++) {
          arr[i].create_time = this.getAItime(arr[i].create_time)
          arr[i].last_update_time = this.getAItime(arr[i].last_update_time)
          if (arr[i].istop) {
            list.push(arr[i])
          }
        }
        for (let i = 0; i < arr.length; i++) {
          if (!arr[i].istop) {
            list.push(arr[i])
          }
        }
        return list
      },
      goAnnouncement (announcement) {
        this.announcement = announcement
        this.dialogVisible = true
      }
    },
    computed: {
      title () {
        if (this.listVisible) {
          return this.isContest ? this.$i18n.t('m.Contest_Announcements') : this.$i18n.t('m.Announcements')
        } else {
          return this.announcement.title
        }
      },
      isContest () {
        return !!this.$route.params.contestID
      }
    }
  }
</script>

<style  lang="less">
  .announcements-container {
    margin-top: -10px;
    margin-bottom: 10px;

    li {
      padding-top: 15px;
      list-style: none;
      padding-bottom: 15px;
      margin-left: 20px;
      font-size: 16px;
      border-bottom: 1px solid rgba(187, 187, 187, 0.5);

      &:last-child {
        border-bottom: none;
      }

      .flex-container {
        .title {
          flex: 1 1;
          text-align: left;
          padding-left: 10px;

          a.entry {
            color: #495060;

            &:hover {
              color: #2d8cf0;
              border-bottom: 1px solid #2d8cf0;
            }
          }
        }

        .creator {
          flex: none;
          width: 200px;
          text-align: center;
        }

        .date {
          flex: none;
          width: 200px;
          text-align: center;
        }
      }
    }
  }

  img{
    max-width: 100%;
    object-fit: contain;
  }

  .content-container {
    padding: 0 20px 20px 20px;
  }

  .no-announcement {
    text-align: center;
    font-size: 16px;
  }

  changeLocale
  .announcement-animate-enter-active {
    animation: fadeIn 1s;
  }
</style>
