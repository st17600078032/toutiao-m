<template>
  <div class="channel-edit">
    <van-cell :border="false">
      <div slot="title" class="title-text">我的频道</div>
      <van-button
        class="edit-btn"
        type="danger"
        plain
        round
        size="mini"
        @click="isEdit = !isEdit"
        >{{ isEdit ? '完成' : '编辑' }}</van-button
      >
    </van-cell>
    <van-grid class="my-grid" :gutter="10">
      <van-grid-item
        class="grid-item"
        v-for="(channel, index) in myChannels"
        :key="index"
        @click="onMyChannelClick(channel, index)"
      >
        <van-icon
          v-show="isEdit && !fiexdChannels.includes(channel.id)"
          slot="icon"
          name="clear"
        ></van-icon>
        <span class="text" :class="{ active: index === active }" slot="text">{{
          channel.name
        }}</span>
      </van-grid-item>
    </van-grid>
    <!-- 频道推荐 -->
    <van-cell :border="false">
      <div slot="title" class="title-text">频道推荐</div>
    </van-cell>
    <van-grid class="recommend-grid" :gutter="10">
      <van-grid-item
        class="grid-item"
        v-for="(channel, index) in recommendChannels"
        :key="index"
        icon="plus"
        :text="channel.name"
        @click="onAddChannel(channel)"
      />
    </van-grid>
  </div>
</template>

<script>
import {
  getAllChannels,
  addUserChannel,
  deleteUserChannel
} from '@/api/channel'
import { mapState } from 'vuex'
import { setItem } from '@/utils/storage.js'

export default {
  name: 'ChannelEdit',
  components: {},
  props: {
    myChannels: {
      type: Array,
      required: true
    },
    active: {
      type: Number,
      required: true
    }
  },
  data() {
    return {
      allChannels: [],
      isEdit: false,
      fiexdChannels: [0] // 固定频道，不允许删除
    }
  },
  computed: {
    recommendChannels() {
      return this.allChannels.filter((channel) => {
        const mychannel = this.myChannels.find((myChannel) => {
          return myChannel.id === channel.id
        })
        return !mychannel
      })
      // const channels = []
      // this.allChannels.forEach((channel) => {
      //   const ret = this.myChannels.find((myChannel) => {
      //     return myChannel.id === channel.id
      //   })
      //   if (!ret) {
      //     channels.push(channel)
      //   }
      // })
      // return channels
    },
    ...mapState(['user'])
  },
  watch: {},
  created() {
    this.loadAllChannels()
  },
  mounted() {},
  methods: {
    async loadAllChannels() {
      try {
        const { data } = await getAllChannels()
        this.allChannels = data.data.channels
        console.log(this.allChannels)
      } catch (err) {
        this.$toast('获取数据失败')
      }
    },

    // 点击添加频道
    async onAddChannel(channels) {
      // eslint-disable-next-line vue/no-mutating-props
      this.myChannels.push(channels)
      if (this.user) {
        try {
          // 已登录，数据存储到线上
          await addUserChannel({
            id: channels.id, // 频道 id
            seq: this.myChannels.length // 频道的 序号
          })
          this.$toast('添加成功')
        } catch (err) {
          this.$toast('保存失败')
        }
      } else {
        // 未登陆
        setItem('TOUTIAO_CHANNELS', this.myChannels)
      }
    },

    async deleteChannel(channel) {
      try {
        if (this.user) {
          await deleteUserChannel(channel.id)
        } else {
          // 未登陆
          setItem('TOUTIAO_CHANNELS', this.myChannels)
        }
      } catch (err) {
        this.$toast('请稍后重试')
      }
    },

    onMyChannelClick(channel, index) {
      if (this.isEdit) {
        // 如果是固定频道 则不删除
        if (this.fiexdChannels.includes(channel.id)) {
          return
        }
        // eslint-disable-next-line vue/no-mutating-props
        this.myChannels.splice(index, 1)
        // 编辑状态 执行删除频道
        if (index <= this.active) {
          // 让激活频道的索引- 1
          this.$emit('update-active', this.active - 1, true)
        }

        // 处理持久化
        this.deleteChannel(channel)
      } else {
        // 非编辑状态 执行切换频道
        this.$emit('update-active', index, false)
      }
    }
  }
}
</script>

<style scoped lang="less">
.channel-edit {
  padding: 85px 0;

  .title-text {
    font-size: 32px;
    color: #333;
  }

  .edit-btn {
    width: 104px;
    height: 48px;
    font-size: 26px;
    color: #f85959;
    border: 1px solid #f85959;
  }

  /deep/ .grid-item {
    width: 160px;
    height: 86px;
    .van-grid-item__content {
      white-space: nowrap;
      background-color: #f4f5f6;
      .van-grid-item__text,
      .text {
        color: #222;
        font-size: 28px;
        margin-top: 0;
      }
      .active {
        color: red;
      }
      .van-grid-item__icon-wrapper {
        position: unset;
      }
    }
  }

  /deep/ .my-grid {
    .grid-item {
      .van-icon-clear {
        position: absolute;
        right: -10px;
        top: -10px;
        font-size: 30px;
        color: #cacaca;
        z-index: 2;
      }
    }
  }

  /deep/ .recommend-grid {
    .grid-item {
      .van-grid-item__content {
        flex-direction: row;
        .van-icon-plus {
          font-size: 28px;
          margin-right: 6px;
        }
      }
    }
  }
}
</style>
