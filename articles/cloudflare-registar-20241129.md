---
title: "Cloudflare Registarでドメイン取得"
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Cloudflare"]
published: false
---
現在、ポートフォリオサイトの作成を行っているので、外部公開するにあたって独自ドメインで公開したいと考えていました。
どのレジストラでドメインを購入しようかと考えていましたが、Cloudflareでもドメイン管理サービス「Cloudflare Registar」を
提供していたので、Cloudflare Registarでドメインを取得することにしました。

# Cloudflare Registarとは
Cloudflare Registarとは、Cloudflare内で提供されているドメイン管理サービスのことを言います。
Cloudflare Registarでは、ドメインを**原価**で購入・更新することができます。
他のレジストラは、初回購入を格安でできるようにし、更新料が高くなることがありますが、Cloudflare Registarは更新も原価で提供しており、
公式ドキュメントでも、「追加料金または更新料の値上げはありません。」と記載があります。


https://www.cloudflare.com/ja-jp/products/registrar/

# 取得手順
## サインアップ
自身のCloudflareアカウントにサインアップしてください。

## ドメイン登録
左ペインから「ドメイン登録」->「ドメインの登録」をクリックしてください。
![](/images/cloudflare-registar-20241129/cloudflare-registar01.png)


ドメイン名を検索する欄があるので、購入するドメイン名（今回はcham-pion.org）を入力して検索します。

Cloudflare Registarでの、cham-pion.orgのドメイン費用は以下の通りでした。
- 初回購入費用：7.50$
- 更新料：10.11$
問題なければ、「購入」をクリックします。

![](/images/cloudflare-registar-20241129/cloudflare-registar02.png)

## 登録者情報の入力
登録を完了するためには、「登録者情報」と「お支払い」情報を入力する必要があるため入力します。
登録者情報配下の項目を入力する必要があります。

| 項目 | 必須 | 
| ---- | ---- | 
| 名 | Yes | 
| 苗字 | Yes | 
| 組織 | Yes | 
| メールアドレス | No | 
| 電話番号 | Text | 
| 内線 | No | 
| 番号 | Yes | 
| 国 | Yes | 
| 住所１ | Yes | 
| 住所２ | No | 
| 市町村 | Yes | 
| 都道府県 | Yes | 
| 郵便番号 | Yes | 

「登録者情報」と「お支払い」情報を入力し、問題がなければ「購入完了」をクリックします。

が、、、、
謎のエラー「申し訳ありませんが、問題が発生しました。」が出てしまい購入できない事態になりました。。
入力画面でどこがエラーになっているかは表示されないので、エラーの解消に苦労しました。
（結局、決済で使用しているクレジットカードを変更したら通りました。）

![](/images/cloudflare-registar-20241129/cloudflare-registar03.png)

これでドメイン登録は完了です。

![](/images/cloudflare-registar-20241129/cloudflare-registar04.png)

![](/images/cloudflare-registar-20241129/cloudflare-registar05.png)

## DNSSECの有効化/WHOIS privacy
Cloudflare Registrarでドメインを取得すると**DNSSECをワンクリックかつ無料で利用**することができます。
ワンクリックかつ無料なので、即座に有効化することをお勧めします。
また、**WHOIS privacy**の機能によりWHOIS上の個人情報にあたるほとんどの部分は「REDACTED FOR PRIVACY」と表示されるようになります。
国と、県は表示されてしまうようです。

![](/images/cloudflare-registar-20241129/cloudflare-registar06.png)


```bash
$ whois cham-pion.org
Domain Name: cham-pion.org
Registry Domain ID: 93bf1e20ec5f45548f4e16c910f991bb-LROR
Registrar WHOIS Server: http://whois.cloudflare.com
Registrar URL: http://www.cloudflare.com
Updated Date: 2024-11-28T21:42:03Z
Creation Date: 2024-11-28T21:41:59Z
Registry Expiry Date: 2025-11-28T21:41:59Z
Registrar: CloudFlare, Inc.
Registrar IANA ID: 1910
Registrar Abuse Contact Email: registrar-abuse@cloudflare.com
Registrar Abuse Contact Phone: +1.6503198930
Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited
Domain Status: addPeriod https://icann.org/epp#addPeriod
Registry Registrant ID: REDACTED FOR PRIVACY
Registrant Name: REDACTED FOR PRIVACY
Registrant Organization: 
Registrant Street: REDACTED FOR PRIVACY
Registrant City: REDACTED FOR PRIVACY
Registrant State/Province: Saitama
Registrant Postal Code: REDACTED FOR PRIVACY
Registrant Country: JP
Registrant Phone: REDACTED FOR PRIVACY
Registrant Phone Ext: REDACTED FOR PRIVACY
Registrant Fax: REDACTED FOR PRIVACY
Registrant Fax Ext: REDACTED FOR PRIVACY
Registrant Email: Please query the RDDS service of the Registrar of Record identified in this output for information on how to contact the Registrant, Admin, or Tech contact of the queried domain name.
Registry Admin ID: REDACTED FOR PRIVACY
Admin Name: REDACTED FOR PRIVACY
Admin Organization: REDACTED FOR PRIVACY
Admin Street: REDACTED FOR PRIVACY
Admin City: REDACTED FOR PRIVACY
Admin State/Province: REDACTED FOR PRIVACY
Admin Postal Code: REDACTED FOR PRIVACY
Admin Country: REDACTED FOR PRIVACY
Admin Phone: REDACTED FOR PRIVACY
Admin Phone Ext: REDACTED FOR PRIVACY
Admin Fax: REDACTED FOR PRIVACY
Admin Fax Ext: REDACTED FOR PRIVACY
Admin Email: Please query the RDDS service of the Registrar of Record identified in this output for information on how to contact the Registrant, Admin, or Tech contact of the queried domain name.
Registry Tech ID: REDACTED FOR PRIVACY
Tech Name: REDACTED FOR PRIVACY
Tech Organization: REDACTED FOR PRIVACY
Tech Street: REDACTED FOR PRIVACY
Tech City: REDACTED FOR PRIVACY
Tech State/Province: REDACTED FOR PRIVACY
Tech Postal Code: REDACTED FOR PRIVACY
Tech Country: REDACTED FOR PRIVACY
Tech Phone: REDACTED FOR PRIVACY
Tech Phone Ext: REDACTED FOR PRIVACY
Tech Fax: REDACTED FOR PRIVACY
Tech Fax Ext: REDACTED FOR PRIVACY
Tech Email: Please query the RDDS service of the Registrar of Record identified in this output for information on how to contact the Registrant, Admin, or Tech contact of the queried domain name.
Name Server: algin.ns.cloudflare.com
Name Server: sandra.ns.cloudflare.com
DNSSEC: unsigned
URL of the ICANN Whois Inaccuracy Complaint Form: https://www.icann.org/wicf/
>>> Last update of WHOIS database: 2024-11-28T21:57:41Z <<<
```


# まとめ
Cloudflare Registrarでドメインの取得を実施しました。
決済周りでてこずってしまいましたが、無事取得できてよかったです。
レコード周りについてはまた別の記事でアップしたいと思います。