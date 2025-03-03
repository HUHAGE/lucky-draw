<template>
  <div id="root">
    <header>
      <div class="title">年会抽奖</div>
      <div class="result-display">
        <Publicity v-show="!running" />
      </div>
      <div class="header-buttons">
        <el-button class="res" type="text" @click="showResult = true">
          <i class="el-icon-trophy"></i>
          抽奖结果
        </el-button>
        <el-button class="con" type="text" @click="showConfig = true">
          <i class="el-icon-setting"></i>
          抽奖配置
        </el-button>
      </div>
    </header>

    <div class="main-content">
      <div id="main" :class="{ mask: showRes }"></div>
      <div id="tags">
        <ul v-for="item in datas" :key="item.key">
          <li>
            <a
              href="javascript:void(0);"
              :style="{
                color: '#fff',
              }"
            >
              {{ item.name ? item.name : item.key }}
              <img v-if="item.photo" :src="item.photo" :width="50" :height="50" />
            </a>
          </li>
        </ul>
      </div>
      <transition name="bounce">
        <div id="resbox" v-show="showRes">
          <p @click="showRes = false">{{ categoryName }}抽奖结果：</p>
          <div class="container">
            <span
              v-for="item in resArr"
              :key="item"
              class="itemres"
              :style="resCardStyle"
              :data-id="item"
              @click="showRes = false"
              :class="{
                numberOver:
                  !!photos.find((d) => d.id === item) ||
                  !!list.find((d) => d.key === item),
              }"
            >
              <span class="cont" v-if="!photos.find((d) => d.id === item)">
                <span
                  v-if="!!list.find((d) => d.key === item)"
                  :style="{
                    fontSize: '40px',
                  }"
                >
                  {{ list.find((d) => d.key === item).name }}
                </span>
                <span v-else>
                  {{ item }}
                </span>
              </span>
              <img
                v-if="photos.find((d) => d.id === item)"
                :src="photos.find((d) => d.id === item).value"
                alt="photo"
                :width="160"
                :height="160"
              />
            </span>
          </div>
        </div>
      </transition>

      <el-button
        class="audio"
        type="text"
        @click="
          () => {
            playAudio(!audioPlaying);
          }
        "
      >
        <i
          class="iconfont"
          :class="[audioPlaying ? 'iconstop' : 'iconplay1']"
        ></i>
      </el-button>

      <LotteryConfig :visible.sync="showConfig" @resetconfig="reloadTagCanvas" />
      <Tool
        @toggle="toggle"
        @resetConfig="reloadTagCanvas"
        @getPhoto="getPhoto"
        :running="running"
        :closeRes="closeRes"
      />
      <Result :visible.sync="showResult"></Result>

      <span class="copy-right">
        Copyright©zhangyongfeng5350@gmail.com
      </span>

      <audio
        id="audiobg"
        preload="auto"
        controls
        autoplay
        loop
        @play="playHandler"
        @pause="pauseHandler"
      >
        <source :src="audioSrc" />
        你的浏览器不支持audio标签
      </audio>
    </div>
  </div>
