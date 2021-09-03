---
layout: post
title: "ブログを Ghost から Jekyll に移行した"
image: ""
date: "2021-08-29 22:33:27"
tags:
---

2016 年 3 月 15 日 (火) からスタートしたブログを移行した。以前使っていたのは [Ghost](https://ghost.org/) というブログシステムで、今回、静的サイトジェネレータである [Jekyll](https://jekyllrb.com/) に移行した。



# 移行のきっかけ
5 年前と比べて、考え方がかなり変わったことが要因として挙げられる。[Ghost ブログを開始したときの一番初めの記事](/start-blog) を見てもらえればわかると思うが、当時はサーバの構築からデプロイまで、すべて自分でやることで「自分だけのオリジナルのブログ」という存在感を味わいたかった。

しかし、今は違う。自分でサーバを借りてブログシステムをホスティングするということは、すべてが自分の管理下にある。それは自分の思い通りのブログを作ることができる反面、リスクでもあると考えている。

たとえば、自分がこの世からいなくなると、そのブログを管理できる人間がいなくなる。自分しか管理していないのだから当然である。そうなると、自分のブログの存在期限は、自分が残りの人生で活動できるそのときまで、ということになる。実際、Ghost ブログを立ち上げてから今までの間、しばらくブログを放置していた期間があったが、SSL 証明書の期限切れでしばらくアクセスできなくなっていたこともしばしばあった。

しかし、個人的には、このブログだけでなく、自分がそれまでに紡いできたすべてのコンテンツが、後世に残り続けてほしいと願っている。変に聞こえるかもしれないが、自分が生きている間だけではなく、長き時を経ても残り続けるものとして残しておきたいのだ。

この思想と、自分ですべてを管理することは矛盾する。だから、自分で管理せずとも、ずっと残り続けるような形でブログを残そうと思ったのだ。

どうやったらそれが実現できるか。正直なところ、100 % 永遠に残し続けることを保証するのは不可能だ。なぜなら、自分の死後、世界がどうなっているかは知ることができないし、完全にコントロール不能だからだ。その中でも、確率的に残り続ける可能性が高い方法を考えた結果、[GitHub Pages](https://pages.github.com/) にホスティングするのが良いのではないかという結論に至った。

最近では [Note](https://note.com/) や [Medium](https://medium.com/) など、次世代ともいえるブログサービスが台頭してきているが、ブログを後世に残し続けるという観点においては、以下のデメリットが挙げられる。

* ブログ記事の物理的な所有権を持っていないので、運営の裁量で消されたり、運営のデータ管理ミスで消えたりしてしまう可能性がある
* 時代とともに移り変わるので、サービスが終了する可能性があり、そのときにすべてが失われてしまう

もちろん、上記の 2 点は、GitHub Pages では存在しない、というわけではない。GitHub Pages だって、サーバは GitHub, Inc. が管理しているわけだから、ホスティングされているブログサイトの実質的な所有権を自分が持っているわけではないし、データベースの管理ミスで消えてしまう可能性がないわけではない。それに GitHub Pages が消えてしまう可能性も否定はできない。

しかしそれはどこにおいても同じなのだ。絶対的に安全な場所など存在しない。その中で GitHub Pages を選んだ理由は、以下の通りだ。

* [GitHub Archive Program](https://archiveprogram.github.com/) によって今後 1,000 年間に渡りソースコードが維持されることを宣言している
* `git pull` することですべてのコピーが瞬時に手元にダウンロードできる
* 大企業である Microsoft が管理しているので、そうそうサービスが終了することはないと予想できる

上記は GitHub Pages のホスティングの永続性というよりも、主にデータの永続性に関してだが、基本的には手元で記事を書いて公開することになるので、ローカルとサーバ (GitHub) 両方にデータが存在することになる。しかも、PC を新しくするなど、ローカル環境を一新したとしても、CLI で `git pull` コマンドを一発叩くだけですぐに手元にコピーを復元することができる。

さらに、GitHub 自体、IT 界隈ではかなり巨大なインフラともいえるので、GitHub Pages もそう簡単に終了するとも思えない。

以上より、GitHub にブログデータをホスティングし、GitHub Pages で公開することが、現状で最も安全に長くデータを残しインターネット上に公開し続けることができる方法なのではないかと結論づけた。

これは最近たどり着いた答えではなく、1 〜 2 年ほど前からそうしようと思っていたのだが、なかなか重い腰が上がらず、実行に移せなかった。それをついに実行に移すときが来た。



# Ghost からのデータのインポート
さて、ブログを移行するためにはまずデータのインポートを行う必要があるのだが、これに関しては、想像していた以上に簡単だった。

Ghost ブログから Jekyll に移行するツールを 2 つ見つけた。一つは Jekyll 公式のインポーターで、もう一つは、その公式が別の方法として紹介しているインポーターだ。

* [jekyll-import](https://import.jekyllrb.com/docs/ghost/)
* [jekyll_ghost_importer](https://github.com/eloyesp/jekyll_ghost_importer)

どちらも使い方は簡単だ。Ghost サーバからデータベースファイルないしエクスポート用 JSON ファイルをダウンロードしてきて、gem をインストールして、数行もしくは一行のコマンドを実行するだけだ。

両方試してみたのだが、公式の [jekyll-import](https://import.jekyllrb.com/docs/ghost/) は以下の問題が見つかった。

* ブログの公開日が正しくインポートできていない
* 記事のサムネイル画像のパスがインポートできない
* 記事のタグがインポートできない

それぞれのインポーターの生成した記事ファイルの diff を取ってみたら、記事の内容自体は全く一緒だったので、単純に情報が正しく、かつ多く取得できている [jekyll_ghost_importer](https://github.com/eloyesp/jekyll_ghost_importer) のインポート結果を採用することにした。



# 独自ドメインは使用しない
Ghost ブログは、はじめ `blog.noraworld.jp` というドメインで利用開始し、後に `noraworld.blog` に変更した。

このドメイン自体はとても気に入っているのだが、今後は独自ドメインの利用は極力控えていこうと思っている。

その理由に関しては、先ほども書いたとおり、独自ドメインにしてしまうと「管理」が必要になってしまうからだ。

仮にブログ記事とホスティングが残り続けたとしても、ドメインの更新期限が切れてそのままになってしまえば、そのサイトにアクセスすることはできなくなる。それは結局のところ、何から何まで自分で管理し、管理ができなくなった瞬間に破綻するのと同じだ。

だから、自分が管理し続けないとそこにアクセスできなくなるもの、いわゆる「管理コスト」が発生するものに関しては徹底的に排除することにした。

ドメインは GitHub Pages が提供するデフォルトのものを利用させてもらうことにする。これなら自分で管理することがないので、少なくとも自分が管理できなくなることによってこのサイトが見られなくなることはないだろう。



# Jekyll を採用した理由
サイト自体を自分の管理不要で残せるようにするのは良いとして、あとはどのようにブログを構築するかということだ。外部のツールや技術に依存すればするほど、それらのサポートが終了した際に、サイトが意図したとおりに表示させられなくなる可能性がある。

例として CDN を利用して CSS フレームワークや JS ライブラリを読み込んでいる場合は、その CDN がサービス終了したり、使用しているフレームワークやライブラリのサポートが終了したりした瞬間に、サイトが意図通りに表示されなくなる。

なので、これらのものにもなるべく依存したくない。

正直なところ、静的サイトジェネレータ自体も使わずに済むなら理想的なのだが、それはさすがに現実的ではない。新しく記事を書くたびに毎回 HTML を書くのはゴメンだ。

となると、どの静的サイトジェネレータを選ぶのが最適かだが、Jekyll を選んだ理由は、何を隠そう散々話題にした GitHub がサポートしているジェネレータだからだ。

それに Jekyll は Ruby 製で、自分が一番親しみのある言語なので親和性も高い。そのためあまり迷うことはなかった。

さらに、Jekyll に関しては GitHub Pages によるビルドがサポートされているので、予めローカルで HTML をビルドしてから push しなくても、元のソース (マークダウン等) を公開すれば、あとは GitHub のサーバが自動的にビルドしてページを公開してくれる。

これこそもしかしたらサポートが終了してしまう可能性はあるのだが、まあそうなったらそうなったで、他のジェネレータと同じ要領で、自分で HTML をビルドしてから push するように切り替えれば問題なかろう。

## プラグインを多用しない
Jekyll は豊富なプラグインが存在するが、あまり多用すると、何度も言っているとおりサポートの終了等でいつか使えなくなる可能性が出てくるので、なるべく使わないようにする。

そもそも、GitHub 側で自動ビルドをしてもらう場合は、限られたプラグインしか使えないようになっている。これはおそらくセキュリティ上の問題だろう。何でもかんでもプラグインを使ってビルドできるようになってしまったら、誰かが悪意のあるプラグインを作って GitHub サーバ上で実行させて悪さをすることができてしまうので。

ちなみに GitHub の自動ビルドでサポートされている Jekyll プラグイン一覧は以下のページで確認できる。

[Dependency versions](https://pages.github.com/versions/)

基本的にはこの中にあるもので運用していこうと考えているが、どうしても無理な場合は、ローカルで自分でビルドしてビルド済みのファイルを GitHub Pages で公開するようにすることも考える。



# 趣味開発のモチベーションと今後について
ここまで話してきた理由により、最近は技術記事や個人用のプライベートメモなんかも GitHub リポジトリ上で保管するようにしている。

今後、自分の作ったコンテンツを、自分で借りているサーバ等で管理しない方針で行くとなると、趣味のサービス開発のモチベーションは皆無となる。

当然のことながら、今後、Web サービス等を個人で開発したとしても、自分が管理できなくなったとき、そのサービスのデータベース上に保管されているデータも失われてしまう可能性がある。そう考えると、データを自分自身で持つ必要があるサービスの趣味開発はする気が起こらなくなる。

2016 年 8 月 10 日 (水) に公開した日記も、まさに自分オリジナルの日記サイトが欲しいという強いモチベーションから開発を始め、それ以来、定期的に改良を重ね運用し続けてきたのだが、この日記も将来的には GitHub Pages に移行するべきなのだろうと考えると、日記サービスの開発に対してのモチベーションもすっかりなくなってしまった。

このシステム、個人的にはとても気に入っていて我ながら最高の日記サービスだと思っているのだが、それ以上に今までに綴ってきた日々の日記のデータのほうが大切なので、その大切なデータを将来的に長く残すことを考えたら、借りているサーバで管理するよりも、GitHub に残しておくほうが安心だろう。

そのため、今後は Web サービス等の、自分でデータを管理する必要のある媒体の開発ではなく、スマートフォンアプリやブラウザ拡張機能、ライブラリ、エンジニアツールなどの、自分でデータを管理する必要のない媒体の趣味開発に舵を切っていこうと思っている。

さっそく、直近では、以下のような趣味開発をしてみようかなという機運が高まっている。

* プランニングポーカー用の静的サイトの作成
    * 仕事でプランニングポーカーをする機会があったので
* macOS 用ポモドーロアプリの作成
    * 現在は Be Focused というアプリを使っているが、ポモドーロセッション中に Do Not Disturb を自動的にオンにするアプリがなかなか見つからないので、自分で作ってみようかなと思って
    * 数年前にポモドーロテクニックを導入してからやめてしまい、しばらくやっていなかったが、最近また導入し始めたらとっても心地よかった
* [vscode-ruby-rubocop](https://github.com/misogi/vscode-ruby-rubocop) のバグ修正 PR
    * https://github.com/misogi/vscode-ruby-rubocop/issues/155 の修正
* YouTube Studio の動画管理ページから埋め込み用の iframe タグと Twitter 共有用の動画タイトル、動画リンクをコピーできる Chrome 拡張機能の開発
    * [これ](https://zenn.dev/noraworld/articles/access-restriction-using-dnsmasq) によって夜まで YouTube にアクセス制限をかけているせいで iframe タグや共有機能が使えないため

macOS や iPhone のアプリなら、ドックフーディング的な感じで、自分で使いたいものを作り、使うことができる上に、自分でそのアプリのホスティング等を管理する必要がないので、自分が管理できなくなったことによって、そのアプリが使えなくなるといったことは発生しないだろう。もちろん OS のバージョンが上がることによる互換性の問題等で動かなくなる可能性はあるだろうが、少なくともデータベースに存在する大切なデータが失われる、というようなことは発生しない。



# さいごに
めっきりブログを更新することもなくなってしまったが、つい最近になってからは、自分用のメモ的な感じのものをブログで残そうということを始めている。自分専用のメモだったとしても、別に公開して困るようなものじゃないなら、誰でも見られる状態でインターネット上に垂れ流してしまっても良いだろうと、最近は思っている。

このへんも昔とは考え方が変わっている。昔は、外部に公開するものはちゃんとした状態じゃないと自分の気がすまなかったところがあった。今でもこの完璧主義なところは抜けてはいないのだが、以前よりは軽い気持ちで適当なものでも外部に公開できるメンタリティになってきた。

今後は、ブログかつ個人用メモ的な感じでこのブログを活用していきたいと思う。このブログの内容が、後世に渡って残り続け、たった一人の誰かにでも役に立つことを願って。