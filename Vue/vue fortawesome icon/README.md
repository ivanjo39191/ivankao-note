# vue fortawesome icon

Created: February 1, 2022 5:22 PM
Tags: Vue

[https://www.npmjs.com/package/@fortawesome/vue-fontawesome](https://www.npmjs.com/package/@fortawesome/vue-fontawesome)

npm install

```python
$ npm i --save @fortawesome/fontawesome-svg-core
$ npm i --save @fortawesome/free-solid-svg-icons
$ npm i --save @fortawesome/free-brands-svg-icons
$ npm i --save @fortawesome/free-regular-svg-icons

# Using Vue 2.x
$ npm i --save @fortawesome/vue-fontawesome@latest
# Using Vue 3.x
$ npm i --save @fortawesome/vue-fontawesome@prerelease

```

main.js

```jsx
# main.js

import { library } from '@fortawesome/fontawesome-svg-core'
import { faLongArrowAltDown } from '@fortawesome/free-solid-svg-icons' //你要使用的 icon
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'

library.add(faLongArrowAltDown);
Vue.component('font-awesome-icon', FontAwesomeIcon)
```

html

```html
<font-awesome-icon icon="long-arrow-alt-down" />
```