</template>
<script>
import LotteryConfig from '@/components/LotteryConfig';
import Publicity from '@/components/Publicity';
import Tool from '@/components/Tool';
import bgaudio from '@/assets/bg.mp3';
import beginaudio from '@/assets/begin.mp3';
import {
  getData,
  configField,
  resultField,
  newLotteryField,
  conversionCategoryName,
  listField,
} from '@/helper/index';
import { luckydrawHandler } from '@/helper/algorithm';
import Result from '@/components/Result';
import { database, DB_STORE_NAME } from '@/helper/db';
export default {
  name: 'App',

  components: { LotteryConfig, Publicity, Tool, Result },

  computed: {
    resCardStyle() {
      const style = { fontSize: '30px' };
      const { number } = this.config;
      if (number < 100) {
        style.fontSize = '100px';
      } else if (number < 1000) {
        style.fontSize = '80px';
      } else if (number < 10000) {
        style.fontSize = '60px';
      }
      return style;
    },
    config: {
      get() {
        return this.$store.state.config;
      },
    },
    result: {
      get() {
        return this.$store.state.result;
      },
      set(val) {
        this.$store.commit('setResult', val);
      },
    },
    list() {
      return this.$store.state.list;
    },
    allresult() {
      let allresult = [];
      for (const key in this.result) {
        if (this.result.hasOwnProperty(key)) {
          const element = this.result[key];
          allresult = allresult.concat(element);
        }
      }
      return allresult;
    },
    datas() {
      const { number } = this.config;
      const nums = number >= 1500 ? 500 : this.config.number;
      const configNum = number > 1500 ? Math.floor(number / 3) : number;
      const randomShowNums = luckydrawHandler(configNum, [], nums);
      const randomShowDatas = randomShowNums.map((item) => {
        const listData = this.list.find((d) => d.key === item);
        const photo = this.photos.find((d) => d.id === item);
        return {
          key: item * (number > 1500 ? 3 : 1),
          name: listData ? listData.name : '',
          photo: photo ? photo.value : '',
        };
      });
      return randomShowDatas;
    },
    categoryName() {
      return conversionCategoryName(this.category);
    },
    photos() {
      return this.$store.state.photos;
    },
  },
  created() {
    const data = getData(configField);
    if (data) {
      this.$store.commit('setConfig', Object.assign({}, data));
    }
    const result = getData(resultField);
    if (result) {
      this.$store.commit('setResult', result);
    }

    const newLottery = getData(newLotteryField);
    if (newLottery) {
      const config = this.config;
      newLottery.forEach((item) => {
        this.$store.commit('setNewLottery', item);
        if (!config[item.key]) {
          this.$set(config, item.key, 0);
        }
      });
      this.$store.commit('setConfig', config);
    }

    const list = getData(listField);
    if (list) {
      this.$store.commit('setList', list);
    }
  },

  data() {
    return {
      running: false,
      showRes: false,
      showConfig: false,
      showResult: false,
      resArr: [],
      category: '',
      audioPlaying: false,
      audioSrc: bgaudio,
    };
  },
  watch: {
    photos: {
      deep: true,
      handler() {
        this.$nextTick(() => {
          this.reloadTagCanvas();
        });
      },
    },
  },
  mounted() {
    this.startTagCanvas();
    setTimeout(() => {
      this.getPhoto();
    }, 1000);
    window.addEventListener('resize', this.reportWindowSize);
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.reportWindowSize);
  },
  methods: {
    reportWindowSize() {
      const AppCanvas = this.$el.querySelector('#rootcanvas');
      if (AppCanvas.parentElement) {
        AppCanvas.parentElement.removeChild(AppCanvas);
      }
      this.startTagCanvas();
    },
    playHandler() {
      this.audioPlaying = true;
    },
    pauseHandler() {
      this.audioPlaying = false;
    },
    playAudio(type) {
      if (type) {
        this.$el.querySelector('#audiobg').play();
      } else {
        this.$el.querySelector('#audiobg').pause();
      }
    },
    loadAudio() {
      this.$el.querySelector('#audiobg').load();
      this.$nextTick(() => {
        this.$el.querySelector('#audiobg').play();
      });
    },
    getPhoto() {
      database.getAll(DB_STORE_NAME).then((res) => {
        if (res && res.length > 0) {
          this.$store.commit('setPhotos', res);
        }
      });
    },
    speed() {
      return [0.1 * Math.random() + 0.01, -(0.1 * Math.random() + 0.01)];
    },
    createCanvas() {
      const canvas = document.createElement('canvas');
      canvas.width = document.body.offsetWidth;
      canvas.height = document.body.offsetHeight;
      canvas.id = 'rootcanvas';
      this.$el.querySelector('#main').appendChild(canvas);
    },
    startTagCanvas() {
      this.createCanvas();
      const { speed } = this;
      window.TagCanvas.Start('rootcanvas', 'tags', {
        textColour: null,
        initial: speed(),
        dragControl: 1,
        textHeight: 20,
        noSelect: true,
        lock: 'xy',
      });
    },
    reloadTagCanvas() {
      window.TagCanvas.Reload('rootcanvas');
    },
    closeRes() {
      this.showRes = false;
    },
    toggle(form) {
      const { speed, config } = this;
      if (this.running) {
        this.audioSrc = bgaudio;
        this.loadAudio();

        window.TagCanvas.SetSpeed('rootcanvas', speed());
        this.showRes = true;
        this.running = !this.running;
        this.$nextTick(() => {
          this.reloadTagCanvas();
        });
      } else {
        this.showRes = false;
        if (!form) {
          return;
        }

        this.audioSrc = beginaudio;
        this.loadAudio();

        const { number } = config;
        const { category, mode, qty, remain, allin } = form;
        let num = 1;
        if (mode === 1 || mode === 5) {
          num = mode;
        } else if (mode === 0) {
          num = remain;
        } else if (mode === 99) {
          num = qty;
        }
        const resArr = luckydrawHandler(
          number,
          allin ? [] : this.allresult,
          num
        );
        this.resArr = resArr;

        this.category = category;
        if (!this.result[category]) {
          this.$set(this.result, category, []);
        }
        const oldRes = this.result[category] || [];
        const data = Object.assign({}, this.result, {
          [category]: oldRes.concat(resArr),
        });
        this.result = data;
        window.TagCanvas.SetSpeed('rootcanvas', [5, 1]);
        this.running = !this.running;
      }
    },
  },
};
</script>
<style lang="scss">
#root {
  height: 100%;
  position: relative;
  background-color: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);

  header {
    height: 80px;
    padding: 0 30px;
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(10px);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    display: flex;
    align-items: center;
    justify-content: space-between;
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    z-index: 100;

    .title {
      font-size: 36px;
      font-weight: bold;
      color: white;
      text-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
      letter-spacing: 4px;
      background: linear-gradient(45deg, #fff 30%, #ffeb3b 50%, #fff 70%);
      -webkit-background-clip: text;
      background-clip: text;
      -webkit-text-fill-color: transparent;
      background-size: 200% auto;
      animation: shine 3s linear infinite;
      width: 200px;
    }

    .result-display {
      flex: 1;
      display: flex;
      justify-content: center;
      align-items: center;
      padding: 0 20px;
      overflow: hidden;
    }

    .header-buttons {
      display: flex;
      gap: 20px;
      width: 200px;
      justify-content: flex-end;
      
      .el-button {
        font-size: 16px;
        color: white;
        padding: 10px 20px;
        border: 2px solid rgba(255, 255, 255, 0.4);
        border-radius: 25px;
        backdrop-filter: blur(5px);
        transition: all 0.3s ease;
        
        i {
          margin-right: 5px;
          font-size: 18px;
        }
        
        &:hover {
          background: rgba(255, 255, 255, 0.2);
          transform: translateY(-2px);
          box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
        }
        
        &.res {
          background: linear-gradient(45deg, #ff6b6b, #ff8e8e);
          border: none;
          
          &:hover {
            background: linear-gradient(45deg, #ff8e8e, #ff6b6b);
          }
        }
        
        &.con {
          background: linear-gradient(45deg, #23a6d5, #23d5ab);
          border: none;
          
          &:hover {
            background: linear-gradient(45deg, #23d5ab, #23a6d5);
          }
        }
      }
    }
  }

  .main-content {
    margin-top: 80px;
    height: calc(100vh - 80px);
    position: relative;
    overflow: hidden;
  }

  .audio {
    position: absolute;
    top: 100px;
    right: 30px;
    width: 40px;
    height: 40px;
    line-height: 40px;
    border: 1px solid #fff;
    border-radius: 50%;
    padding: 0;
    text-align: center;
    .iconfont {
      position: relative;
      left: 1px;
    }
  }
  .copy-right {
    position: absolute;
    right: 0;
    bottom: 0;
    color: #ccc;
    font-size: 12px;
  }
  .bounce-enter-active {
    animation: bounce-in 1.5s;
  }
  .bounce-leave-active {
    animation: bounce-in 0s reverse;
  }
}
#main {
  height: 100%;
}

#resbox {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 1280px;
  transform: translateX(-50%) translateY(-50%);
  text-align: center;
  p {
    color: red;
    font-size: 50px;
    line-height: 120px;
  }
  .container {
    display: flex;
    justify-content: center;
    flex-wrap: wrap;
  }
  .itemres {
    background: rgba(255, 255, 255, 0.9);
    border-radius: 8px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    transition: all 0.3s ease;
    width: 160px;
    height: 160px;
    border: 1px solid #ccc;
    line-height: 160px;
    font-weight: bold;
    margin-right: 20px;
    margin-bottom: 20px;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    position: relative;
    .cont {
      display: flex;
      justify-content: center;
      align-items: center;
    }
    &.numberOver::before {
      content: attr(data-id);
      width: 30px;
      height: 22px;
      line-height: 22px;
      background-color: #fff;
      position: absolute;
      bottom: 0;
      left: 0;
      font-size: 14px;
      z-index: 1;
    }
    &:hover {
      transform: translateY(-5px);
      box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
    }
  }
}

@keyframes shine {
  to {
    background-position: 200% center;
  }
}
</style>
