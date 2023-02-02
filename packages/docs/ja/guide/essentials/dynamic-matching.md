# Params を用いた動的なルートマッチング {#dynamic-route-matching-with-params}

<VueSchoolLink
  href="https://vueschool.io/lessons/dynamic-routes"
  title="Learn about dynamic route matching with params"
/>

多くの場合、指定されたパターンのルートを同じコンポーネントにマップする必要があります。たとえば、すべてのユーザーに対してレンダリングする必要があるが、ユーザー ID が異なる `User` コンポーネントがあるとします。Vue Router では、パスで動的セグメントを使用してそれを実現できます。これを _param_ と呼びます:

```js
const User = {
  template: '<div>User</div>',
}

// these are passed to `createRouter`
const routes = [
  // dynamic segments start with a colon
  { path: '/users/:id', component: User },
]
```

これで、`/users/johnny` と `/users/jolyne` のような URL は、どちらも同じルートにマッピングされるようになります。

_param_ はコロン `:` で表します。ルートがマッチすると、その _params_ の値が `this.$route.params` として各コンポーネントで公開されます。したがって、`User` のテンプレートをこのように更新することで、現在のユーザー ID をレンダリングすることができます:

```js
const User = {
  template: '<div>User {{ $route.params.id }}</div>',
}
```

同じルートに複数の _params_ を指定することができ、それらは `$route.params` の対応するフィールドに対応します。例:

| pattern                        | matched path             | \$route.params                           |
| ------------------------------ | ------------------------ | ---------------------------------------- |
| /users/:username               | /users/eduardo           | `{ username: 'eduardo' }`                |
| /users/:username/posts/:postId | /users/eduardo/posts/123 | `{ username: 'eduardo', postId: '123' }` |

`$route` オブジェクトは、`$route.params` に加えて `$route.query` (URL にクエリがある場合)、 `$route.hash` などの有用な情報を公開しています。詳細については、[API リファレンス](../../api/#routelocationnormalized) で確認することができます。

この例の動作デモは、[こちら](https://codesandbox.io/s/route-params-vue-router-examples-mlb14?from-embed&initialpath=%2Fusers%2Feduardo%2Fposts%2F1) でご覧いただけます。

<!-- <iframe
  src="https://codesandbox.io/embed//route-params-vue-router-examples-mlb14?fontsize=14&theme=light&view=preview&initialpath=%2Fusers%2Feduardo%2Fposts%2F1"
  style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
  title="Route Params example"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
  sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe> -->

## Params の変更への対応 {#reacting-to-params-changes}

<VueSchoolLink
  href="https://vueschool.io/lessons/reacting-to-param-changes"
  title="Learn how to react to param changes"
/>

params を使ったルートを使う場合の注意点として、ユーザーが `/users/johnny` から `/users/jolyne` に移動する際に、**同じコンポーネントのインスタンスが再利用されることが挙げられます**。どちらのルートも同じコンポーネントをレンダリングするので、古いインスタンスを破棄して新しいインスタンスを作成するよりも効率的です。**ただし、これはコンポーネントのライフサイクルフックが呼び出されないことも意味します**。

同じコンポーネントの paramsの 変更に反応するには、単に `$route` オブジェクト上の何か、このシナリオでは `$route.params` を監視すればよいです:

```js
const User = {
  template: '...',
  created() {
    this.$watch(
      () => this.$route.params,
      (toParams, previousParams) => {
        // react to route changes...
      }
    )
  },
}
```

または、ナビゲーションをキャンセルすることもできる `beforeRouteUpdate` [ナビゲーションガード](../advanced/navigation-guards.md) を使用します:

```js
const User = {
  template: '...',
  async beforeRouteUpdate(to, from) {
    // react to route changes...
    this.userData = await fetchUser(to.params.id)
  },
}
```

## Catch all / 404 Not found Route {#catch-all-404-not-found-route}

<VueSchoolLink
  href="https://vueschool.io/lessons/404-not-found-page"
  title="Learn how to make a catch all/404 not found route"
/>

通常のパラメーターは、`/` で区切られた url フラグメント間の文字のみに一致します。**何か** に一致させたい場合は、_param_ の直後に括弧内に正規表現を追加することで、カスタム param 正規表現を使用できます:

```js
const routes = [
  // will match everything and put it under `$route.params.pathMatch`
  { path: '/:pathMatch(.*)*', name: 'NotFound', component: NotFound },
  // will match anything starting with `/user-` and put it under `$route.params.afterUser`
  { path: '/user-:afterUser(.*)', component: UserGeneric },
]
```

このシナリオでは、括弧の間に [カスタム正規表現](./route-matching-syntax.md#custom-regex-in-params) を使用し、`pathMatch` パラメータを [オプションで反復可能](./route-matching-syntax.md#optional-parameters) であるとマークしています。これにより、`path` を配列に分割することで、必要に応じてルートに直接ナビゲートすることができます:

```js
this.$router.push({
  name: 'NotFound',
  // preserve current path and remove the first char to avoid the target URL starting with `//`
  params: { pathMatch: this.$route.path.substring(1).split('/') },
  // preserve existing query and hash if any
  query: this.$route.query,
  hash: this.$route.hash,
})
```

詳しくは [repeated params](./route-matching-syntax.md#repeatable-params) の項をご覧ください。

[履歴モード](./history-mode.md) を使用する場合は、必ず指示に従って、サーバーも正しく設定してください。

## 高度なマッチングパターン {#advanced-matching-patterns}

Vue Router は、`express` で使われている構文にヒントを得て、独自のパスマッチング構文を使用しています。そのため、オプションのパラメータ、ゼロ以上 / 1 以上の要件、さらにはカスタム正規表現パターンなど、多くの高度なマッチングパターンがサポートされています。[Advanced Matching](./route-matching-syntax.md) のドキュメントを参照してください。
