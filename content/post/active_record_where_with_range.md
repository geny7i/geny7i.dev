---
title: "ActiveRecordの範囲検索で無限大/無限小を扱う"
date: 2022-12-09T19:06:26+09:00
tags: ["Rails"]
draft: false
---

ActiveRecordのwhereにはRangeが渡せるのは前々から知ってたんですが、Rangeの性質上始端を持たなければいけないよなって思ってたんですが、昨今のrubyは始端のないRange扱えるということに気がついたというか思い出したというか

要はこういうコードが書けます

<!--more-->

```ruby
# 始端無し
User.where(activated_on: ..Date.new(2022,12,31)).to_sql
=> "SELECT \"users\".* FROM \"users\" WHERE \"users\".\"activated_on\" <= '2022-12-31'"
# 終端無し
User.where(activated_on: Date.new(2022,1,1)..).to_sql
=> "SELECT \"users\".* FROM \"users\" WHERE \"users\".\"activated_on\" >= '2022-01-01'"
```

これが使えることによって、whereに生SQL差し込んだりArel使わなくても不等号のwhereをかけられるので便利です  

始端の値を範囲に含まないRangeは作れないので`>`(lager than)はできないんじゃないか?と思いきや`.not`で範囲を反転してくれるのでできます

```ruby
User.where.not(activated_on: ..Date.new(2022,12,31)).to_sql
=> "SELECT \"users\".* FROM \"users\" WHERE \"users\".\"activated_on\" > '2022-12-31'"
```

しかしこれは流石に可読性がよろしくないですよね  
単純なscopeに使うだけならギリ許せますがドメインロジックに急に現れたら厳しすぎる  
どうしても必要ならArelやら文字列やら使ったほうがマシでしょうね  

```ruby
User.where(User.arel_table[:activated_on].gt(Date.new(2022,12,31))).to_sql
=> "SELECT \"users\".* FROM \"users\" WHERE \"users\".\"activated_on\" > '2022-12-31'"
User.where("activated_on > ?", Date.new(2022,12,31)).to_sql
=> "SELECT \"users\".* FROM \"users\" WHERE (activated_on > '2022-12-31')"

```

まぁ大抵Range使いたくなるのは日付検索とかで、始端/終端の値含むことが多いと思うので困ることは少なそうですが



