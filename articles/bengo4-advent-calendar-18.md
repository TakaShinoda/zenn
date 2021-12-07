---
title: 'supabase'
emoji: '🦌'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: ['supabase', 'アドベントカレンダー']
published: false
---

:::message
この記事は [弁護士ドットコム Advent Calendar 2021](https://qiita.com/advent-calendar/2021/bengo4com) の 18 日目の記事です。
:::

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

- プロジェクト名とパスワードを入力し、リージョンを指定 (Tokyo も存在します)
  ![画像4](https://storage.googleapis.com/zenn-user-upload/143ee560077f-20211204.png)

## Next.js で新規プロジェクト作成

`create-next-app`を使って、プロジェクトを新規作成します。

```
npx create-next-app@latest --typescript
```

# チュートリアルをやってみる

<!-- Set up the database schema# -->
- Set up the database schema#

<!-- Get the API Keys# -->
- Get the API Keys#

- supabase-js のインストール

```bash
npm install @supabase/supabase-js
```

```text: .env.local
NEXT_PUBLIC_SUPABASE_URL=https://xxxxxxxxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=xxxxxxxxxxxxxxxxxxxx
```

```ts: supabaseClient.ts
import { createClient } from '@supabase/supabase-js'

const supabaseUrl= process.env.NEXT_PUBLIC_SUPABASE_URL as string
const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY as string

export const supabase = createClient(supabaseUrl, supabaseAnonKey)
```

## テーブル作成

## サインイン

ユーザーは E メールまたは OAuth のいずれかでサインインできます。
パスワードなしでメールを提供した場合、ユーザーにはマジックリンクが送信されます。(今回はこれ)

```tsx: Auth.tsx
const { error } = await supabase.auth.signIn({ email })
```

実際に下記のようなメールが送られてきました。
![マジックリンクのメール](https://storage.googleapis.com/zenn-user-upload/9e1dd99cdf7e-20211206.png)

サードパーティのプロバイダを利用したサインインは主に下記をサポートしています。(一部抜粋)

- GitHub
- Google
- GitLab
- Bitbucket

## サインインしたユーザの情報表示

```tsx: Account.tsx
const user = supabase.auth.user()

let { data, error, status } = await supabase
    .from<definitions['profiles']>('profiles')
    .select(`username, website, avatar_url`)
    .eq('id', user.id)
    .single()
```
