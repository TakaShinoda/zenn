---
title: 'supabase'
emoji: '🦌'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: ['supabase']
published: false
---

:::message
この記事は [弁護士ドットコム Advent Calendar 2021](https://qiita.com/advent-calendar/2021/bengo4com) の 18 日目の記事です。
:::

# はじめに

こんにちは。
弁護士ドットコム株式会社 クラウドサイン事業本部 プロダクト部の篠田([@tttttt_621_s](https://twitter.com/tttttt_621_s))です。

<!-- 前日は、[@komtaki](https://qiita.com/komtaki)さんの XXXXX でした！ -->

前日は、XXXXX さんの XXXXX でした！

私は、個人開発の際に Firebase を使うことがあるのですが、最近 Supabase というものを知ってこの機会にさわってみました。
また、並行して Zenn への投稿も初めてやってみました。

よろしくお願いします。

# Supabase とは

Firebase Alternative と公式サイトで謳ってあるオープンソースの Baas です。
具体的には下記の機能があります。

- Database(PostgreSQL)
- Authentication
- Storage
- Functions(Comming soon) <!-- <- 2021/12/03 のステータス -->

https://supabase.com/

また、11/29 から 5 日間 [Supabase Launch Week III: Holiday Special](https://supabase.com/blog/2021/11/26/supabase-launch-week-the-trilogy) が行われていました。

- Monday: Community Day
- Tuesday: Dashboard Surprise
- Wednesday: Realtime Updates
- Thursday: A new player enters the game
- Friday: (One more thing)

Supabase コミュニティを構成するコミュニティと彼らのアップデートの紹介、新機能の概要、買収などありました。

# 環境構築

## Supabase に登録・プロジェクト作成

- 公式サイトから「Start your project」を選択

![画像1](https://storage.googleapis.com/zenn-user-upload/9e14030a059a-20211204.png)

- GitHub でログイン後、「New Project」を選択

![画像2](https://storage.googleapis.com/zenn-user-upload/87faa5d13928-20211204.png)

![画像3](https://storage.googleapis.com/zenn-user-upload/e6dd877255f2-20211204.png)

- プロジェクト名とパスワードを入力し、リージョンを指定します。 (Tokyo も存在します。)

![画像4](https://storage.googleapis.com/zenn-user-upload/143ee560077f-20211204.png)

<!-- # チュートリアルをやってみる -->

## データベーススキーマの設定

- 今回は「User Management Starter」のクイックスタートを使用します。

![User Management Starter](https://storage.googleapis.com/zenn-user-upload/75dc4c0a38ed-20211211.png)

- 「RUN」を押下してクエリを実行します。

![クエリ実行](https://storage.googleapis.com/zenn-user-upload/9ddd5a7db4d8-20211211.png)

- テーブルが作成されます。

![テーブル](https://storage.googleapis.com/zenn-user-upload/20bbeb38bd85-20211211.png)

<!-- Get the API Keys# -->

## API キーの取得

- 「設定」->「API」から Config URL と API keys を確認します。

![API setting](https://storage.googleapis.com/zenn-user-upload/2808f68f4684-20211212.png)

## Next.js で新規プロジェクト作成

- `create-next-app`を使って、プロジェクトを新規作成します。

```
npx create-next-app@latest --typescript
```

- supabase-js をインストールします

```bash
npm install @supabase/supabase-js
```

https://github.com/supabase/supabase-js

- Next.js のプロジェクト配下に`.env.local`ファイルを作成し、先程の Config URL と API keys を記述します。

```text: .env.local
# Config URL
NEXT_PUBLIC_SUPABASE_URL=https://XXXXXXXXXXXXXXXXXXXX.supabase.co
# API keys
NEXT_PUBLIC_SUPABASE_ANON_KEY=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

```ts: supabaseClient.ts
import { createClient } from '@supabase/supabase-js'

const supabaseUrl= process.env.NEXT_PUBLIC_SUPABASE_URL as string
const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY as string

export const supabase = createClient(supabaseUrl, supabaseAnonKey)
```

# 機能作成

## サインイン

ユーザーは E メールまたは OAuth のいずれかでサインインできます。
パスワードなしでメールを提供した場合、ユーザーにはマジックリンクが送信されます。(今回はこれ)
デフォルトでは、60 秒に 1 回マジックリンクを要求できます。

```tsx: Auth.tsx
const { error } = await supabase.auth.signIn({ email })
```

:::details email に useState を使ってフォームに入力されたアドレスを入れています。

```tsx: Auth.tsx
// ここに詳細を表示する
```

:::

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

<!-- メールの内容も変更でき、実際に変更してみました。 -->
<!-- ここうまくいかない -->




サードパーティのプロバイダを利用したサインインは主に下記をサポートしています。(一部抜粋)

- GitHub
- Google
- GitLab
- Bitbucket

https://supabase.com/docs/guides/auth/intro

## サインインしたユーザの情報表示

```tsx: Account.tsx
const user = supabase.auth.user()

let { data, error, status } = await supabase
    .from<definitions['profiles']>('profiles')
    .select(`username, website, avatar_url`)
    .eq('id', user.id)
    .single()
```

# おわりに

明日は、XXXXX さんで XXXXXX です！
