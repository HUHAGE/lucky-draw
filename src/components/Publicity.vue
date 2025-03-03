<template>
  <div class="c-Publicity">
    <div class="publicity-content">
      <div v-if="currentMessage" class="item" :class="{ actiname: currentMessage.key === 0 }">
        <span v-if="currentMessage.title" class="title">
          {{ currentMessage.title }}
        </span>
        <span v-if="currentMessage.value" class="value">
          {{ currentMessage.value }}
        </span>
      </div>
    </div>
  </div>
</template>

<script>
import { conversionCategoryName } from '@/helper/index';

export default {
  name: 'Publicity',
  data() {
    return {
      currentIndex: 0,
      timer: null
    };
  },
  computed: {
    config() {
      return this.$store.state.config;
    },
    result() {
      return this.$store.state.result;
    },
    message() {
      const { result, config } = this;
      const fields = Object.keys(config);

      let message = [{ key: 0, title: config.name }];
      fields.forEach((item, index) => {
        let label = conversionCategoryName(item);
        if (result[item] && config[item] > 0) {
          message.push({
            key: index + 1,
            title: `${label}抽奖结果:`,
            value: `${
              result[item].length > 0 ? result[item].join('、') : '暂未抽取'
            }`
          });
        }
      });

      return message;
    },
    currentMessage() {
      return this.message[this.currentIndex];
    }
  },
  methods: {
    startRotation() {
      this.timer = setInterval(() => {
        this.currentIndex = (this.currentIndex + 1) % this.message.length;
      }, 3000);
    },
    stopRotation() {
      clearInterval(this.timer);
    }
  },
  mounted() {
    this.startRotation();
  },
  beforeDestroy() {
    this.stopRotation();
  }
};
</script>

<style lang="scss">
.c-Publicity {
  height: 100%;
  padding: 0 200px;
  position: relative;
  z-index: 1;
  
  .publicity-content {
    height: 100%;
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  
  .item {
    text-align: center;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 20px;
    height: 100%;
    width: 100%;
    animation: fadeIn 0.5s ease-in-out;
    
    .title {
      color: #ffffff;
      font-size: 16px;
      text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.1);
    }
    
    .value {
      color: #ffffff;
      text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.1);
    }
    
    &.actiname {
      .title {
        font-size: 36px;
        font-weight: bold;
        background: linear-gradient(45deg, #ff6b6b, #ff8e8e);
        -webkit-background-clip: text;
        background-clip: text;
        -webkit-text-fill-color: transparent;
        text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
        letter-spacing: 2px;
      }
    }
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
