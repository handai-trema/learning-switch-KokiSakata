# 情報ネットワーク学演習Ⅱ 10/12 レポート課題

## 課題内容 (複数スイッチ対応版 ラーニングスイッチ)

複数スイッチに対応したラーニングスイッチ (multi_learning_switch.rb) の動作を説明しよう。

* 複数スイッチの FDB をどのように実現しているか、コードと動作を解説する
* 動作の様子やフローテーブルの内容もステップごとに確認すること
* 必要に応じて図解すること

![](multi_learning_switch.jpeg)

## 解答

### コード解説
この節では、multi_learning_switch.rbの各メソッドを説明する。

* startメソッド
```ruby
  def start(_argv)
    @fdbs = {}
    logger.info "#{name} started."
  end
```
初めにFDB（フォワーディングデータベース）を初期化しプログラムが開始したことを出力する。

* switch_readyメソッド
```ruby
  def switch_ready(datapath_id)
    @fdbs[datapath_id] = FDB.new
  end
```
起動してきたスイッチに対してそれぞれdatapath_idをキーとしてFDBのインスタンスを生成する。

* packet_inメソッド
```ruby
  def packet_in(datapath_id, packet_in)
    return if packet_in.destination_mac.reserved?
    @fdbs.fetch(datapath_id).learn(packet_in.source_mac, packet_in.in_port)
    flow_mod_and_packet_out packet_in
  end
```
初めに、未知のパケットでなければ、その時点で終了してパケットを破棄している。
未知のパケットであれば、そのパケットが届いたスイッチのFDBにパケットの送信元MACアドレスとポート番号を登録し、パケットを宛先のポートに出力するかフラッディングする。

* age_fdbsメソッド
```ruby
  def age_fdbs
    @fdbs.each_value(&:age)
  end
```
このメソッドは5秒毎に実行され、すべてのスイッチのFDBのすべてのエントリをエージングし、寿命を過ぎたものを消す。

* flow_mod_and_packet_outメソッド
```ruby
  def flow_mod_and_packet_out(packet_in)
    port_no = @fdbs.fetch(packet_in.dpid).lookup(packet_in.destination_mac)
    flow_mod(packet_in, port_no) if port_no
    packet_out(packet_in, port_no || :flood)
  end
```
初めに、パケットが到着したスイッチのFDBと宛先MACアドレスから出力先のポート番号を検索する。
もしポート番号が見つかれば、以降は同じパケットは同様に転送せよ、というフローエントリをスイッチに書き込み、パケットをそのポートに出力する。
見つからなければ、パケットをフラッディングしてばらまく。

* flow_modメソッド
```ruby
  def flow_mod(packet_in, port_no)
    send_flow_mod_add(
      packet_in.datapath_id,
      match: ExactMatch.new(packet_in),
      actions: SendOutPort.new(port_no)
    )
  end
```
Packet Inメッセージとポート番号からスイッチに新しいフローエントリを登録する。


* packet_outメソッド
```ruby
  def packet_out(packet_in, port_no)
    send_packet_out(
      packet_in.datapath_id,
      packet_in: packet_in,
      actions: SendOutPort.new(port_no)
    )
  end
```
Packet Inメッセージから送られてきたパケットの情報を得て、それを指定されたポートに出力する。

### 動作確認
今回、サンプルとして配布されたtrema.multi.confによって構成されるネットワークを用いて動作確認を行う。そのモデル図を以下に示す。

![](trema.multi1.JPG)

4つのスイッチにそれぞれ2つのホストが接続され、すべてのスイッチが1つのコントローラに接続されている。
今回、以下のようなシナリオを想定する。

	1. host1-1からhost1-2にパケットを送信する。
	2. host1-2からhost1-1にパケットを送信する。
	3. host1-1からhost1-2にパケットを送信する。


1. host1-1からhost1-2にパケットを送信する。

スイッチ0x1は未知のパケットが送られてきたので、コントローラにPacket Inを送る。
![](trema.multi2.JPG)
コントローラは送信元MACアドレスとポート番号をFDBに登録して、宛先MACアドレスを検索する。
宛先MACアドレスが見つからなかったので、フラッディングしてパケットをばらまくようにPacket Outを送信する。　
![](trema.multi3.JPG)

2. host1-2からhost1-1にパケットを送信する。

スイッチ0x1は未知のパケットが送られてきたので、コントローラにPacket Inを送る。
![](trema.multi4.JPG)
コントローラは送信元MACアドレスとポート番号をFDBに登録して、宛先MACアドレスを検索する。
宛先MACアドレスが見つかったので、そのポートにパケットを送るようにPacket Outと、フローテーブルに新しいエントリを登録するためにFlow Modを送信する。
![](trema.multi5.JPG)

3. host1-1からhost1-2にパケットを送信する。

スイッチ0x1は送られてきたパケットの転送先ポートを知っているので、Packet Inがコントロールまで上がることなく、スイッチ側だけでパケットを転送している。
![](trema.multi6.JPG)





