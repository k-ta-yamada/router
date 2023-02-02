# はじめに {#getting-started}

<VueSchoolLink
  href="https://vueschool.io/courses/vue-router-4-for-everyone"
  title="Learn how to build powerful Single Page Applications with the Vue Router on Vue School"
>Watch a Free Vue Router Video Course</VueSchoolLink>

Vue + Vue Router を使用した単一ページ アプリケーションの作成は自然に感じられます: Vue.js では、すでにコンポーネントでアプリケーションを構成しています。Vue Router をミックスに加える際に必要なのは、コンポーネントをルートにマッピングして、Vue Router にそれらをレンダリングする場所を知らせることです。以下は基本的な例です:

## HTML {#html}

```html
<script src="https://unpkg.com/vue@3"></script>
<script src="https://unpkg.com/vue-router@4"></script>

<div id="app">
  <h1>Hello App!</h1>
  <p>
    <!-- use the router-link component for navigation. -->
    <!-- specify the link by passing the `to` prop. -->
    <!-- `<router-link>` will render an `<a>` tag with the correct `href` attribute -->
    <router-link to="/">Go to Home</router-link>
    <router-link to="/about">Go to About</router-link>
  </p>
  <!-- route outlet -->
  <!-- component matched by the route will render here -->
  <router-view></router-view>
</div>
```

### `router-link` {#router-link}

通常の `a` タグを使用する代わりに、カスタムコンポーネントの `router-link` を使用してリンクを作成していることに注意してください。これにより、Vue Router は、ページを再読み込みせずに URL を変更し、URL の生成とそのエンコーディングを処理することができます。これらの機能をどのように利用するかは、後で説明します。

### `router-view` {#router-view}

`router-view` は、url に対応するコンポーネントを表示します。レイアウトに合わせるために、どこにでも置くことができます。

<VueMasteryLogoLink></VueMasteryLogoLink>

## JavaScript {#javascript}

```js
// 1. ルートコンポーネントを定義します。
// これらは、他のファイルからインポートすることができます
const Home = { template: '<div>Home</div>' }
const About = { template: '<div>About</div>' }

// 2. いくつかのルートを定義します
// 各ルートは、コンポーネントにマッピングする必要があります。
// ネストされたルートについては、後で説明します。
const routes = [
  { path: '/', component: Home },
  { path: '/about', component: About },
]

// 3. ルーターインスタンスを作成し、`routes` オプションを渡します
// ここで追加のオプションを渡すことができますが、
// 今はシンプルにしておきましょう。
const router = VueRouter.createRouter({
  // 4. 使用する履歴の実装を提供します。ここでは簡単のため、ハッシュ履歴を使用します。
  history: VueRouter.createWebHashHistory(),
  routes, // short for `routes: routes`
})

// 5. ルートインスタンスを作成し、マウントします。
const app = Vue.createApp({})
// アプリ全体をルーター対応にするために、
// 必ずルーター インスタンスを _使用_ してください。
app.use(router)

app.mount('#app')

// Now the app has started!
```

`app.use(router)` を呼び出すと、`this.$router` として、また、任意のコンポーネント内で `this.$route` として現在のルートにアクセスすることができます:

```js
// Home.vue
export default {
  computed: {
    username() {
      // `params` が何であるかはすぐにわかります
      return this.$route.params.username
    },
  },
  methods: {
    goToDashboard() {
      if (isAuthenticated) {
        this.$router.push('/dashboard')
      } else {
        this.$router.push('/login')
      }
    },
  },
}
```

`setup` 関数内部でルーターやルートにアクセスするには、`useRouter` 関数や `useRoute` 関数を呼び出します。これについては [Composition API](./advanced/composition-api.md#accessing-the-router-and-current-route-inside-setup) で詳しく説明します。

このドキュメントでは、しばしば `router` インスタンスを使用します。`this.$router` は、`createRouter` で作成した `router` インスタンスを直接使用するのとまったく同じだということを覚えておいてください。`this.$router` を使用する理由は、ルーティングを操作する必要のあるコンポーネントでいちいちルーターをインポートしたくないからです。
