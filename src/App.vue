<template>
  <div class="address-book" ref="addressBook">
    <dl
      class="letter-list"
      :class="{
        active: selectingLetter
      }"
      @touchmove.stop="handleTouchMove"
      @touchend="handleTouchEnd"
    >
      <dt
        v-for="item in letters"
        :key="item.letter"
        @click.capture="go(item.top)"
        :data-offset="item.top"
        :class="{
          current: prevOffset == item.top
        }"
      >
        {{ item.letter }}
        <span class="select-bubble">
          {{ item.letter }}
        </span>
      </dt>
    </dl>
    <div ref="letterBlocks">
      <div
        class="letter-block"
        :data-letter="item.letter"
        v-for="item in letters"
        :key="item.letter"
      >
        <div class="header">{{ item.letter }}</div>
        <div class="body">
          <div class="item" v-for="f in item.friends" :key="f.id">
            <img class="avatar" :src="f.avatar" />
            <span>{{ f.username }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// test only
const testAvatar =
  "http://static-cdn.lxzmww.xyz/onechat/avatar/1578408726092?imageView2/2/w/100/h/200/format/jpg/q/74";
const LetterMap = "A,B,C,D,E,F,G,H,I,J,K,L,M,N,O,P,Q,R,S,T,U,V,W,X,Y,Z,a,b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t,u,v,w,x,y,z,_,1,2,3,4,5,6,7,8,9,0,!,@,#,$,%,%,^,&".split(
  ","
);
const AlphabetMap = "A,B,C,D,E,F,G,H,I,J,K,L,M,N,O,P,Q,R,S,T,U,V,W,X,Y,Z".split(
  ","
);

export default {
  data() {
    return {
      // 记录每个字母所拥有的好友、首字母以及距顶部偏移量
      // {
      //   top: number,
      //   friends: Array<FriendItem>,
      //   letter: string
      // }
      letters: [],
      prevOffset: -999,
      selectingLetter: false,
      indexBarPosX: "",
      friends: []
    };
  },
  mounted() {
    // 模拟数据
    this.mockData();
    // 初始化通讯录
    this.init();
    // 计算字母块所占的高度
    this.calculateLocation();
  },
  methods: {
    mockData() {
      for (let i = 0; i < 200; ++i) {
        const f = {
          id: i,
          username:
            LetterMap[parseInt(Math.random() * LetterMap.length)] +
            "开头测试名字",
          avatar: testAvatar
        };
        this.friends.push(f);
      }
    },
    /**
     * 初始化数据，让通讯录中的人按首字母分离
     */
    init() {
      let letters = []; // 数据列表
      let friendsMap = {}; // 每一个索引指向一个数组，这个数组存储该字母索引的朋友项
      this.friends.forEach(friend => {
        let firstLetter = friend.username[0].toUpperCase();
        if (!AlphabetMap.includes(firstLetter)) {
          firstLetter = "#";
        }
        friendsMap[firstLetter] || (friendsMap[firstLetter] = []);
        friendsMap[firstLetter].push(friend);
      });

      for (let letter in friendsMap) {
        letters.push({
          letter: letter,
          top: 0,
          height: 0,
          friends: friendsMap[letter]
        });
      }

      // 按首字母排序
      // 字符串比较不能用减号得出大小，返回都是NaN
      // 因此这里需要通过小于号比较
      letters.sort((a, b) => {
        return a.letter <= b.letter ? -1 : 1;
      });

      this.letters = letters;
    },
    /**
     * 计算位置信息
     */
    calculateLocation() {
      this.$nextTick(() => {
        // 按字母排序分开的 dom 列表
        let list = [].slice.call(this.$refs.letterBlocks.children, 0);
        let len = list.length;
        // 对于每一个块计算距顶部的距离top 以及 通过getBoundingClientRect计算自身的高度
        // 顶部的距离 top = 上一块的高度 + 上一块的top值
        list.forEach((node, idx) => {
          if (idx !== -1) {
            this.letters[idx].top =
              idx > 0
                ? this.letters[idx - 1].top + this.letters[idx - 1].height
                : 0;
            this.letters[idx].height = node.getBoundingClientRect().height;
          }
        });
      });
    },
    go(offset) {
      this.$refs.addressBook.scrollTop = offset;
    },
    handleTouchEnd() {
      this.prevOffset = -9999;
      this.selectingLetter = false;
    },
    handleTouchMove(e) {
      if (!this.selectingLetter) {
        this.indexBarPosX = e.touches[0].clientX;
      }
      this.selectingLetter = true;
      const x = this.indexBarPosX;
      const y = e.touches[0].clientY;
      let target = document.elementFromPoint(x, y);
      let offset = target && target.dataset && target.dataset.offset;
      if (offset && offset !== this.prevOffset) {
        this.prevOffset = offset;
        this.go(offset);
      }
    }
  }
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
html,
body {
  width: 100%;
  height: 100%;
}
</style>

<style lang="scss" scoped>
.address-book {
  position: relative;
  width: 100%;
  height: 100%;
  overflow-y: scroll;
}

.letter-list {
  position: fixed;
  top: 100px;
  right: 30px;
  padding: 10px;
  color: #3f536e;
  border-radius: 10px;

  &.active {
    background: #f8f8f8;
  }

  dt {
    position: relative;
    padding: 1px 10px;

    &.current {
      transform: scale(1.5);

      .select-bubble {
        display: block;
      }
    }

    .select-bubble {
      position: absolute;
      top: 50%;
      right: 200%;
      transform: translateY(-50%);
      display: none;
      width: 200px;
      height: 200px;
      text-align: center;
      line-height: 200px;
      font-size: 100px;
      background: #8eaadf;
      border-radius: 50% 50%;
      color: #fff;
    }
  }
}

.letter-block {
  font-size: 80px;
  font-weight: bold;

  .header {
    margin-left: 20px;
    color: #3f536e;
    height: 150px;
    line-height: 150px;
    font-size: 120px;
    font-weight: bold;
    border-bottom: 1px solid #e8e8e8;
  }

  .item {
    padding: 10px 20px;
    height: 200px;
    line-height: 200px;
    user-select: none;

    .avatar {
      float: left;
      margin-top: 20px;
      margin-right: 40px;
      width: 160px;
      height: 160px;
      border-radius: 10px;
    }

    &:active {
      background: #e9e9e9;
    }
  }
}
</style>
