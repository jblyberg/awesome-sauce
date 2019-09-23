# Vue Composition API notes

## VueX options

### From discord:

There's nothing special going on except that you can now essentially write Vuex-like modules with state, getters (`computed()`) and actions with the composition API and share it in the App (or a prt of the component tree) with `provide()` and `inject()`. IMHO, the only thing left for vuex to do is to allow multiple modules to interact and integrate with devtools.

```js
import { reactive, computed, provide, inject } from 'vue'
const state = reactive({
  users: []

})

const getters = {
    numberofUsers: computed(() => state.users.length)
}

function addUser (user) {
  state.users.push(user)
}

export const key = Symbol('UserStore')
export provideUserStore = () => provide(key, {
  state,
  getters,
  actions: {
    addUser
  }
})

export const injectUserStore  = () => inject(key)
```

a parent would then just do:

```js
setup() {
  useUserStore()
}
```

and any descendants would do:

```js
setup() {
  return {
    users: injectUserStore()
  }
}
```

Some of that (already small) boilerplate could be put into a factory function and you basically have a DIY Vuex
