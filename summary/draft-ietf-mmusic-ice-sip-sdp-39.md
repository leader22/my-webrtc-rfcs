> Read [original](https://tools.ietf.org/html/draft-ietf-mmusic-ice-sip-sdp-39) / [markdown](../markdown/draft-ietf-mmusic-ice-sip-sdp-39.md)

---

# SDP Offer/Answer procedures for ICE

## 1. Introduction

- ICEがオファー・アンサーでどう使われるかについて

## 2. Conventions

- いつもの

## 3. Terminology

- ICEを使わない場合のデフォルトの値について
- RTPなら`c=`行のアドレスと、`m=`行のプロトコルとポート
- RTCPなら`a=rtcp`属性から
  - それがない場合は、RTPより1大きいポート

## 4. SDP Offer/Answer Procedures

- 本文なし

### 4.1. Introduction

- ICEの仕様にある用語と、オファー・アンサーの仕様にある用語を対照する
  - RFC8445
  - RFC3264
- ICEの処理は、候補が集まってからはじめる

### 4.2. Generic Procedures

- 本文なし

#### 4.2.1. Encoding

- Section 5でSDPの属性の詳細に触れる

##### 4.2.1.1. Data Streams

- `m=`セクションによって、どのようなデータをやり取りしたいかがわかる

##### 4.2.1.2. Candidates

- `m=`セクションごとに送信経路の候補情報をつける
- 経路が決まる前は、`c=`行にデフォルト候補のIPが記載される
  - ポートとプロトコルは、`m=`行に
- 経路が決まると、その候補のIPとポートとプロトコルになる

##### 4.2.1.3. Username and Password

- ICEのユーザー名は、`a=ice-ufrag`属性
- ICEのパスワードは、`a=ice-pwd`属性

##### 4.2.1.4. Lite Implementations

- ICE-Liteの実装は`a=ice-lite`属性をつける
- ICE-Fullの実装はつけてはならない

##### 4.2.1.5. ICE Extensions

- その他のICEのオプションに関しては、`a=ice-options`属性で表す
- この仕様を満たす実装は、`a=ice-options:ice2`をつける
  - そうでない場合は、RFC5245までをサポートしているはず

##### 4.2.1.6. Inactive and Disabled Data Streams

- `m=`セクションが`inactive`の場合や、帯域の指定が`0`の場合は、その他の属性を残す
- しかし`m=`行のポートが`0`にされた場合は、その他の属性をつけるべきではない

#### 4.2.2. RTP/RTCP Considerations

- RTP/RTCPを多重化する場合は、両方に関する属性を同じ`m=`セクションに含める
- 別にポートを使う場合は、`a=rtcp`属性で指定する
  - その場合はRTPの`ポート番号 + 1`
- RTCPをそもそも使わない場合は、`b=RS:0`や`b=RR:0`をつける

#### 4.2.3. Determining Role

- オファー側からICEの処理を開始する
- ICEの役割はICEの仕様通りに決まる
  - Full-Fullの場合は、オファー側が`CONTROLLING`
  - Full-Liteの場合は、Full側が`CONTROLLING`

#### 4.2.4. STUN Considerations

- SDPで経路の候補をオファー・アンサーしたら、いつSTUNのパケットが送信されてもよいように

#### 4.2.5. Verifying ICE Support Procedures

- SDPに`ice-pwd`と`ice-ufrag`属性があるならば、ICEをサポートしてると推測できる
  - 本仕様を満たすならば、同時に`ice-options:ice2`も必要
- 受け取ったSDPの各ストリームごとにICEの処理をすすめる
- RTPのストリームの場合、`a=candidate`には`c=`行のIPと`m=`行のポートが記載される
- 特定のGWを通ると、アドレス情報が書き換えられるかもしれない
  - その場合は、書き換えられたアドレスを使って処理を続けてもよいし
  - `a=ice-mismatch`を返すかも
- デフォルトのトランスポートアドレスは、`0.0.0.0:9`

#### 4.2.6. SDP Example

- ここまで紹介した属性がついたSDPの例

### 4.3. Initial Offer/Answer Exchange

- 本文なし

#### 4.3.1. Sending the Initial Offer

- オファー側が初期オファーを生成するとき
  - `ice-ufrag`と`ice-pwd`属性が必須
- ICE-Fullなら、`ice-pacing`属性も必要
  - 省略した場合はデフォルトの値
  - ICE-Liteなら不要
- 初期オファーに経路情報が載らないこともある
  - それは`prflx`だけを使うつもりという意味

#### 4.3.2. Sending the Initial Answer

- 初期オファーへのアンサーを送信するとき
  - こちらも`ice-ufrag`と`ice-pwd`属性が必須
- ICE-Fullなら、`ice-pacing`属性も必要
  - 省略した場合はデフォルトの値
  - ICE-Liteなら不要
- アンサーでは、オファーと同じプロトコルで`m=`行を用意する
- アンサーを送信したら、接続確認をはじめられる
- ICEを使えない場合は、それにまつわる属性をつけてアンサーしてはいけない
- ICEミスマッチに対する処理はSection 4.2.5

#### 4.3.3. Receiving the Initial Answer

- 初期オファーに対する初期アンサーを受け取ったら
- 接続確認をはじめられる
- ICEミスマッチに対する処理はSection 4.2.5

#### 4.3.4. Concluding ICE

- ICEの処理が終わって、経路が決まったら、それをSDPと照らし合わせる
  - ローカルのICEの情報と、SDPに記載の情報
- `ice2`をサポートしている場合、その経路はデフォルトの経路として以降のセッションで使われる
  - リスタートされるまで
  - サポートしていない場合は、追ってオファーを生成する
- ICEのチェックリストでFailedになったものがあるなら
  - 該当する`m=`セクションのポート値を`0`にしてオファーを再送
- draft-ietf-ice-pacをサポートすれば、`prflx`な候補を見つけられる可能性は高まる

### 4.4. Subsequent Offer/Answer Exchanges

- セッション開始後のオファー・アンサーについて

#### 4.4.1. Sending Subsequent Offer

- 本文なし

##### 4.4.1.1. Procedures for All Implementations

- 本文なし

###### 4.4.1.1.1. ICE Restart

- `c=`行のIPを`0.0.0.0` OR `::`にセットすることは、ICEのリスタートを意味する
- この仕組みは保留のために使ってはならず、そのためには`a=inactive`や`a=sendonly`を使う
- リスタートしたら、`ice-ufrag`と`ice-pwd`を更新する

###### 4.4.1.1.2. Removing a Data Stream

- `m=`行のポートを`0`にしてストリームを削除したなら、その他の属性はつけてはいけない

###### 4.4.1.1.3. Adding a Data Stream

- ストリームを追加する場合
- 初期オファーと同じように（通常通り）

##### 4.4.1.2. Procedures for Full Implementations

- ICE-Fullの実装向けの追加の手順について

###### 4.4.1.2.1. Before Nomination

- 経路確定前
- 初期オファー・アンサーと同じICEの属性をつける
- `prflx`の候補などは追加されるかも
- デフォルトの経路は変更するかも

###### 4.4.1.2.2. After Nomination

- 経路確定後
- そのストリームについては確定した経路のみを含める
  - その他の候補は含めてはいけない
- `CONTROLLING`の側は、確定している証として`a=remote-candidates`属性をつける

##### 4.4.1.3. Procedures for Lite Implementations

- ICE-Liteの実装向けの追加の手順について
- ICEの処理が途中の場合は、すべての経路候補をつける
- Lite実装の場合は、その後のオファー・アンサーの際に、あらたな候補を追加してはいけない
  - そうしたい場合はリスタートが必要

#### 4.4.2. Sending Subsequent Answer

- オファーに`a=remote-candidates`属性がない場合は、オファー側と同じ手順（通常通り）
  - ただし`a=remote-candiadtes`を含めてはいけない
- `CONTROLLED`側は`a=remote-candidates`つきのオファーを受け取るかもしれないし
  - ICEで確定した経路情報を知るかもしれない（オファー側で既にICEが完了している場合）
  - このケースの詳細はAppendix Bにて
- 以下の組み合わせでペアを形成する
  - リモート: オファー側のデフォルト（`m=`行のポートと`c=`行のIP）
  - ローカル: `a=remote-candidates`と同じトランスポートの経路
- そしてこのペアがValidリストにいるかチェックし、ない場合はLosingペアとする

##### 4.4.2.1. ICE Restart

- ICEをリスタートするときは、改めて初期オファーを送る
- アンサー側はそれまでと同じ経路情報を提示してもよい
  - `ice-ufrag`と`ice-pwd`は更新する必要がある

##### 4.4.2.2. Lite Implementation specific procedures

- Lite実装の場合
- 以下の組み合わせでペアを形成する
  - リモート: オファー側のデフォルト（`m=`行のポートと`c=`行のIP）
  - ローカル: `a=remote-candidates`と同じトランスポートの経路
- ICEの処理を完了とする

#### 4.4.3. Receiving Answer for a Subsequent Offer

- 本文なし

##### 4.4.3.1. Procedures for Full Implementations

- 最初のアンサーにはICEの属性があったのに、後続のアンサーでそれがなくなってた場合
  - B2BUA (Back-to-Back User Agent) などの場合にありえる
  - 基本的にこれは予期してないパターンである
- リスタートの場合は、リスタートする
- ストリームの削除・拒否された場合は、Validリストを空にしてSTUNの処理も止める
- ストリームの追加は、通常通りの手順で
- ICEが実行中の場合は、チェックリストを再度作り直すところから

###### 4.4.3.1.1. ICE Restarts

- リスタートする前に、ストリームごとの確定していた経路情報を記憶しておく
- リスタートが完了するまでは、その経路でデータを送受信し続ける

##### 4.4.3.2. Procedures for Lite Implementations

- 新たなValidリストを作り直す
- リスタートが完了するまでは、その経路でデータを送受信し続ける

## 5. Grammar

- この仕様で追加される新たなSDPの属性について
  - `candidate`
  - `remote-candidates`
  - `ice-lite`
  - `ice-mismatch`
  - `ice-ufrag`
  - `ice-pwd`
  - `ice-pacing`
  - `ice-options`

### 5.1. "candidate" Attribute

- `a=candidate`
  - メディアレベル限定
  - STUNの接続確認で使われる情報を含む
- `port`: IPv4かIPv6かは、アドレスに含まれる`:`で判定する
  - IPv6をサポートしてない場合などは、そのcandidateを無視する
- `transport`: 現状は`UDP`のみだが、将来的には拡張される
- `foundation`: アルファベットと数字と`+`と`/`から成る1-32文字
- `component-id`: 1-256の数値
  - `1`からはじまってインクリメントされていく
- `rel-addr`/`rel-port`: `srflx`と`prflx`と`relay`のときに必要
  - `host`のときにはつけてはいけない
  - プライバシー保護などIPを開示したくない場合は、IPは`0.0.0.0` OR `::`でポートは`9`に

### 5.2. "remote-candidates" Attribute

- `a=remote-candidates`
  - メディアレベル限定
  - `CONTROLLING`側のオファーにのみ記載される

### 5.3. "ice-lite" and "ice-mismatch" Attributes

- `a=ice-lite`
  - セッションレベル限定
- `a=ice-mismatch`
  - メディアレベル限定

### 5.4. "ice-ufrag" and "ice-pwd" Attributes

- どちらも
  - メディアレベルかセッションレベルかどちらか
  - 両方に出たらメディアレベル優先
- `a=ice-ufrag`
  - 最低でも24bitのランダムな値
  - = 最低でも4文字
  - 最長256文字だが32文字くらいまでがベター
- `a=ice-pwd`
  - 最低でも128bitランダムな値
  - = 最低でも22文字

### 5.5. "ice-pacing" Attribute

- `a=ice-pacing`
  - セッションレベル限定
- 接続確認のタイマーの間隔を指定するための値
  - デフォルトは`50ms`

### 5.6. "ice-options" Attribute

- `a=ice-options`
  - セッションレベル、メディアレベル
- オファーにそれがある場合は、サポートしておりその機能を使いたい旨を表す

## 6. Keepalives

- キープアライブはもちろん実行する
- `inactive`でも`bandwidth`の指定も関係なく実行する

## 7. SIP Considerations

- SIP関連のため割愛

### 7.1. Latency Guidelines

- SIP関連のため割愛

#### 7.1.1. Offer in INVITE

- SIP関連のため割愛

#### 7.1.2. Offer in Response

- SIP関連のため割愛

### 7.2. SIP Option Tags and Media Feature Tags

- SIP関連のため割愛

### 7.3. Interactions with Forking

- SIP関連のため割愛

### 7.4. Interactions with Preconditions

- SIP関連のため割愛

### 7.5. Interactions with Third Party Call Control

- SIP関連のため割愛

## 8. Interactions with Application Layer Gateways and SIP

- Application Layer Gateways (ALGs)との兼ね合い
  - NAT越えのために使われるもの
- 基本的にはICEに問題はないはず
  - 問題があったら`a=ice-mismatch`を返すだけ
- Session Border Controllers (SBCs)との兼ね合い
  - ICEのcandidateを削除してしまうかも
- SDPに書かれたアドレスが外部のものであれば書き換えない
  - 内部のアドレスな場合は書き換えられる

## 9. Security Considerations

- ICEに関してはRFC8445を参照
- SDPのオファーアンサーに関してはRFC3264を参照

### 9.1. IP Address Privacy

- アドレスとポートを公開したくないときもあるはず
- その場合は、`0.0.0.0`か`::`と、ポートは`9`にする

### 9.2. Attacks on the Offer/Answer Exchanges

- オファー・アンサーが攻撃者によって改ざんされるケース
- TLSなどでセキュアにSDPをやり取りするように

#### 9.3. The Voice Hammer Attack

- 関係ないIP・ポートに対してパケットを送出させるよう仕組む攻撃
  - ICE特有の攻撃ではないけど
  - ICEの情報を受取ると、接続確認でパケットを送信するはずなので

## 10. IANA Considerations

- 本文なし

### 10.1. SDP Attributes

- 7つの新しいSDPの属性を定義した

#### 10.1.1. candidate Attribute

- `a=candidate`
  - メディアレベル

#### 10.1.2. remote-candidates Attribute

- `a=remote-candidates`
  - メディアレベル

#### 10.1.3. ice-lite Attribute

- `a=ice-lite`
  - セッションレベル

#### 10.1.4. ice-mismatch Attribute

- `a=ice-mismatch`
  - メディアレベル

#### 10.1.5. ice-pwd Attribute

- `a=ice-pwd`
  - セッションレベルかメディアレベル

#### 10.1.6. ice-ufrag Attribute

- `a=ice-ufrag`
  - セッションレベルかメディアレベル

#### 10.1.7. ice-options Attribute

- `a=ice-options`
  - セッションレベル

#### 10.1.8. ice-pacing Attribute

- `a=ice-pacing`
  - セッションレベル

### 10.2. Interactive Connectivity Establishment (ICE) Options Registry

- `a=ice-options`の値は、20文字以下が推奨

### 10.3. Candidate Attribute Extension Subregistry Establishment

- Candidate Attribute Extensionsを追加
- 追加する場合はRFC8126にあるポリシーに従う

## Appendix A. Examples

- SDPの例

## Appendix B. The remote-candidates Attribute

- `a=remote-candidates`が存在する理由を示す例
  - R側がSTUNのレスポンスを受信できなかったとき
  - L側から確認できてることを伝えてあげる

## Appendix C. Why Is the Conflict Resolution Mechanism Needed?

- お互いに`CONTROLLING` OR `CONTROLLED`だと認識してしまう場合について
  - シグナリングプレーンで解消できる問題ではある

## Appendix D. Why Send an Updated Offer?

- オファーの更新がなぜ必要なのかについて
- オファー・アンサーの結果を確信するために送ってるなら、不要ともいえる
- ただし中継機など様々な要因を認識するために必須としている
  - WebRTCではつかってない
