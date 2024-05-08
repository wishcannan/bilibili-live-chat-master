<template>
  <div id="live">
    <danmaku-list
      ref="giftPinList"
      v-bind="props"
      :gift-show-face="showFace"
      :is-gift-list="true"
      v-if="props.giftPin"
    />
    <danmaku-list ref="danmakuList" v-bind="props" />
  </div>
</template>

<script>
import { onBeforeUnmount, ref, onMounted, computed } from 'vue';
import { KeepLiveWS } from 'bilibili-live-ws';
import { propsType } from '@/utils/props';
import DanmakuList from '@/components/DanmakuList';

export default {
  components: { DanmakuList },
  props: {
    ...propsType,
    anchor: Number,
    liveWsOptions: Object,
  },
  setup(props) {
    const giftPinList = ref(null);
    const danmakuList = ref(null);

    const giftCombMap = new Map();
    const showFace = computed(() => props.face !== 'false');

    const blockUIDs = computed(() => new Set(props.blockUID.split(/,|\|/).map(uid => uid.trim())));
    const isBlockedUID = uid => blockUIDs.value.has(String(uid));

    let failedTimestamps = [];

    const addInfoDanmaku = message => {
      danmakuList.value.addDanmaku({
        type: 'info',
        message,
        stay: props.stay || 5000,
      });
    };
    const addDanmaku = danmaku => {
      if (props.limit) danmakuList.value.addSpeedLimitDanmaku(danmaku);
      else danmakuList.value.addDanmaku(danmaku);
    };
    function timestampToTime(timestamp){
      timestamp = timestamp ? timestamp : null;
      let date = new Date(timestamp);
      let h = (date.getHours() < 10 ? '0' + date.getHours() : date.getHours()) + ':';
      let m = (date.getMinutes() < 10 ? '0' + date.getMinutes() : date.getMinutes()) + ':';
      let s = date.getSeconds() < 10 ? '0' + date.getSeconds() : date.getSeconds();
      return h + m +s
    }

    onMounted(() => {
      console.log('正在连接直播弹幕服务器', props.room);
      const live = new KeepLiveWS(props.room, props.liveWsOptions || { protover: 3, uid: 0 });
      live.interval = 1000;
      onBeforeUnmount(() => live.close());
      live.on('open', () => {
        if (live.closed) return;
        console.log('已连接直播弹幕服务器');
        addInfoDanmaku('已连接直播弹幕服务器');
      });
      live.on('live', () => {
        if (live.closed) return;
        console.log('已连接直播间', props.room);
        addInfoDanmaku(`已连接直播间 ${props.room}`);
      });
      live.on('close', () => {
        if (live.closed) return;
        console.log('连接已断开');
        addInfoDanmaku('连接已断开');
        const now = Date.now();
        failedTimestamps = failedTimestamps.filter(time => now - time < 10000);
        failedTimestamps.push(now);
        if (failedTimestamps.length >= 3) {
          console.log('连接失败过于频繁，停止重连');
          addInfoDanmaku('连接失败过于频繁，停止重连');
          live.close();
        }
      });

      // 礼物
      const giftList = props.giftPin ? giftPinList : danmakuList;
      live.on('SEND_GIFT', ({ data }) => {
        handleSendGift(data);
      });
      live.on('LIVE_OPEN_PLATFORM_SEND_GIFT', ({ data: { uid, uname, gift_name, gift_num, uface } }) => {
        handleSendGift({ uid, uname, giftName: gift_name, num: gift_num, face: uface });
      });
      const handleSendGift = ({ uid, uname, giftName, num,price, face, }) => {
        if (isBlockedUID(uid)) {
          console.log(`屏蔽了来自[${uname}]的礼物：${giftName}*${num}`);
          return;
        }
        if (props.giftComb) {
          const key = `${uid}-${giftName}`;
          const existComb = giftCombMap.get(key);
          if (existComb) {
            giftCombMap.set(key, {
              ...existComb,
              num: existComb.num + num,
            });
          } else {
            giftCombMap.set(key, {
              type: 'gift',
              showFace: showFace.value,
              uid,
              uname,
              giftName,
              num,
              price,
              face,
            });
            setTimeout(() => {
              giftList.value.addDanmaku(giftCombMap.get(key));
              giftCombMap.delete(key);
            }, props.giftComb);
          }
        } else {
          giftList.value.addDanmaku({
            type: 'gift',
            showFace: showFace.value,
            uid,
            uname,
            giftName,
            num,
            price,
            face,
          });
        }
      };

      // 弹幕
      // live.on('DANMU_MSG', ({ info: [, message, [uid, uname, isOwner]], dm_v2 }) => {
      live.on('DANMU_MSG', ({ info }) => {
        const uid = info[2][0]//用户uid
        const uname = info[2][1]//用户名
        const admin = info[2][2]//房管 非房管为0 房管为1
        const message = info[1] //消息
        const privilege_type=info[7] //舰队类型，0非舰队，1总督，2提督，3舰长
        const face = info[0][15]['user']['base']['face'] //头像
        const mode_info_extra = JSON.parse(info[0][15]['extra']) //一点点关于弹幕的信息
        const emoticon_options = info[0][13] ? info[0][13]['url']:''//表情 如果是发这种表情 都是单发的
        const timestamp = timestampToTime(info[0][4]) //时间戳
        // console.log(timestamp)
        if(mode_info_extra['reply_mid'] !== 0){ 
          //这个reply_mid 就是被@的人的uid
          handleDanmaku({ uid, uname, message,emoticon:emoticon_options, admin,privilege_type,reply_uname:mode_info_extra['reply_uname'],face,timestamp });
        }else{
          handleDanmaku({ uid, uname, message,emoticon:emoticon_options, admin,privilege_type,face,timestamp });
        }
      });
      live.on('LIVE_OPEN_PLATFORM_DM', ({ data: { uid, uname, msg, uface } }) => {
        handleDanmaku({ uid, uname, message: msg, face: uface });
      });
      const handleDanmaku = ({ uid, uname, message,emoticon, isOwner, privilege_type,reply_uname,face,timestamp }) => {
        if (isBlockedUID(uid)) {
          console.log(`屏蔽了来自[${uname}]的弹幕：${message}`);
          return;
        }
        const danmaku = {
          type: 'message',
          showFace: true,
          uid,
          uname,
          message,
          emoticon,
          isAnchor: uid === props.anchor,
          isOwner: !!isOwner,
          privilege_type:privilege_type,
          reply_uname,
          face,
          timestamp,
        };
        if (props.delay > 0) setTimeout(() => addDanmaku(danmaku), props.delay * 1000);
        else addDanmaku(danmaku);
      };

      // SC
      live.on(
        'SUPER_CHAT_MESSAGE',
        ({
          data: {
            uid,
            user_info: { uname, face },
            message,
          },
        }) => {
          handleSuperChat({ uid, uname, message, face });
        }
      );
      live.on('LIVE_OPEN_PLATFORM_SUPER_CHAT', ({ data: { uid, uname, message, uface } }) => {
        handleSuperChat({ uid, uname, message, face: uface });
      });
      const handleSuperChat = ({ uid, uname, message, face }) => {
        giftList.value.addDanmaku({
          type: 'sc',
          showFace: showFace.value,
          uid,
          uname,
          message,
          face,
        });
      };

      // 舰长
      const guardLevelMap = { 1: '总督', 2: '提督', 3: '舰长' };
      live.on('GUARD_BUY', ({ data: { uid, username, gift_name, num } }) => {
        handleGuard({ uid, uname: username, giftName: gift_name, num });
      });
      live.on('USER_TOAST_MSG', ({ data: { uid, username, role_name, num, unit } }) => {
        handleGuard({ uid, uname: username, giftName: role_name, num, unit });
      });
      live.on(
        'LIVE_OPEN_PLATFORM_GUARD',
        ({
          data: {
            user_info: { uid, uname, uface },
            guard_level,
            guard_num,
            guard_unit,
          },
        }) => {
          handleGuard({
            uid,
            uname,
            giftName: guardLevelMap[guard_level],
            num: guard_num,
            unit: guard_unit,
            face: uface,
          });
        }
      );
      const handleGuard = ({ uid, uname, giftName, num, unit, face }) => {
        giftList.value.addDanmaku({
          type: 'gift',
          showFace: showFace.value,
          uid,
          uname,
          giftName: unit ? `${num}个${unit}${giftName}` : giftName,
          num: unit ? 0 : num,
          face,
        });
      };
    });

    return { props, showFace, giftPinList, danmakuList };
  },
};
</script>

<style lang="scss">
#live {
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
  background-color: transparent;
  display: flex;
  flex-direction: column;
}
</style>
