<template>
  <div id="app">
    <!--
      <navbar />
    -->

    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'App',
};
</script>

<style>

*{
  transition: .15s;
}

body{
  background-color: rgb(250, 250, 250);
  margin: 0;
  margin-bottom: 100px;
  padding: 0;
  font-family: Helvetica Neue,Helvetica,sans-serif;
  font-weight: 400;
  -webkit-font-smoothing: antialiased;
}

p{
  margin: 0;
}

a{
  text-decoration: none;
  /*color: #039be5;*/
  color: #000;
}

.material-icons{
  font-size: 17px;
  vertical-align: text-top;
}

</style>
