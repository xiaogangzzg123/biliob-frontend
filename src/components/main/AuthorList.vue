<template>
  <div class="author-list-main">
    <div>
      <div>
        <BiliobCard class="card-tabs">
          <VLayout slot="header">
            <VIcon right>mdi-sort</VIcon>
            <VFlex>
              <VTabs show-arrows color="transparent" slider-color="white">
                <VTab @click="sortChange(0)">
                  <VIcon>
                    mdi-account-heart
                  </VIcon>
                  <div style="margin-left:10px">
                    粉丝总数
                  </div>
                </VTab>
                <VTab @click="sortChange(1)">
                  <VIcon>
                    mdi-play-circle-outline
                  </VIcon>
                  <div style="margin-left:10px">
                    播放总量
                  </div>
                </VTab>
                <VTab @click="sortChange(2)">
                  <VIcon>
                    mdi-script-text-outline
                  </VIcon>
                  <div style="margin-left:10px">
                    专栏阅读
                  </div>
                </VTab>
              </VTabs>
            </VFlex>
          </VLayout>
          <div>
            <VSearchForm
              slot="search"
              hint="请输入UP主名称，或者uid"
              @getSearchValue="getSearchValue"
            />
          </div>
          <VSlideYTransition group>
            <VCard
              v-for="eachAuthor in authorList.content"
              :key="eachAuthor.mid"
              ripple
              :to="`/author/${eachAuthor.mid}`"
              class="author-cards"
            >
              <div style="padding:5px;display:flex">
                <div>
                  <VImg
                    class="author-face"
                    :src="zipPic(eachAuthor.face.replace('http:', ''))"
                    :lazy-src="zipPic(eachAuthor.face.replace('http:', ''))"
                  />
                </div>
                <div style="margin-left:10px;width:100%">
                  <div class="font-weight-bold author-title">
                    {{ eachAuthor.name }}
                    <SexIcon :sex="eachAuthor.sex" />
                  </div>
                  <div
                    v-if="eachAuthor.official !== ''"
                    class="caption  author-info"
                  >
                    <VIcon color="#FBC02D" small> mdi-flash-circle </VIcon
                    ><span style="vertical-align: middle">
                      {{ eachAuthor.official }}</span
                    >
                  </div>
                  <ObserveStatus class="observe-status" :object="eachAuthor" />
                </div>
              </div>
              <VDivider></VDivider>
            </VCard>
          </VSlideYTransition>
          <VBtn
            v-if="!notFound"
            block
            outlined
            style="border-width:1px"
            color="primary darken-2"
            :disabled="nextBtnDisabled"
            tile
            @click.stop="next"
            >{{ nextBtnText }}</VBtn
          >
          <div v-else>
            <h4 class="primary--text text--darken-2">
              <VIcon class="primary--text text--darken-2">mdi-ship-wheel</VIcon
              >抱歉！什么都没有找到QwQ
            </h4>
            <p>
              搜索功能可能并不完善，为了精确搜索请在上方输入相关UP主的ID！
            </p>
            <p>
              如果搜索ID仍然没有结果，可能是因为该UP主并未被本站观测。你可以点击页面右下角的圆形按钮进行添加！
            </p>
          </div>
        </BiliobCard>
      </div>
    </div>
  </div>
</template>

<script>
import VSearchForm from "../common/VSearchForm.vue";
import ObserveStatus from "../common/ObserveStatus.vue";
import SexIcon from "../common/SexIcon.vue";
export default {
  name: "AuthorList",
  components: {
    VSearchForm,
    SexIcon,
    ObserveStatus
  },
  data() {
    return {
      sort: 0,
      authorList: [],
      currentApiurl: String(),
      currentPage: 0,
      text: String(),
      nextBtnText: "请给我更多...",
      nextBtnDisabled: false,
      requestUrl: String(),
      notFound: false
    };
  },
  watch: {
    text: function() {
      this.currentPage = 0;
      this.axios
        .get(
          this.currentApiurl +
            "?page=" +
            this.currentPage +
            "&text=" +
            this.text +
            "&sort=" +
            this.sort
        )
        .then(response => {
          this.authorList.content = response.data.content;
          if (this.authorList.content.length == 0) {
            this.notFound = true;
          } else {
            this.notFound = false;
          }
        });
    },
    currentPage: function changePage(page) {
      this.axios
        .get(
          `${this.currentApiurl}?page=${page}&text=${this.text}&sort=${this.sort}`
        )
        .then(response => {
          // 判断是否为最后一页
          if (response.data.last) {
            this.nextBtnText = "没有更多了";
            this.nextBtnDisabled = true;
          }
          response.data.content.forEach(e => {
            this.authorList.content.push(e);
          });
        });
    }
  },
  created() {
    this.currentApiurl = "/author";
    this.axios.get(this.currentApiurl).then(response => {
      this.refreshList(response);
    });
  },
  methods: {
    refreshList(response) {
      this.authorList = response.data;
      // 判断是否为最后一页
      if (response.data.last) {
        this.nextBtnText = "没有更多了";
        this.nextBtnDisabled = true;
      }
    },
    onScroll() {
      var scrollTop =
        document.documentElement.scrollTop || document.body.scrollTop;
      var windowHeight =
        document.documentElement.clientHeight || document.body.clientHeight;
      var scrollHeight =
        document.documentElement.scrollHeight || document.body.scrollHeight;
      if (scrollTop + windowHeight == scrollHeight) {
        // this.currentPage += 1;
      }
    },
    next() {
      this.currentPage += 1;
    },
    getSearchValue(value) {
      this.text = value;
    },
    handleChoosed(index, row) {
      this.$router.push({
        path: "/author/" + row.mid + "/author/" + row.aid
      });
    },
    sortChange(sort) {
      this.sort = sort;
      this.currentPage = 0;
      this.axios
        .get(
          `${this.currentApiurl}?page=${this.currentPage}&text=${this.text}&sort=${this.sort}`
        )
        .then(response => {
          this.refreshList(response);
        });
    },
    toAuthorDetail(mid) {
      this.$router.push("/author/" + mid);
    }
  }
};
</script>

<style>
.face {
  position: relative;
  align-content: center;
  border-radius: 4px;
}

.el-table td {
  padding: 2px 0;
}

.author-cards {
  margin: 10px 0px;
  padding: 0px 0px;
  border-radius: 5px;
  position: relative;
}

.author-title {
  max-width: 50vw;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.observe-status {
  position: absolute;
  bottom: 5px;
  right: 5px;
}

.author-info {
  margin-top: 5px;
}

.author-face {
  border-radius: 40px;
  width: 80px;
  height: 80px;
}
</style>
