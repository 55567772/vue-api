<script setup>
  import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <header>
    <el-menu
        default-active="0"
        class="el-menu-demo"
        mode="horizontal"
        :ellipsis="false"
        @select=""
    >
      <el-menu-item index="0"><RouterLink to="/">首页</RouterLink></el-menu-item>
      <div class="flex-grow" />
      <el-menu-item index="1"><RouterLink to="/">文案API</RouterLink></el-menu-item>
      <el-menu-item index="2"><RouterLink to="/music-api">免费音乐API</RouterLink></el-menu-item>
    </el-menu>
  </header>
  <RouterView />
</template>

<script>
//import { ref } from 'vue'

export default {
  name: "app",
  setup(){
    //const activeIndex = ref('0')
    // const handleSelect = (key: string, keyPath: string[]) => {
    //   console.log(key, keyPath)
    // }
    return {

    }
  }

}

</script>

<style>
.flex-grow {
  flex-grow: 1;
}
li a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
  text-decoration:none;
}

</style>
