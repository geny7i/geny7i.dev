---
title: "SourceTree依存者向けGitコマンドチートシート"
date: 2022-12-08T19:44:39+09:00
tags: ["Tips"]
draft: false
---

### これは何?

WSL2環境で開発する時にSourceTree使おうとすると色々面倒そうなので脱gitGUIしますってことで、  
SourceTreeでよくやってた操作をCUIのコマンドに翻訳していくっていうページ  
GUI一回も使ったことない人が見ても大丈夫な程度には基本操作から書いていくけど、ワークツリーって何?とかいった概念的な説明はしない  
割と書いてる途中

<!--more-->

### ど基本

#### git init

空のGitリポジトリを作成

```
git init
```

GUI使っていてもこれはCUIでやってる人多そうだが一応

#### git add

全ファイルをステージング

```
git add .
```

マークダウンファイルのみステージング

```
git add *.md
```

#### git commit

コミットする。コミットメッセージ編集用のエディタが開く

```
git commit
```

エディタはconfigで指定できる

```
git config --global core.editor "vim" # vimにする
git config --global core.editor "code --wait" # vscodeにする
```

`--allow-empty`をつけるとステージングされたファイルがなくてもコミットできる  
`-m`コメントを指定するとでエディタを開かずワンラインでコミットできる

```
git commit --allow-empty -m 'commit message' 
```

### git status

`-s`でステージングの仕方/外し方の説明とブランチ名が出なくなり、`-b`でブランチ名が出るようになる

```
git status -sb
```

基本これでいい気がするaliasにした方がいいかも

### git diff

ワークツリーとステージの差分を出す
```
git diff
```

### git log

オプションなしだと何が何やらわからんのでツリー表示できるエイリアスを設定した方が良さそう
[参考](https://qiita.com/take4s5i/items/15d8648405f4e7ea3039)

```
# そこそこ綺麗
git log --oneline --graph --decorate
# いい感じに情報を出しつつ見やすい
log --graph --date=short --pretty="format:%C(yellow)%h %C(cyan)%ad %C(green)%an%Creset%x09%s %C(red)%d%Creset"
```

#### git fetch

```
git fetch
```

pruneオプションをつけるとリモートで消されたブランチを消してくれる

```
git fetch --prune
```

### 余談

SourceTreeにはコマンド履歴ってのが存在していてmacOS版だと表示タブから開ける(あるいはcontrol + command + w)  
ありがたいんだけどこれがなかなか見づらくて毎度毎度長いオプションがついてる