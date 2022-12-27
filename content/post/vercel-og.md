---
title: "Vercel Og"
date: 2022-12-19T20:09:57+09:00
tags: ["Next.js"]
draft: true
---

簡単にOGP画像出力を実装できる[@vercel/og](https://github.com/vercel/og-image)というのがVercelにあって、今回それを使ってみたのでメモ  
パスパラメーターを元に、WebAPIから情報を持ってきた画像と文言をOGP画像に反映するのをゴールにやっていく

環境
- aaa
- aaa

<!--more-->

何はともあれパッケージをインストール  

```
npm i @vercel/og
```

今回Next.jsでやってるので`pages/api`配下がAPI Routes  
パスはどうでもいいと思うが、Reactの糖衣構文を使うんで拡張子は`.jsx`なり`.tsx`にする必要がある  
今回は記事毎のogp画像だったので`pages/api/og/articles/[id].tsx`というファイルにした  

```tsx:pages/api/og/articles/[id].tsx
import { ImageResponse } from '@vercel/og';

export const config = {
  runtime: 'edge',
};

export default function () {
  return new ImageResponse(
    (
      <div
        style={{
          fontSize: 128,
          background: 'white',
          width: '100%',
          height: '100%',
          display: 'flex',
          textAlign: 'center',
          alignItems: 'center',
          justifyContent: 'center',
        }}
      >
        Hello world!
      </div>
    ),
    {
      width: 1200,
      height: 600,
    },
  );
}
```

これで開発用サーバーを立ち上げて`http://localhost:3000/api/og/articles/test`にアクセスすると

画像

とりあえず動く

上に書いたコードは[公式ドキュメント](https://vercel.com/docs/concepts/functions/edge-functions/og-image-generation)からのコピペだが割とこの時点でよくわかんなかった   
API Routesってこんな書き方だっけという  

調べてみるとNext.jsには[Edge APIルート](https://nextjs.org/docs/api-routes/edge-api-routes)なるものがあり、そいつはCDNエッジで動くのでNode.jsの関数は使えなくて所謂[`standard Web APIs`しか使えないよ](https://nextjs.org/docs/api-reference/edge-runtime)って話らしい  
~~いわゆらねぇよ`standard Web APIs`という言葉自体初耳なんだが...~~  
まぁ普通にブラウザで動くAPI≒ECMAScriptのことっぽい(多分)  
node_modulesにあるパッケージもES modulesに限りimportできるとのこと  

とにかくfetchは使えるっぽいので今回はなんとかなりそう

```tsx:pages/api/og/articles/[id].tsx
import { ImageResponse } from "@vercel/og";
import type { NextRequest } from "next/server";

export const config = {
  runtime: "edge",
};

export default async (req: NextRequest) => {
    const { searchParams } = req.nextUrl
    const articleId = searchParams.get('id') as string
    const res = await fetch(`url-to-article-api/${articleId}`)
    if(res.ok) {
      const article = await res.json();
      return new ImageResponse(
        (
          <img src={article.coverPhotoUrl}
            style={{
                width: "100%",
                height: "100%",
                objectFit: "cover"
            }}
           />
        ),
        {
          width: 1200,
          height: 630,
        }
      )
    } else {
      return new Response("Not Found", {
        status: 404,
      });
    }
  }
```

画像

できた    

実際Vercelにデプロイしてみたら結構早くて良さそう  
動的OGP画像作ろうっていうとHTMLを返すページを用意してpuppeteerでスクショするみたいな感じで、環境作るのとか実装するのが面倒だったが  
これは実質HTML書いてVercelに置くだけなんで簡単    
非商用サイトのOGP生成するためだけに使うのもアリだと思った  

公式ドキュメントに[Satori](https://github.com/vercel/satori)とかいうのが出てくるので、これを使ってるんだとばかり思ってたが、puppeteerに依存してるところを見るに結局HTML作ってpuppeteerでスクショしてるっぽい    
SatoriはHTMLをsvgに変換できるパッケージらしくこれで作ったsvgをpngにするみたいなアプローチも使われてたりしたみたいですが、現状は`@vercel/og`推奨みたい

終わり

