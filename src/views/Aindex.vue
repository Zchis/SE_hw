<template>
  <div id="index" ref="appRef">
    <div class="bg">
      <dv-loading v-if="loading">Loading...</dv-loading>
      <div v-else class="host-body">
        <!-- 头部样式 -->
        <div class="d-flex jc-center  header">
          <dv-decoration-10 class="dv-dec-10" />
          <div class="d-flex jc-center">
            <dv-decoration-8 class="dv-dec-8" :color="decorationColor" />
            <div class="title">
              <span class="title-text">海洋牧场监测可视化系统</span>
              <dv-decoration-6
                class="dv-dec-6"
                :reverse="true"
                :color="['#50e3c2', '#67a1e5']"
              />
            </div>
            <dv-decoration-8
              class="dv-dec-8"
              :reverse="true"
              :color="decorationColor"
            />
          </div>
          <dv-decoration-10 class="dv-dec-10-s" />
        </div>

        <!-- 导航栏 -->
        <div class="d-flex jc-between px-2 nav-bar">
          <div class="d-flex aside-width nav-items">
            <div
              class="react-left react-item"
              :class="{ bgc: tabbarIndex == 0 }"
            >
              <span
                class="react-left"
                :class="{ bgc: tabbarIndex == 0 }"
              ></span>
              <router-link to="/Aindex/information"
                ><span class="text" @click="changeTabbarIndex(0)"
                  >主要信息</span
                ></router-link
              >
            </div>
            <div class="react-left react-item" :class="{ bgc: tabbarIndex == 1 }">
              <router-link to="/Aindex/underwater"
                ><span class="text" @click="changeTabbarIndex(1)"
                  >水下系统</span
                ></router-link
              >
            </div>
            <div class="react-left react-item" :class="{ bgc: tabbarIndex == 2 }">
              <router-link to="/Aindex/datacenter"
                ><span class="text" @click="changeTabbarIndex(2)"
                  >数据中心</span
                ></router-link
              >
            </div>    
            
            <div class="react-left react-item" :class="{ bgc: tabbarIndex == 3 }">
              <router-link to="/Aindex/intelligent"
                ><span class="text" @click="changeTabbarIndex(3)"
                  >智能中心</span
                ></router-link
              >
            </div>

            <div class="react-left react-item" :class="{ bgc: tabbarIndex == 4 }">
              <router-link to="/Aindex/admin"
                ><span class="text" @click="changeTabbarIndex(4)"
                  >管理员界面</span
                ></router-link
              >
            </div>
          </div>

                   
          <div class="d-flex aside-width justify-end">
<!--            <div class="react-right mr-3" :class="{ bgc: tabbarIndex == 4 }">-->
<!--              <router-link to="/Aindex/admin"-->
<!--                ><span class="text" @click="changeTabbarIndex(4)"-->
<!--                  >管理员界面</span-->
<!--                ></router-link-->
<!--              >-->
<!--            </div>  -->

            <div class="react-right mr-4 react-l-s ">

              <span class="text user-interface">@管理员页面@</span>
              <span class="react-after"></span>
              <span class="text"
                >{{ dateYear }} {{ dateWeek }} {{ dateDay }}</span
              >

            </div>


          </div>

          <div class="container">
            <button @click="triggerFileUpload('water')" class="upload-button">上传水质数据</button>
            <input type="file" ref="fileInputWater" @change="handleFileUpload_water" style="display: none;" />

            <button @click="triggerFileUpload('fish')" class="upload-button">上传鱼类数据</button>
            <input type="file" ref="fileInputFish" @change="handleFileUpload_fish" style="display: none;" />

            <button @click="exportData" class="upload-button">导出数据</button>

          </div>

        </div>
        <!-- 数据内容 -->
        <div class="body-box">
          <router-view />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import drawMixin from "../utils/drawMixin";
import { formatTime } from "../utils/index.js";
import axios from 'axios';

