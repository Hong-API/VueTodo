<template>
  <div class="flex justify-start flex-col bg-gray-300 bg min-h-screen">
    <!-- header -->
    <div class="flex w-full justify-center space-x-5 h-[50px]  background001 relative">
      <div class="text-2xl items-center flex h-full text-white ">
        <span>Assignment</span>
      </div>
    </div>
    <div class="py-2 w-full">
      <router-view></router-view>
    </div>
  </div>
</template>
