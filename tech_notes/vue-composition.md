# Vue Composition API notes

## VueX

### Future of Vuex:

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

### Possible VueX integration

Mapper functions:

```js
import { computed, inject, reactive, toRefs } from "@vue/composition-api";
const StoreSymbol = Symbol.for("Vuex Store");
export function mapState(obj) {
  const store = inject(StoreSymbol);
  let output = {};
  if (Array.isArray(obj)) {
    obj.forEach(item => {
      output[item] = computed(() => store.state[item]);
    });
  } else {
    Object.entries(obj).forEach(([key, value]) => {
      if (typeof value === "function") output[key] = computed(value);
    });
  }
  return toRefs(reactive(output)); // reactive and then let vue do its magic
}

export function mapMutations(obj) {
  const store = inject(StoreSymbol);

  obj = Array.isArray(obj)
    ? obj.reduce((obj, item) => {
        obj[item] = args => store.commit(item, args);
        return obj;
      }, {})
    : Object.entries(obj).reduce((obj, [key, val]) => {
        obj[key] = args => store.commit(val, args);
        return obj;
      }, {});
  return obj;
}

export function mapActions(obj) {
  const store = inject(StoreSymbol);

  obj = Array.isArray(obj)
    ? obj.reduce((obj, item) => {
        obj[item] = arg => store.dispatch(item, arg);
        return obj;
      }, {})
    : Object.entries(obj).reduce((obj, [key, val]) => {
        obj[key] = arg => store.dispatch(val, arg);
        return obj;
      }, {});
  return obj;
}

export function mapGetters(obj) {
  const store = inject(StoreSymbol);

  obj = Array.isArray(obj)
    ? obj.reduce((obj, item) => {
        obj[item] = store.getters[item];
        return obj;
      }, {})
    : Object.entries(obj).reduce((obj, [key, val]) => {
        obj[key] = store.getters[val];
        return obj;
      }, {});
  return obj;
}
```

Store:

```js
const store = new Vuex.Store({
  state: {
    a: "hello",
    b: 0,
    c: "bar"
  },
  mutations: {
    incrementB(state) {
      state.b++;
    }
  }
});

new Vue({
  setup() {
    provide(Symbol.for("Vuex Store"), store);
  },
  store,
  render: h => h(App)
}).$mount("#app");
```
