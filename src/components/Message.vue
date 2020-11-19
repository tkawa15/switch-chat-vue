<template>
  <div class="wrapper" :class="{ wrapper__isMine: isMine }">
    <img :src="icon" class="icon" />
    <div class="card-wrapper">
      <span
        class="display-name"
        v-bind:class="{ 'display-name__isMine': isMine }"
        >{{ displayName }}</span
      >
      <div class="card" :class="{ card__isMine: isMine }">
        <span class="content">{{ message }}</span>
        <ReplyMessage
          v-for="message in messages"
          :key="message.id"
          :msgId="message.id"
          :message="message.message"
          :icon="message.icon"
          :timestamp="message.timestamp"
          :displayName="message.displayName"
          :isMine="message.isMine"
          @delete-replypost="deleteReplyPost(message.id)"
          :class="{ transparent: !message.isAlive }"
        ></ReplyMessage>
      </div>
      <span class="date" :class="{ date__isMine: isMine }"
        >{{ date }}
        <MessageGood class="MessageGood" :uid="user.uid" :msgId="msgId" />
        <fa
          icon="reply"
          type="fas"
          class="fa-icon"
          :class="{ reply__checked: replyIsChecked }"
          @click="$emit('toggle-reply')"
        ></fa>
        <span v-if="isMine" @click="$emit('delete-post')" class="delete"
          >削除
        </span>
      </span>
    </div>
  </div>
</template>

<script>
import MessageGood from './MessageGood'
import ReplyMessage from './ReplyMessage'
import { deletePost, setPostListener } from '../firebase/api.js'

export default {
  name: 'Message',
  created() {
    this.fetchData()
  },
  watch: {
    $route: 'fetchData',
  },
  components: {
    MessageGood,
    ReplyMessage,
  },
  props: {
    msgId: String,
    message: String,
    timestamp: Object,
    displayName: String,
    icon: String,
    isMine: Boolean,
    user: Object,
    replyIsChecked: Boolean,
  },
  computed: {
    date: function() {
      return this.timestamp.toLocaleString()
    },
  },
  methods: {
    fetchData: function() {
      this.roomId = this.$route.params['roomId']
    },
    addReplyMessage: function(newPost) {
      //配列に追加
      newPost.isAlive = true
      //自分の投稿かどうか
      if (newPost.uid === this.user.uid) {
        newPost.isMine = true
      } else {
        newPost.isMine = false
      }
      this.messages.push(newPost)
    },
    deleteReplyPost: function(id) {
      const ans = confirm('メッセージを削除してもよろしいですか') // 確認ダイアログの表示
      if (ans) {
        deletePost(this.roomId, id, this.msgId)
      } else {
        event.preventDefault()
      }
    },
    deleteReplyMessage: function(id) {
      const idx = this.messages.findIndex(message => message.id === id)
      this.messages[idx].isAlive = false
      this.$nextTick(() => {
        this.messages.splice(idx, 1)
      })
    },
  },
  mounted: function() {
    setPostListener(
      this.roomId,
      this.addReplyMessage,
      this.deleteReplyMessage,
      this.msgId
    )
  },
  data: function() {
    return {
      messages: [],
      roomId: this.$route.params['roomId'],
    }
  },
}
</script>

<style lang="scss" scoped>
.wrapper {
  display: grid;
  max-width: 80%;
  padding: 12px 0;
  grid-template-columns: 48px 1fr;
  grid-template-areas: 'areaA areaB';
  &__isMine {
    grid-template-columns: 1fr 48px;
    grid-template-areas: 'areaB areaA';
    margin: 0 0 0 auto;
  }
}
.icon {
  margin-top: 14px;
  width: 48px;
  height: 48px;
  border-radius: 50%;
  grid-area: areaA;
}
.card-wrapper {
  grid-area: areaB;
  margin: 0 24px;
}
.display-name {
  margin: 0 16px;
  font-size: 14px;
  display: block;
  text-align: left;
  &__isMine {
    text-align: right;
  }
}
.card {
  border-radius: 18px;
  padding: 14px;
  line-height: 18px;
  background: $color-secondary;
  &__isMine {
    background: $color-primary;
  }
}
.date {
  margin: 0 16px;
  font-size: 14px;
  display: block;
  text-align: left;
  &__isMine {
    text-align: right;
  }
}
.delete {
  cursor: pointer;
  margin-left: 8px;
  color: $color-font-link;
  text-decoration: underline;
}

.fa-icon {
  width: 12px;
  cursor: pointer;
  margin: 0 16px;
}

.MessageGood {
  display: inline;
  margin: 0 8px;
}

.reply__checked {
  color: lightgreen;
}
</style>