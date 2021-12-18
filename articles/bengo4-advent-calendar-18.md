---
title: 'Supabaseで認証機能を作ってみる'
emoji: '🦌'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: ['supabase', 'nextjs', 'adventcalendar2021']
published: true
---

# はじめに

:::message
この記事は [弁護士ドットコム Advent Calendar 2021](https://qiita.com/advent-calendar/2021/bengo4com) の 18 日目の記事です。
:::

前日は、[@komtaki](https://qiita.com/komtaki) さんの
「[GitHub Actions と semantic-release で、自動で高品質 npm パッケージを一般公開する](https://qiita.com/komtaki/items/d89451aea1b146ca83d6)」
でした！

こんにちは。
クラウドサイン事業本部 プロダクト部の[@tttttt_621_s](https://twitter.com/tttttt_621_s)です。

私は、個人開発の際に Firebase を使うことがあるのですが、
最近 Supabase というものを知り、この機会にさわってみました。

また、並行して Zenn への投稿も初めてやってみました。
よろしくお願いします。

# 今回のゴール

[Quickstart: Next.js](https://supabase.com/docs/guides/with-nextjs)を参照して、実際に Supabase のプロジェクトを作成し、
Next.js を使って認証機能を作成します。

![ログインページ](https://storage.googleapis.com/zenn-user-upload/67e1e77b7bfb-20211218.png)

# Supabase とは

いわゆる BaaS で、公式サイトでは、Firebase Alternative と謳ってあります。
具体的には下記の機能があります。

- Database(PostgreSQL)
- Authentication
- Storage
- Functions(Comming soon) ※2021/12/18 時点

https://supabase.com/

また、11/29 から 5 日間 [Supabase Launch Week III: Holiday Special](https://supabase.com/blog/2021/11/26/supabase-launch-week-the-trilogy) が行われていました。

- Monday: Community Day
- Tuesday: Dashboard Surprise
- Wednesday: Realtime Updates
- Thursday: A new player enters the game
- Friday: (One more thing)

Supabase 構成するコミュニティとそれらのアップデートの紹介、新機能の概要、買収などがありました。

# 環境構築

## Supabase に登録・プロジェクト作成

公式サイトから「Start your project」を選択します。

![画像1](https://storage.googleapis.com/zenn-user-upload/9e14030a059a-20211204.png)

GitHub でログイン後、「New Project」を選択してプロジェクトを新規作成します。

![画像2](https://storage.googleapis.com/zenn-user-upload/87faa5d13928-20211204.png)

![画像3](https://storage.googleapis.com/zenn-user-upload/e6dd877255f2-20211204.png)

プロジェクト名とパスワードを入力し、リージョンを指定します。 (Tokyo も存在します。)

![画像4](https://storage.googleapis.com/zenn-user-upload/143ee560077f-20211204.png)

## データベーススキーマの設定

今回は「User Management Starter」のクイックスタートを使用します。

![User Management Starter](https://storage.googleapis.com/zenn-user-upload/75dc4c0a38ed-20211211.png)

「RUN」を押下してクエリを実行します。

![クエリ実行](https://storage.googleapis.com/zenn-user-upload/9ddd5a7db4d8-20211211.png)

しばらく待つとテーブルが作成されました。

![テーブル](https://storage.googleapis.com/zenn-user-upload/20bbeb38bd85-20211211.png)

## API キーの取得

「設定」->「API」から Config URL と API keys を確認します。

![API setting](https://storage.googleapis.com/zenn-user-upload/2808f68f4684-20211212.png)

## Next.js で新規プロジェクト作成

`create-next-app`を使って、プロジェクトを新規作成します。

```
npx create-next-app@latest --typescript
```

さらに、supabase-js をインストールします。

```bash
npm install @supabase/supabase-js
```

https://github.com/supabase/supabase-js

Next.js のプロジェクト配下に`.env.local`ファイルを作成し、先程の Config URL と API keys を記述します。

```text: .env.local
# Config URL
NEXT_PUBLIC_SUPABASE_URL=https://XXXXXXXXXXXXXXXXXXXX.supabase.co
# API keys
NEXT_PUBLIC_SUPABASE_ANON_KEY=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

Supabase のクライアントを初期化するためのファイルです。
`process.env.NEXT_PUBLIC_XXXXXX`で環境変数に入れた値を呼び出します。

```ts: supabaseClient.ts
import { createClient } from '@supabase/supabase-js'

const supabaseUrl= process.env.NEXT_PUBLIC_SUPABASE_URL as string
const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY as string

export const supabase = createClient(supabaseUrl, supabaseAnonKey)
```

# 機能作成

## サインイン

ユーザーは E メールまたは OAuth のいずれかでサインインできます。
パスワードなしでメールを提供した場合、ユーザーにはマジックリンクが送信されます。(今回はこれでサインインします。)
デフォルトでは、60 秒に 1 回マジックリンクを要求できます。

```tsx: Auth.tsx
const { error } = await supabase.auth.signIn({ email })
```

:::details パスワードありの場合

```tsx: Auth.tsx
const { error } = await supabase.auth.signIn({
  email: 'example@email.com',
  password: 'example-password',
})
```

:::

実際にデフォルトでは、下記のようなメールが送られてきました。
![マジックリンクのメール](https://storage.googleapis.com/zenn-user-upload/9e1dd99cdf7e-20211206.png)

メールの内容も設定画面から変更できそうです。

![メール内容変更](https://storage.googleapis.com/zenn-user-upload/6572b3a1157a-20211215.png)

:::message
`Update config failed: An error has occured: Failed to fetch`と表示される場合[^1]
:::

### サードパーティのプロバイダ

サードパーティのプロバイダを利用したサインインは主に下記をサポートしています。(一部抜粋)

- GitHub
- Google
- GitLab
- Bitbucket
- Twitter

https://supabase.com/docs/guides/auth/intro#authentication

## データベースの型情報ファイルの生成

openapi-typescript を使用して、型情報のファイルを生成できます。
生成されたファイルは DB と同期はされていないので DB を変更した際は再生成が必要です。

```
npx openapi-typescript https://your-project.supabase.co/rest/v1/?apikey=your-anon-key --output types/supabase.ts
```

:::details 今回実際に生成されたファイル

```ts: types/supabase.ts
/**
 * This file was auto-generated by openapi-typescript.
 * Do not make direct changes to the file.
 */

export interface paths {
  "/": {
    get: {
      responses: {
        /** OK */
        200: unknown;
      };
    };
  };
  "/profiles": {
    get: {
      parameters: {
        query: {
          id?: parameters["rowFilter.profiles.id"];
          updated_at?: parameters["rowFilter.profiles.updated_at"];
          username?: parameters["rowFilter.profiles.username"];
          avatar_url?: parameters["rowFilter.profiles.avatar_url"];
          website?: parameters["rowFilter.profiles.website"];
          /** Filtering Columns */
          select?: parameters["select"];
          /** Ordering */
          order?: parameters["order"];
          /** Limiting and Pagination */
          offset?: parameters["offset"];
          /** Limiting and Pagination */
          limit?: parameters["limit"];
        };
        header: {
          /** Limiting and Pagination */
          Range?: parameters["range"];
          /** Limiting and Pagination */
          "Range-Unit"?: parameters["rangeUnit"];
          /** Preference */
          Prefer?: parameters["preferCount"];
        };
      };
      responses: {
        /** OK */
        200: {
          schema: definitions["profiles"][];
        };
        /** Partial Content */
        206: unknown;
      };
    };
    post: {
      parameters: {
        body: {
          /** profiles */
          profiles?: definitions["profiles"];
        };
        query: {
          /** Filtering Columns */
          select?: parameters["select"];
        };
        header: {
          /** Preference */
          Prefer?: parameters["preferReturn"];
        };
      };
      responses: {
        /** Created */
        201: unknown;
      };
    };
    delete: {
      parameters: {
        query: {
          id?: parameters["rowFilter.profiles.id"];
          updated_at?: parameters["rowFilter.profiles.updated_at"];
          username?: parameters["rowFilter.profiles.username"];
          avatar_url?: parameters["rowFilter.profiles.avatar_url"];
          website?: parameters["rowFilter.profiles.website"];
        };
        header: {
          /** Preference */
          Prefer?: parameters["preferReturn"];
        };
      };
      responses: {
        /** No Content */
        204: never;
      };
    };
    patch: {
      parameters: {
        query: {
          id?: parameters["rowFilter.profiles.id"];
          updated_at?: parameters["rowFilter.profiles.updated_at"];
          username?: parameters["rowFilter.profiles.username"];
          avatar_url?: parameters["rowFilter.profiles.avatar_url"];
          website?: parameters["rowFilter.profiles.website"];
        };
        body: {
          /** profiles */
          profiles?: definitions["profiles"];
        };
        header: {
          /** Preference */
          Prefer?: parameters["preferReturn"];
        };
      };
      responses: {
        /** No Content */
        204: never;
      };
    };
  };
}

export interface definitions {
  profiles: {
    /**
     * Note:
     * This is a Primary Key.<pk/>
     */
    id: string;
    updated_at?: string;
    username?: string;
    avatar_url?: string;
    website?: string;
  };
}

export interface parameters {
  /** Preference */
  preferParams: "params=single-object";
  /** Preference */
  preferReturn: "return=representation" | "return=minimal" | "return=none";
  /** Preference */
  preferCount: "count=none";
  /** Filtering Columns */
  select: string;
  /** On Conflict */
  on_conflict: string;
  /** Ordering */
  order: string;
  /** Limiting and Pagination */
  range: string;
  /** Limiting and Pagination */
  rangeUnit: string;
  /** Limiting and Pagination */
  offset: string;
  /** Limiting and Pagination */
  limit: string;
  /** profiles */
  "body.profiles": definitions["profiles"];
  "rowFilter.profiles.id": string;
  "rowFilter.profiles.updated_at": string;
  "rowFilter.profiles.username": string;
  "rowFilter.profiles.avatar_url": string;
  "rowFilter.profiles.website": string;
}

export interface operations {}

export interface external {}

```

:::

## サインインしたユーザーの情報表示

生成した型情報ファイルをインポートして from メソッドの型パラメータとして渡します。
profiles テーブルから、カラム( username, website, avatar_url)の値を、ログインしているユーザと id が同じものを 1 つ取得します。

```tsx: Account.tsx
import { definitions } from '../../types/supabase'

// ログインしているユーザがいればそのユーザデータを返す
const user = supabase.auth.user()

let { data, error, status } = await supabase
    .from<definitions['profiles']>('profiles')
    .select(`username, website, avatar_url`)
    .eq('id', user.id)
    .single()
```

## ユーザー情報の更新

```tsx: Account.tsx
const user = supabase.auth.user()

const updates = {
    id: user.id,
    username,
    website,
    avatar_url,
    updated_at: new Date(),
}
// profilesテーブルへのUPSERT(レコードがなければINSERTを行い、レコードがあればUPDATEを行う処理)を実行
let { error } = await supabase.from('profiles').upsert(updates, {
  returning: 'minimal', // minimal | representation -> デフォルトでは、新しいレコードが返されます。この値が不要な場合'minimal'に設定
})
```

## プロフィール画像

プロフィール画像を Supabase のストレージにアップロード・ダウンロードするコンポーネントです。
`Account.tsx`の子コンポーネントとして設置します。

```tsx: Avatar.tsx
// ダウンロード
const { data, error } = await supabase.storage
    .from('avatars')
    .download(path) // ファイルとパス名を指定します。

// アップロード
let { error: uploadError } = await supabase.storage
    .from('avatars')
    .upload(filePath, file) // filePath: アップロードするファイルのパス, file: ファイルの本体
```

実際に Supabase のストレージに画像がアップロードされているのが確認できます。
![ストレージ](https://storage.googleapis.com/zenn-user-upload/8285ee75bd33-20211218.png)

## ログインページの作成

これまで作成した、2 つのコンポーネント(Auth.tsx と Account.tsx)をページに設置します。
セッション情報の有無で表示するコンポーネントを切り替えます。

```tsx: pages/idndex.tsx
import type { NextPage } from 'next'
import { useState, useEffect } from 'react'
import { supabase } from '../src/utils/supabaseClient'
import { Auth } from '../src/components/Auth'
import { Account } from '../src/components/Account'

const Home: NextPage = () => {
  const [session, setSession] = useState(null)

  useEffect(() => {
    // アクティブなセッションがある場合そのセッションデータが返ってくる
    setSession(supabase.auth.session())

    // 認証イベントが発生するたびに通知を受け取ります。
    supabase.auth.onAuthStateChange((_event, session) => {
      // _event: SIGNED_IN, SIGNED_OUT
      // sessin: セッション情報
      setSession(session)
    })
  }, [])

  return (
    <div className="container" style={{ padding: '50px 0 100px 0' }}>
      {!session ? (
        <Auth />
      ) : (
        <Account key={session.user.id} session={session} />
      )}
    </div>
  )
}

export default Home

```

[^1]:
    メールの内容を変更する際 CORS error になっていました。
    この事象は GitHub の Issue や Discussions をみても解決できなかったので Supabase のサポートに確認中です。進捗があり次第この記事に加筆します。

# おわりに

Supabase と Next.js を使って認証機能を作成しました。
個人的には、DB が PostgreSQL なので、Firebase よりも親しみやすかったです。
個人開発でもっと使ってみたいと思いアイデアを考える日々です。

今回使用したコードは下記リポジトリにあります。
https://github.com/TakaShinoda/supabase-sample
