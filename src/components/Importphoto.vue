<template>
  <el-dialog
    :visible="visible"
    :append-to-body="true"
    width="300px"
    @close="$emit('update:visible', false)"
    class="c-Importphoto"
  >
    <el-row>
      <label for="idinput">抽奖号码</label>
      <el-input
        id="idinput"
        size="mini"
        type="number"
        v-model="id"
        :min="0"
        :max="config.number"
      ></el-input>
    </el-row>
    <el-row>
      <label for="idinput">照片选择</label>
      <span class="selectbg" :data-tip="filename">
        <input
          ref="uploadinput"
          class="upload"
          type="file"
          accept=".jpg,.png"
          @change="inputChange"
        />
      </span>
    </el-row>
    <el-row class="photo">
      <label>已选照片</label>
      <img v-if="value" :src="value" alt="img" :width="140" :height="140" />
      <span v-else>暂未选择</span>
    </el-row>
    <el-row>
      支持jpg和png，照片大小不能超过150kb,建议20-50kb，建议尺寸为160*160px
    </el-row>
    <el-row class="center">
      <el-button size="mini" type="primary" @click="saveHandler"
        >保存</el-button
      >
      <el-button size="mini" @click="$emit('update:visible', false)"
        >取消</el-button
      >
    </el-row>
  </el-dialog>
</template>
<script>
import { database, DB_STORE_NAME } from '@/helper/db';

export default {
  name: 'Importphoto',
  props: {
    visible: Boolean
  },
  computed: {
    config: {
      get() {
        return this.$store.state.config;
      }
    }
  },
  data() {
    return {
      id: 0,
      value: '',
      filename: '点击选择照片'
    };
  },
  methods: {
    inputChange(e) {
      const fileList = e.target.files;
      const formData = new FormData();
      formData.append('uploadImg', fileList[0]);
      const reader = new FileReader();
      const AllowImgFileSize = 1024 * 150;
      const file = fileList[0];
      if (file) {
        this.filename = file.name;
        reader.readAsDataURL(file);
        reader.onload = () => {
          if (
            AllowImgFileSize != 0 &&
            AllowImgFileSize < reader.result.length
          ) {
            return this.$message.error('不允许上传大于150KB的图片');
          } else {
            this.value = reader.result;
          }
        };
      }
    },
    async saveHandler() {
      const { id, value } = this;
      const ID = Number(id);
      if (!ID || ID <= 0) {
        return this.$message.error('号码必须大于0的整数');
      }
      if (!value) {
        return this.$message.error('请选择照片');
      }
      const Data = await database.get(DB_STORE_NAME, ID);
      const param = {
        id: ID,
        value
      };
      database[Data ? 'edit' : 'add'](
        DB_STORE_NAME,
        Data ? ID : param,
        Data ? param : null
      )
        .then(res => {
          if (res) {
            this.$refs.uploadinput.value = '';
            this.value = '';
            this.filename = '点击选择照片';
            this.$emit('update:visible', false);
            this.$emit('getPhoto');
            this.$message({
              message: '保存成功',
              type: 'success'
            });
          } else {
            this.$message.error('保存失败');
          }
        })
        .catch(err => {
          this.$message.error(err.message);
        });
    }
  }
};
</script>
<style lang="scss">
.c-Importphoto {
  .el-dialog {
    background: rgba(255, 255, 255, 0.95);
    border-radius: 16px;
    backdrop-filter: blur(10px);
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
    overflow: hidden;

    .el-dialog__header {
      padding: 20px;
      border-bottom: 1px solid rgba(0, 0, 0, 0.1);
      
      .el-dialog__title {
        font-size: 18px;
        font-weight: 600;
        background: linear-gradient(45deg, #614385, #516395);
        -webkit-background-clip: text;
        -webkit-text-fill-color: transparent;
      }
    }

    .el-dialog__body {
      padding: 25px 20px !important;

      .el-input__inner {
        border-radius: 8px;
        border: 1px solid rgba(0, 0, 0, 0.1);
        transition: all 0.3s ease;
        
        &:hover, &:focus {
          border-color: #516395;
          box-shadow: 0 0 0 2px rgba(81, 99, 149, 0.1);
        }
      }

      .selectbg {
        display: block;
        margin: 10px 0;
        padding: 10px;
        border-radius: 8px;
        border: 1px dashed rgba(81, 99, 149, 0.5);
        background: rgba(81, 99, 149, 0.05);
        transition: all 0.3s ease;
        
        &:hover {
          border-color: #516395;
          background: rgba(81, 99, 149, 0.1);
        }
      }

      .photo {
        margin: 20px 0;
        text-align: center;
        
        img {
          border-radius: 8px;
          box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
        }
      }
    }

    .el-dialog__footer {
      padding: 10px 20px 20px;
      text-align: center;
      border-top: 1px solid rgba(0, 0, 0, 0.1);

      .el-button {
        width: 120px;
        height: 40px;
        font-size: 14px;
        border-radius: 20px;
        
        &--primary {
          background: linear-gradient(45deg, #614385, #516395);
          border: none;
          box-shadow: 0 4px 12px rgba(81, 99, 149, 0.2);
          
          &:hover {
            background: linear-gradient(45deg, #516395, #614385);
            transform: translateY(-2px);
            box-shadow: 0 6px 16px rgba(81, 99, 149, 0.3);
          }
          
          &:active {
            transform: translateY(0);
          }
        }
      }
    }
  }
  label {
    margin-right: 20px;
    vertical-align: top;
  }
  .el-input {
    width: 140px;
  }
  .el-row {
    padding: 5px 0;
  }
  .center {
    text-align: center;
  }
  .selectbg {
    display: inline-block;
    border: 1px solid #ccc;
    border-radius: 2px;
    width: 140px;
    height: 28px;
    position: relative;
    box-sizing: border-box;
    &::before {
      content: attr(data-tip);
      width: 100%;
      height: 100%;
      text-align: center;
      position: absolute;
      top: 0;
      left: 0;
      line-height: 28px;
      cursor: pointer;
      overflow: hidden;
      font-size: 12px;
    }
    .upload {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      opacity: 0;
      z-index: 10;
      cursor: pointer;
    }
  }
  .photo {
    img {
      border: 1px solid #ccc;
    }
    span {
      display: inline-block;
      border: 1px solid #ccc;
      border-radius: 2px;
      width: 140px;
      height: 140px;
      line-height: 140px;
      text-align: center;
      position: relative;
      box-sizing: border-box;
    }
  }
}
</style>
