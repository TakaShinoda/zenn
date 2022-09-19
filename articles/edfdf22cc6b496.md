---
title: 'next/router をモックして、locale で表示切り替えしているコンポーネントをテストする'
emoji: '🐧'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: ['jest','nextjs']
published: false
---

# はじめに
Next.js は i18n の機能をサポートしており、この機能を使う事で多言語でサイトを表示できます。

https://nextjs.org/docs/advanced-features/i18n-routing

今回は日本語 / 英語で表示を切り替えるコンポーネントを作成して、そのコンポーネントをテストします。


# 前提
Next.js のプロジェクトを作成し、jest が導入済であることを想定しています。
以下のように locale を取得し、表示を切り替えているコンポーネントをテストします。

```tsx: Profile.tsx
import { FC } from 'react'
import { useRouter } from 'next/router'
import { en } from '../i18n/en'
import { ja } from '../i18n/ja'

export const Profile: FC = () => {
  const { locale } = useRouter()
  const t = locale === 'en' ? en : ja
  
  return (
    <div>
      <h1>{t.name}</h1>
    </div>
  )
}
```

```ts: en.ts
export const en = {
  name: 'Bob',
}
```

```ts: ja.ts
export const ja = {
  name: 'ボブ',
}
```

```js: next.config.js
/** @type {import('next').NextConfig} */
module.exports = {
  i18n: {
    locales: ['en', 'ja'],
    defaultLocale: 'ja'
  }
}

```

また、以下の環境で動作確認しています。
```
"next": "^12.3.0",
"react": "18.2.0",
"@testing-library/react": "^13.4.0",
"jest": "^29.0.3",
"typescript": "4.8.3"
```

# テストする
以下のように next/router をモックしテストを行います。
```tsx: Profile.test.tsx
import React from 'react'
import { render, screen } from '@testing-library/react'
import { Profile } from '../components/Profile'

jest.mock('next/router', () => ({
  useRouter() {
    return {
      locale: 'ja',
    }
  },
}))

describe('日本語での表示テスト', () => {
  test('名前が表示されているか', () => {
    render(<Profile />)
    expect(screen.getByRole('heading')).toHaveTextContent('ボブ')
  })
})
```

日本語の表示をテストしたいときは、`locale: 'ja'` に設定し、英語での表示をテストしたいときは、`locale: 'en'` にすることで表示内容のテストを行えるようになりました。
