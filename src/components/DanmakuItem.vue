<template>
  <div class="danmaku-item" :class="{ hidden: isHidden }">
    <div v-if="type === 'message'" class="danmaku-content">
      <img v-if="showFace && face" class="danmaku-author-face" :src="face" />
      <div class="danmaku-right">
        <div class="danmaku-author-name with-colon" :class="[{ anchor: isAnchor, owner: isOwner},'guard'+(privilege_type===0?'0':privilege_type+'')]">{{ uname }}</div>
        <div class="danmaku-reply-name" v-if="reply_uname">@{{reply_uname}}</div>
        <img v-if="emoticon" class="danmaku-emoticon" :src="emoticon" />
        <div v-else class="danmaku-message">{{ message }}</div>
        <div class="danmaku-message">{{timestamp}}</div>
      </div>
    </div>
    <div v-else-if="type === 'gift'" class="danmaku-content">
      <div class="danmaku-right">
        <span class="danmaku-message">❤感谢&nbsp;</span>
        <span class="danmaku-author-name">{{ uname }}</span>
        <span class="danmaku-message">&nbsp;赠送的&nbsp;</span>
        <span class="danmaku-gift-name">{{ giftName }}</span>
        <template v-if="num">
          <span class="danmaku-message">&nbsp;×&nbsp;</span>
          <span class="danmaku-gift-num">{{ num }}</span>
          <span class="danmaku-gift-price">￥{{price * num/1000}}</span>
        </template>
      </div>
    </div>
    <div v-else-if="type === 'sc'" class="danmaku-content">
      <span class="danmaku-message">>感谢&nbsp;</span>
      <span class="danmaku-author-name">{{ uname }}</span>
      <span class="danmaku-message">&nbsp;的SC：{{ message }}</span>
    </div>
    <div v-else-if="type === 'info'" class="danmaku-content">
      <span class="danmaku-message info">{{ message }}</span>
    </div>
  </div>
</template>

<script>
import { toRefs, ref, computed, onBeforeUnmount } from 'vue';

export default {
  props: {
    type: {
      type: String,
      default: 'message',
    },
    showFace: Boolean,
    face: String,
    uid: Number,
    uname: String,
    message: String,
    emoticon:String,
    isAnchor: Boolean,
    isOwner: Boolean,
    privilege_type:Number,
    timestamp:String,
    giftName: String,
    num: Number,
    price:Number,
    stay: Number,
    hidden: Boolean,
    reply_uname:String,
  },
  setup(props) {
    const hidden = ref(false);
    const needRemoved = ref(false);
    if (props.stay) {
      const stayTimeout = setTimeout(() => {
        hidden.value = true;
        if (props.type === 'info') {
          setTimeout(() => (needRemoved.value = true), 800);
        }
      }, props.stay);
      onBeforeUnmount(() => clearTimeout(stayTimeout));
    }
    const isHidden = computed(() => props.hidden || hidden.value);
    return { ...toRefs(props), isHidden, needRemoved };
  },
};
</script>

<style lang="scss">
@keyframes danmakuIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
.danmaku {
  &-item {
    padding: 4px;
    display: flex;
    flex-direction: row;
    align-items: flex-start;
    transition: opacity 0.5s;
    user-select: none;
    text-shadow:
      -2px -2px #000000,
      -2px -1px #000000,
      -2px 0 #000000,
      -2px 1px #000000,
      -2px 2px #000000,
      -1px -2px #000000,
      -1px -1px #000000,
      -1px 0 #000000,
      -1px 1px #000000,
      -1px 2px #000000,
      0 -2px #000000,
      0 -1px #000000,
      0 0 #000000,
      0 1px #000000,
      0 2px #000000,
      1px -2px #000000,
      1px -1px #000000,
      1px 0 #000000,
      1px 1px #000000,
      1px 2px #000000,
      2px -2px #000000,
      2px -1px #000000,
      2px 0 #000000,
      2px 1px #000000,
      2px 2px #000000;
    animation: 0.5s danmakuIn;
    opacity: 1;
    &.hidden {
      opacity: 0;
    }
  }
  &-right{
    display: inline-block;
    max-width: 200px;
    background-color:rgba(146, 132, 132, 0.2);
    border-radius:0 8px 8px 8px;
  }
  &-author-face {
    width: 24px;
    height: 24px;
    border-radius: 24px;
    margin-right: 6px;
    display: inline-block;
    pointer-events: none;
    vertical-align: top;
  }
  &-content {
    overflow: initial; //继承元素溢出 父类的处理
  }
  &-author-name {
    color: #0dcc4c;
    margin: 8px;
    //content 的作用就是 加一个前缀 比如这样就加了:
    &.with-colon::after {
      content: ':';
      margin-left: 2px;
    }
    // 主播
    &.anchor {
      color: #fff248;
    }
    // 房管
    &.owner {
      color: #ff9800;
    }
    //总督 
    &.guard1 {
      color: #A21B24;
    }
    //提督
    &.guard2 {
      color: #684A99;
    }
    //舰长
    &.guard3 {
      color:#5A6FC6;
    }
  }
  &-reply-name{
    color:#F997B3
  }
  &-message{
    margin-top: 8px;
    color: #0dcc4c;
  }
  &-emoticon{
    width: 100px;
    display: inline-block;
    margin-top: 4px;
    vertical-align:bottom;
  }
  &-gift-num {
    color: #fff;
  }
  &-gift-name {
    color: #eb76ff;
  }
  &-gift-price{
    color: #c9bc0d;
  }
  &-message {
    font-family: 'Imprima', 'Microsoft YaHei';
    font-size: 18px;
    line-height: 18px;
    &.info {
      white-space: pre-wrap;
    }
  }
  &-gift-num {
    font-family: 'Imprima', 'Microsoft YaHei';
    font-size: 20px;
    line-height: 20px;
    font-weight: 500;
  }
  &-author-name,
  &-gift-name {
    font-family: 'Changa One', 'Microsoft YaHei';
    font-size: 20px;
    line-height: 20px;
    font-weight: 500;
  }
}
</style>
