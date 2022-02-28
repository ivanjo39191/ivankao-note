# vue 使用 google font

Created: February 1, 2022 5:22 PM
Tags: Vue

Just figured it out, I have to import it like so:

```json
<style lang="scss">
@import url(//fonts.googleapis.com/css?family=Noto+Sans+TC&display=swap);

#app {
  font-family: 'Noto Sans TC', Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  /* margin-top: 60px; */
}
</style>
```

[https://stackoverflow.com/questions/55403412/how-to-use-google-web-fonts-in-vuejs-app](https://stackoverflow.com/questions/55403412/how-to-use-google-web-fonts-in-vuejs-app)