---
layout: cover
background: https://source.unsplash.com/collection/94734566/1920x1080
---

# メンヘラ判定AI LINE botを<br>作った時の話
<div align="right">
<img src="/img/mental_health_woman.png" alt="属性" title="メンヘラ">
</div>
<!-- <div style="image-align: right;">
![](/img/mental_health_woman.png) 
</div>-->
---
theme: seriph # https://sli.dev/themes/gallery.html
title: メンヘラ判定AI LINE botを作った時の話
download: true
lineNumbers: true
background: https://source.unsplash.com/collection/94734566/1920x1080
class: 'text-center'
---

# メンヘラとは何か？
自分も改めて考えると定義がわからなかったのでググってみた

ネット上から生まれた造語でメンタルヘルスに関することを表す。
メンヘラとは一般的に「メンタルヘルス（心の健康）になんらかの問題を抱えている人」をあらわして使われることが多いよう。
[[1]](https://domani.shogakukan.co.jp/428928)

→**情緒が不安定な人(感情の起伏が激しい人)のことをさす認識**

<br>
<br>

<div align="center">
<img src="/img/mental_health_man.png" alt="属性" title="メンヘラ">
</div>

---

# なんで作ったの？
背景

- AIを使った面白いものを作りたかった
- メンヘラは写真の取り方にかなり特徴があるので、AIで精度よく判定できるんじゃね？と思いついた 
- せっかく作るなら誰かに自慢したかった(話題のネタになる)
  - LINEならだれでも使ってて、友達登録するだけですぐに使えるのでLINE botにメンヘラ判定させようと決定
- 修士論文の提出が終わり、やることがなかった
  - ~~周りに遊ぶところも遊ぶ人もいなかった~~




<div align="right">
<img src="/img/chuunen_neet_snep.png" alt="属性" title="メンヘラ">
</div>

---
class: 'text-reft'

---

# 作成物デモ
画像をLINEのbotに投げるとその人がメンヘラである確率が何%あるのか、また、AIがその判定をするに至って重視した画像の領域を強調表示した画像を提示(2018年頃に発表されたgrad_camという技術)


<div class="grid grid-cols-[50%,50%] gap-4"><div>

<center>
QRコード
</center>
<img src="/img/menhera.png" alt="属性" title="QR">

</div><div>

<iframe width="300" height="350" src="https://youtube.com/embed/U0dGHA2edIA" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

</div></div>

<center>
もしよかったら皆さんも実際に使ってみてください!メンヘラ判定AIで検索書けると出てきます！
</center>

---

# システム構成図
システム構成紹介

<div align="center">
<img src="/img/sistems.jpg" alt="属性" title="メンヘラ">
</div>

---
layout: cover
background: https://source.unsplash.com/collection/94734566/1920x1080
---

# 実装するにあたり苦労した事3選
需要があれば次回までに用意してきます！
---

# 1つ目：学習データの用意
メンヘラ判定用データセットなど当然世の中にないので、自身で作成する必要がある。

一枚一枚画像保存してlabel付けするのは~~めんどくさい~~ナンセンスなので、スクレイピングでの収集を考えた。
<div align="center">
↓
</div>
 インスタグラム、Google、Twitterは各会社が提供しているAPIの使用が必要で、インスタグラムに至っては自アカウントで投稿した写真しかAPIが提供されておらず使用不可(一番使いたかったのに...)
  GoogleとTwitterは一日100リクエストまでと制限があり、すぐに集められれないし申請が面倒
(GoogleはAPIを使わずともスクレイピングできるが規約違反にあたる)

<div class="grid grid-cols-[60%,50%] gap-4"><div>

### 早々にあきらめかけたところ...
運よく下記画像投稿サイトを発見できた。
[[コミュニティ プリ画像]](https://prcm.jp/new)

規約をみてもスクレイピングだめとは書いてなかったので使わせてもらうことに。

## <u>教訓①あきらめなければなんとかなる</u>

</div><div>

![](/img/yaruki_moeru_businessman.png)

</div></div>

---

## スクレイピングのコード

<style>
.language-bash span.line { /* bashのコード */
  margin-left: -40px; /* 左に40px移動して行番号を隠す(邪道) */
}
</style>
スクレイピングのコード自体はすごくシンプルに書ける

本コードではサイトの検索機能を使用して、検索したキーワードにヒットした画像の上位約1万枚を収集するコードになっている

本スクレイピングのソースコード紹介

```python {0-}
    pages = [i for i in range(1,1201)] #1200*9=10800枚収集

    for p in pages:
        if p == 1:
            url = "https://prcm.jp/list/" + keyword
        else:
            url = "https://prcm.jp/list/" + keyword +"?page=" + str(p)
        r = requests.get(url)
        soup = BeautifulSoup(r.text,'lxml')
        imgs = soup.find_all('img',src=re.compile("^https://pics.prcm.jp/"))
        for img in imgs:
            r = requests.get(img['src'])
            with open(str('./data/scraping/menhera') + str(uuid.uuid4()) + str('.jpeg'),'wb') as file:
                file.write(r.content)
```
スクレイピングのプログラム自体はシンプルに書けるが、各サイトのhtmlの表現方法に依存するので、ソースのコピペ運用はできない

---
class: 'text-reft'
---

# 2つ目：bot を稼働させるサーバがない
herokuという無料レンタルサーバがあるが、、、

### AIの学習済みモデルファイルがでかすぎて無料枠ではデプロイできない事が判明

<br>

# まだ作成中


---

# 3つ目：AIの精度がゴミ
人間じゃない写真なのに率が高くでたりする

まだ作成中

---
layout: cover
background: https://source.unsplash.com/collection/94734566/1920x1080
---

# ご清聴ありがとうございました。


