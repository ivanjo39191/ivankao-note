# vue router transition

Created: February 1, 2022 5:22 PM
Tags: Vue

template 區塊

```python
<transition name="fade">
  <router-view></router-view>
</transition>
```

style 區塊

```python
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
```

參考:

[https://vuejs.org/v2/guide/transitions.html](https://vuejs.org/v2/guide/transitions.html)

[https://router.vuejs.org/guide/advanced/transitions.html#per-route-transition](https://router.vuejs.org/guide/advanced/transitions.html#per-route-transition)