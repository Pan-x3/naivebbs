<template>
    <div class="container-body search-body">
        <div class="search-panel">
            <el-form :model="formData" ref="formDataRef" @submit.native.prevent class="searchForm">
                <!--input输入-->
                <el-form-item prop="keyword">
                    <el-input size="large" clearable v-model.trim="formData.keyword" @keyup.enter="search"
                         @change="changeInput" placeholder="请至少输入三个关键字">
                        <template #suffix>
                            <span class="iconfont icon-search" @click="search"
                                @blur="formData.keyword = $event.target.value.trim()"></span>
                        </template>
                    </el-input>
                </el-form-item>
            </el-form>
        </div>
        <div class="article-list">
        <DataList
          :dataSource="articleListInfo"
          :loading="loading"
          @loadData="search"
          noDataMsg="暂无文章"
        >
          <template #default="{ data }">
            <ArticleListItem :data="data" /> </template
        ></DataList>
      </div>
    </div>
</template>

<script setup>
import { ref, getCurrentInstance} from "vue";
const { proxy } = getCurrentInstance();
import ArticleListItem from "../forum/ArticleListItem.vue";
const api = {
    loadArticle: '/forum/search',
}
const formData = ref({})
const formDataRef = ref()
// const rules = {
//     keyword: [
//         { required: true, message: '请输入标题' },
//         { min: 3,  message: '请至少输入三个关键字'}
//     ],
// };
const loading = ref(false);
const articleListInfo = ref({});
const search = async () => {
  loading.value = true;
  let params = {
    pageNo: articleListInfo.value.pageNo,
    keyword: formData.value.keyword,
  };
  console.log(params)
  let result = await proxy.Request({
    url: api.loadArticle,
    params: params,
    showLoading: false,
  });
  loading.value = false;
  if (!result) {
    return;
  }
  articleListInfo.value = result.data;
};
const changeInput = () => {
    if(formData.value.keyword == ""){
        articleListInfo.value = {};
    }
};
</script>

<style lang="scss" scoped>
.container-body {
    width:1000px;
    background-color: #fff;
    padding: 30px;
    min-height: calc(100vh - 210px);
    border-radius: 5px;
    .search-panel {
        display: flex;
        justify-content: center;

        .el-input {
            width: 500px;
            transition: all 0.5s ease;
            span{
                cursor: pointer;
            }
        }
    }
}
@media screen and (max-width: 768px) {
    .container-body {
        width:95vw;
        .searchForm{
            width:100%;
        }
    }
}
</style>