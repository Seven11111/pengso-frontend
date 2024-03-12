<template>
  <div class="index-page">
    <div style="margin-left: 45%; margin-bottom: 12px">
      <h2>P 搜索</h2>
    </div>
    <a-divider />
    <a-auto-complete
      v-model:value="searchText"
      :options="options"
      style="width: 90%"
      placeholder="input here"
      @select="onSelect"
      @search="onSearch"
      @keyup.enter="onSearch"
    />
    <a-button
      type="primary"
      style="left-margin: 2px; width: 10%"
      @click="onSearch"
      >搜索
    </a-button>
    <!--    <a-input-search-->
    <!--      v-model:value="searchText"-->
    <!--      placeholder="input search text"-->
    <!--      enter-button="Search"-->
    <!--      size="large"-->
    <!--      @search="onSearch"-->
    <!--    />-->

    <MyDivider />
    <a-tabs v-model:activeKey="activeKey" @change="onTabChange">
      <a-tab-pane key="post" tab="文章">
        <PostList :postList="postList" />
      </a-tab-pane>
      <a-tab-pane key="picture" tab="图片">
        <PictureList :picture-list="pictureList" />
      </a-tab-pane>
      <a-tab-pane key="user" tab="用户" force-render>
        <UserList :user-list="userList" />
      </a-tab-pane>
    </a-tabs>
  </div>
</template>

<script setup lang="ts">
import { ref, watchEffect } from "vue";
import PostList from "@/components/PostList.vue";
import UserList from "@/components/UserList.vue";
import PictureList from "@/components/PictureList.vue";
import MyDivider from "@/components/MyDivider.vue";
import { useRoute, useRouter } from "vue-router";
import myAxios from "../../plugins/myAxios";
import { message } from "ant-design-vue";
import _ from "lodash";

const router = useRouter();
const route = useRoute();
const activeKey = route.params.category || "post";
const initSearchParams = {
  text: "",
  type: activeKey,
  pageSize: 10,
  pageNum: 1,
};
const searchParams = ref(initSearchParams);
const postList = ref([]);
const userList = ref([]);
const pictureList = ref([]);
const searchText = ref(route.query.text || "");
const options = ref<any[]>([]);

/**
 * 加载数据
 * @param params
 */
const loadAll = (params: any) => {
  const query = {
    ...params,
    searchText: params.text,
  };
  myAxios.post("/search/search/all", query).then((res: any) => {
    console.log(res);
    pictureList.value = res.pictureList;
    userList.value = res.userList;
    postList.value = res.postList;
  });
};

/**
 * 加载单类数据
 * @param params
 */

const load = (params: any) => {
  const { type } = params;
  if (!type) {
    message.error("类别为空");
    return;
  }
  const query = {
    ...params,
    searchText: params.text,
  };
  myAxios.post("/search/search/all", query).then((res: any) => {
    console.log(res);
    if (type === "post") {
      postList.value = res.dataList;
    } else if (type === "user") {
      userList.value = res.dataList;
    } else if (type === "picture") {
      pictureList.value = res.dataList;
    }
  });
};

const prompts = (params: any) => {
  const { type } = params;
  if (!type) {
    message.error("类别为空");
    return;
  }
  const query = {
    ...params,
    searchText: params.text,
  };
  options.value = [];
  myAxios.post("/search/search/prompt", query).then((res: any) => {
    console.log("res =>", res);
    if (res) {
      for (let i = 0; i < res.length; i++) {
        const tempMap = {
          value: res[i],
        };
        options.value.push(tempMap);
      }
    }

    console.log("opstions =>", options);
  });
};
const debounce_load = _.debounce(load, 500, { maxWait: 10000 });
const debounce_prompts = _.debounce(prompts, 500, { maxWait: 1000 });
// 首次搜索
// load(initSearchParams);

watchEffect(() => {
  searchParams.value = {
    ...initSearchParams,
    text: route.query.text,
    type: route.params.category || activeKey,
  } as any;
  debounce_load(searchParams.value);
  debounce_prompts(searchParams.value);
});

const onSelect = (value: string) => {
  searchText.value = value;
};

const onSearch = (value: string) => {
  // alert(value);
  router.push({
    query: {
      ...searchParams.value,
      text: searchText.value,
    },
  });
};

const onTabChange = (key: string) => {
  router.push({
    path: `/${key}`,
    query: searchParams.value,
  });
};
</script>
