---
title: 'Error Boundary をやってみたい'
emoji: '🔧'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: ['react', 'vue']
published: true
---

# はじめに

Error Boundary について [React 公式ドキュメント](https://ja.reactjs.org/docs/error-boundaries.html)で調べると、class component での使用方法が書いてあります。
今回 functional component での使用方法と、Vue で同じ事がしたいときについて調べたことをメモします。
下記それぞれフォールバック用の UI のレンダリングができることを確認済みです。
環境構築には vite を使用しています。

# react-error-boundary

Error Boundary を functional component でも使用できるライブラリです。

```
npm install react-error-boundary
```

エラー時のフォールバック用のコンポーネントを作成します。

```tsx: ErrorFallback.tsx
import React, { FC } from 'react'
import { FallbackProps } from 'react-error-boundary'

export const ErrorFallback: FC<FallbackProps> = ({
  error,
  resetErrorBoundary,
}) => {
  return (
    <div>
      <p>Error Message</p>
      <p>{error.message}</p>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  )
}
```

`App.tsx` で定期的に Error を throw します。

```tsx: App.tsx
export const App: FC = () => {
  const random = Math.random()
  if (random <= 0.6) {
    throw new Error('Error!')
  }
  return <div>OK</div>
}
```

`main.tsx` で `App.tsx` を `<ErrorBoundary></ErrorBoundary>` でラップします。

```tsx: main.tsx
import { App } from './App'
import { ErrorFallback } from './components/ErrorFallback'
import { ErrorBoundary } from 'react-error-boundary'

ReactDOM.render(
  <React.StrictMode>
    <ErrorBoundary FallbackComponent={ErrorFallback}>
      <App />
    </ErrorBoundary>
  </React.StrictMode>,
  document.getElementById('root')
)
```

正常時
![正常時](https://storage.googleapis.com/zenn-user-upload/b24901cb6878-20220228.png)

エラー時
![エラー時](https://storage.googleapis.com/zenn-user-upload/cefc8dc12252-20220228.png)

https://github.com/bvaughn/react-error-boundary

# vue-error-boundary

React の Error Boundary に基づいて作成された Vue のライブラリです。

```
npm install vue-error-boundary
```

同様に、エラー時のフォールバック用のコンポーネントを作成します。

```javascript: ErrorFallback.vue
<template>
  <div>
    <p>Error Message</p>
    <p>Try again</p>
  </div>
</template>
```

デフォルトで入っている `HelloWorld.vue` で定期的に Error を throw します。

```javascript: HelloWorld.vue
<script setup lang="ts">
const random = Math.random()
  if (random <= 0.6) {
    throw new Error('Error!')
  }
</script>

<template>
// 略
</template>
```

`App.vue` で `HelloWorld.vue` を `<VErrorBoundary></VErrorBoundary>` でラップします。

```javascript: App.vue
<script setup lang="ts">
import VErrorBoundary from 'vue-error-boundary'
import HelloWorld from './components/HelloWorld.vue'
import ErrorFallback from './components/ErrorFallback.vue'

// エラー時に実行するコールバック関数
const handleError = (err: any) => {
  console.log(err.message)
}
</script>

<template>
  <VErrorBoundary :fall-back="ErrorFallback" :on-error="handleError">
    <HelloWorld />
  </VErrorBoundary>
</template>
```

https://github.com/dillonchanis/vue-error-boundary

# おわりに

今回使用したコードは下記リポジトリにあります。
https://github.com/TakaShinoda/error-boundary-sample

# 参考記事

https://ja.reactjs.org/docs/error-boundaries.html
https://kudolog.net/posts/func-error-boundary/
