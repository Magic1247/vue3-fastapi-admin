<script setup>
const currentEnv = ref('')
const fake_env_list = ref([
  {name: '阿里云', host: 'https://xxx.com'},
  {name: '本地环境', host: 'http://localhost/'}
])

function changeEnv() {
  console.log('使用VueX全局管理环境', currentEnv.value)
}


</script>

<template>

  <div>
    <el-select
        placeholder="环境选择"
        v-model="currentEnv"
        @change="changeEnv"
    >
      <el-option
          v-for="env in fake_env_list"
          :key="env.name"
          :label="env.name"
          :value="env.host"

      >
        {{ env.name }} {{ env.host }}
      </el-option>
    </el-select>
  </div>


</template>

<style scoped lang="scss">

</style>