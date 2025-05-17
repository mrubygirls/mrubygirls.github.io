# mruby Girls

mruby Girls

## ディレクトリ構成

```
ROOT
  |- blog/                                         # 開催ブログ
  |   |- index.markdown
  |
  |- evetns/                                       # イベント情報
  |   |- index.markdown
  |   |- 2025-11-08-matsue_1st.markdown
  |
  |- glossary/                                     # 用語集
  |   |- index.html
  |   |- breadboard.markdown
  |
  |- guides/                                      # ガイド
  |   |- index.html
  |   |- led_blinking.markdown
  |   |- thermistor.markdown
  |
  |- assets/
  |   |- css/
  |       |- style.css
  |
  |- iamges/
  |
  |- _posts/
  |    |- YYYY-MM-DD-xxx.markdown   # ブログ
  |
  |- index.html
```

## 編集方法

gitリポジトリをcloneして、bundle installする。
```
$ git clone git@github.com:mrubygirls/mrubygirls.github.io.git
$ bundle install
```

Serverを起動して、ブラウザで http://localhost:4000/ を表示する。
```
$ bundle exec jekyll serve --watch
```
