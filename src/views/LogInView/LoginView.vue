<script setup>
import Login from './components/Login.vue';
</script>

<template>
    <div class="div">
      <div class="div-9">로그인</div>
        <Login/>
    </div>
  </template>
  
  
  
  <style scoped>
  .div {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding-top: 0%;
  }
  .div-9 {
    text-shadow: 0px 4px 4px rgba(0, 0, 0, 0.25);
    text-align: center;
    -webkit-text-stroke-width: 1;
    -webkit-text-stroke-color: #000;
    border-color: rgba(0, 0, 0, 1);
    border-style: solid;
    border-width: 1px;
    justify-content: center;
    align-items: center;
    border-radius: 14px;
    box-shadow: 0px 4px 4px 0px rgba(0, 0, 0, 0.25);
    background-color: #fffefd;
    z-index: 10;
    margin-top: -19px;
    width: 586px;
    max-width: 100%;
    color: #434343;
    white-space: nowrap;
    padding: 12px 47px;
    font: 400 48px Actor, sans-serif;
  }
  @media (max-width: 991px) {
    .div-9 {
      font-size: 40px;
      white-space: initial;
      padding: 0 20px;
    }
  }
  </style>