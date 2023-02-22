<template>
  <div style="margin:30px; font-size: 15px; text-align: center; text-decoration: underline;">
    请选择直接上传填写完整的数据登入表或者在线录入数据以便后续绘制控制图
    <a-divider/>
  </div>

  <div class="dash-template">
    <div class="dash-box">

          <a-upload
          >
            <a-button type="primary" size="large">
              <UploadOutlined/>
              上传数据登入表
            </a-button>
          </a-upload>

      <div class="dash-top"></div>
      <div class="dash-right"></div>
      <div class="dash-bottom"></div>
      <div class="dash-left"></div>
      <div class="dash-top-radius"></div>
      <div class="dash-right-radius"></div>
      <div class="dash-bottom-radius"></div>
      <div class="dash-left-radius"></div>
    </div>




    <div class="dash-box">
      <a-button type="primary" size="large" @click="showModal">在线录入数据</a-button>
      <a-modal
        v-model:visible="visible"
        title="收集表"
        @ok="handleOk"
      >
        <p> test </p>
        <p> test </p>
        <p> test </p>
      </a-modal>


      <div class="dash-top"></div>
      <div class="dash-right"></div>
      <div class="dash-bottom"></div>
      <div class="dash-left"></div>
      <div class="dash-top-radius"></div>
      <div class="dash-right-radius"></div>
      <div class="dash-bottom-radius"></div>
      <div class="dash-left-radius"></div>
    </div>
  </div>
</template>

<script>
import { defineComponent, ref } from 'vue';
import { UploadOutlined } from '@ant-design/icons-vue';
import { message } from 'ant-design-vue';

export default defineComponent({
  name: "DataEntry",
  components: {UploadOutlined},

  setup() {
    const visible = ref(false)

    const showModal = () => {
      visible.value = true;
    }

    const handleOk = () => {
      visible.value = false;
    }

    return {
      showModal,
      handleOk,
      visible,
    }
  }
})
</script>

<style scoped>
.dash-template {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.dash-box {
  position: relative;
  width: 50%;
  height: 450px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.dash-left {
  width: 2px;
  position: absolute;
  left: 30px;
  top: 40px;
  bottom: 40px;
  background-image: linear-gradient(to top, #2e6bee 0%, #2e6bee 50%, transparent 50%);
  background-size: 1px 8px;
}

.dash-right {
  width: 2px;
  position: absolute;
  right: 30px;
  top: 40px;
  bottom: 40px;
  background-image: linear-gradient(to bottom, #2e6bee 0%, #2e6bee 50%, transparent 50%);
  background-size: 1px 8px;
}

.dash-top {
  height: 2px;
  position: absolute;
  left: 40px;
  right: 40px;
  top: 30px;
  background-image: linear-gradient(to right, #2e6bee 0%, #2e6bee 50%, transparent 50%);
  background-size: 8px 1px;
}

.dash-bottom {
  height: 2px;
  position: absolute;
  left: 40px;
  right: 40px;
  bottom: 30px;
  background-image: linear-gradient(to left, #2e6bee 0%, #2e6bee 50%, transparent 50%);
  background-size: 8px 1px
}

.dash-top-radius {
  position: absolute;
  left: 30px;
  top: 30px;
  width: 7px;
  height: 7px;
  border-top:2px solid #2e6bee;
  border-left: 2px solid #2e6bee;
  border-top-left-radius: 100%;
}

.dash-right-radius {
  position: absolute;
  right: 30px;
  top: 30px;
  width: 7px;
  height: 7px;
  border-top:2px solid #2e6bee;
  border-right: 2px solid #2e6bee;
  border-top-right-radius: 100%;
}

.dash-bottom-radius {
  position: absolute;
  bottom: 30px;
  right: 30px;
  width: 7px;
  height: 7px;
  border-bottom:2px solid #2e6bee;
  border-right: 2px solid #2e6bee;
  border-bottom-right-radius: 100%;
}

.dash-left-radius {
  position: absolute;
  left: 30px;
  bottom: 30px;
  width: 7px;
  height: 7px;
  border-bottom:2px solid #2e6bee;
  border-left: 2px solid #2e6bee;
  border-bottom-left-radius: 100%;
}
</style>