export default {
  mixins: [drawMixin],
  data() {
    return {
      timing: null,
      loading: true,
      dateDay: null,
      dateYear: null,
      dateWeek: null,
      weekday: ["周日", "周一", "周二", "周三", "周四", "周五", "周六"],
      decorationColor: ["#568aea", "#000000"],
      tabbarIndex: 0,
    };
  },
  components: {},
  mounted() {
    this.timeFn();
    this.cancelLoading();
  },
  beforeDestroy() {
    clearInterval(this.timing);
  },
  methods: {
    timeFn() {
      this.timing = setInterval(() => {
        this.dateDay = formatTime(new Date(), "HH: mm: ss");
        this.dateYear = formatTime(new Date(), "yyyy-MM-dd");
        this.dateWeek = this.weekday[new Date().getDay()];
      }, 1000);
    },
    cancelLoading() {
      setTimeout(() => {
        this.loading = false;
      }, 500);
    },
    changeTabbarIndex(index) {
      this.tabbarIndex = index;
    },

    triggerFileUpload(type) {
      if (type === 'water') {
        this.$refs.fileInputWater.click();
      } else if (type === 'fish') {
        this.$refs.fileInputFish.click();
      }
    },
    async handleFileUpload_water(event) {
      const file = event.target.files[0];
      if (file) {
        const formData = new FormData();
        formData.append('file', file);

        try {
          const response = await axios.post('/api/water-data/import_water_file/', formData, {
            headers: {
              'Content-Type': 'multipart/form-data'
            }
          });
          console.log(response.data);
          // 可以根据需要添加进一步处理逻辑
        } catch (error) {
          console.error('上传失败:', error);
        }
      }
    },
    async handleFileUpload_fish(event) {
      const file = event.target.files[0];
      if (file) {
        const formData = new FormData();
        formData.append('file', file);

        try {
          const response = await axios.post('/api/fish-data/import_fish_file/', formData, {
            headers: {
              'Content-Type': 'multipart/form-data'
            }
          });
          console.log(response.data);
          // 可以根据需要添加进一步处理逻辑
        } catch (error) {
          console.error('上传失败:', error);
        }
      }
    },
    async exportData() {
      try {
        // 注意这里不要使用get, get的好处在于可以重命名文件,选择存放位置, 但是get是直接把viewset的queryset和serializer返回给前端,
        // 意味着前端还需要再处理这个json数据, 有点小麻烦, 在后端可以直接用pandas处理,比较方便
        // 所以这里使用post, 在后端处理好直接下载, 还可以直接导出多张表多个文件
        const response = await axios.post('/api/export/export_data/', {
          headers: {
              'Content-Type': 'multipart/form-data'
            },
          responseType: 'blob'
        });
        // const url = window.URL.createObjectURL(new Blob([response.data]));
        // const link = document.createElement('a');
        // link.href = url;
        // link.setAttribute('download', 'data.xlsx');
        // document.body.appendChild(link);
        // link.click();

        console.log(response.data);
      } catch (error) {
        console.error('导出失败:', error);
      }
    },
  },
};
</script>

<style lang="scss" scoped>
@import "../assets/scss/index.scss";

.header {
  width: 100%;
  padding: 0 10px;
}

.nav-bar {
  width: 100%;
  padding: 0 10px;
}

.nav-items {
  display: flex;
  justify-content: space-between;
}

.react-item {
  display: flex;
  align-items: center;
  justify-content: center;
  min-width: 120px; /* 统一宽度，具体数值可根据需求调整 */
  margin-right: 1rem;
}

.title {
  display: flex;
  align-items: center;
  justify-content: center;
  white-space: nowrap; /* 强制文本在一行内显示 */
  overflow: hidden; /* 防止文本溢出 */
  text-overflow: ellipsis; /* 添加省略号来处理溢出的文本 */
}

.title-text {
  font-size: 1.5rem; /* 根据需要调整字体大小 */
  color: #fff;
  margin: 0 10px; /* 根据需要调整左右间距 */
  line-height: 2rem; /* 增加行高，确保文本不会被遮挡 */
}

.mr-3 {
  background-color: #0f1325;
}
.bgc {
  background-color: #1a5cd7 !important;
}
.text {
  color: #fff;
}

.user-interface {
  margin-right: 20px; /* 调整与时间的间距 */
}

.justify-end {
  justify-content: flex-end;
}

.upload-button {
  //修改按钮大小,美化

  //margin-top: 20px;
  margin: 0px 4px;
  font-size: 18px;
  padding: 8px 4px;  // 增加内部空间
}

</style>