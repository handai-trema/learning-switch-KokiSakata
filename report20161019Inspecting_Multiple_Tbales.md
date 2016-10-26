# 情報ネットワーク学演習Ⅱ 10/19 レポート課題
## 課題内容 (マルチプルテーブルを読む)

OpenFlow1.3 版スイッチの動作を説明しよう。

スイッチ動作の各ステップについて、trema dump_flows の出力 (マルチプルテーブルの内容) を混じえながら動作を説明すること。

### 実行のしかた

以下のように `--openflow13` オプションが必要です。

``` shellsession
$ bundle exec trema run lib/learning_switch13.rb --openflow13 -c trema.conf
```

## 解答

今回、以下の4つのステップでtrema dump_flowsを実行し、マルチプルテーブルの内容を参照する。
1. 初期状態
2. host1からhost2にパケットを送信
3. host2からhost1にパケットを送信
4. 再びhost1からhost2にパケットを送信
このようにして、ポート番号がホストのMACアドレスがFDBに登録され、パケットの宛先ポートがスイッチのフローエントリに登録され、それを用いてコントローラを用いずにパケットを転送する、という流れを確認する。

### 1. 初期状態
```
cookie=0x0, duration=36.164s, table=0, n_packets=0, n_bytes=0, priority=2,dl_dst=01:00:00:00:00:00/ff:00:00:00:00:00 actions=drop
cookie=0x0, duration=36.127s, table=0, n_packets=0, n_bytes=0, priority=2,dl_dst=33:33:00:00:00:00/ff:ff:00:00:00:00 actions=drop
cookie=0x0, duration=36.127s, table=0, n_packets=0, n_bytes=0, priority=1 actions=goto_table:1
cookie=0x0, duration=36.127s, table=1, n_packets=0, n_bytes=0, priority=3,dl_dst=ff:ff:ff:ff:ff:ff actions=FLOOD
cookie=0x0, duration=36.127s, table=1, n_packets=0, n_bytes=0, priority=1 actions=CONTROLLER:65535

```
### 2. host1からhost2にパケットを送信
```
cookie=0x0, duration=120.016s, table=0, n_packets=0, n_bytes=0, priority=2,dl_dst=01:00:00:00:00:00/ff:00:00:00:00:00 actions=drop
cookie=0x0, duration=119.979s, table=0, n_packets=0, n_bytes=0, priority=2,dl_dst=33:33:00:00:00:00/ff:ff:00:00:00:00 actions=drop
cookie=0x0, duration=119.979s, table=0, n_packets=1, n_bytes=42, priority=1 actions=goto_table:1
cookie=0x0, duration=119.979s, table=1, n_packets=0, n_bytes=0, priority=3,dl_dst=ff:ff:ff:ff:ff:ff actions=FLOOD
cookie=0x0, duration=119.979s, table=1, n_packets=1, n_bytes=42, priority=1 actions=CONTROLLER:65535
```

### 3. host2からhost1にパケットを送信
```
cookie=0x0, duration=186.683s, table=0, n_packets=0, n_bytes=0, priority=2,dl_dst=01:00:00:00:00:00/ff:00:00:00:00:00 actions=drop
cookie=0x0, duration=186.646s, table=0, n_packets=0, n_bytes=0, priority=2,dl_dst=33:33:00:00:00:00/ff:ff:00:00:00:00 actions=drop
cookie=0x0, duration=186.646s, table=0, n_packets=2, n_bytes=84, priority=1 actions=goto_table:1
cookie=0x0, duration=186.646s, table=1, n_packets=0, n_bytes=0, priority=3,dl_dst=ff:ff:ff:ff:ff:ff actions=FLOOD
cookie=0x0, duration=2.76s, table=1, n_packets=0, n_bytes=0, idle_timeout=180, priority=2,in_port=2,dl_src=c6:73:74:e5:71:88,dl_dst=ad:4a:5e:6c:b5:20 actions=output:1
cookie=0x0, duration=186.646s, table=1, n_packets=2, n_bytes=84, priority=1 actions=CONTROLLER:65535
```

### 4. 再びhost1からhost2にパケットを送信
```
cookie=0x0, duration=245.165s, table=0, n_packets=0, n_bytes=0, priority=2,dl_dst=01:00:00:00:00:00/ff:00:00:00:00:00 actions=drop
cookie=0x0, duration=245.128s, table=0, n_packets=0, n_bytes=0, priority=2,dl_dst=33:33:00:00:00:00/ff:ff:00:00:00:00 actions=drop
cookie=0x0, duration=245.128s, table=0, n_packets=3, n_bytes=126, priority=1 actions=goto_table:1
cookie=0x0, duration=245.128s, table=1, n_packets=0, n_bytes=0, priority=3,dl_dst=ff:ff:ff:ff:ff:ff actions=FLOOD
cookie=0x0, duration=61.242s, table=1, n_packets=0, n_bytes=0, idle_timeout=180, priority=2,in_port=2,dl_src=c6:73:74:e5:71:88,dl_dst=ad:4a:5e:6c:b5:20 actions=output:1
cookie=0x0, duration=3.285s, table=1, n_packets=0, n_bytes=0, idle_timeout=180, priority=2,in_port=1,dl_src=ad:4a:5e:6c:b5:20,dl_dst=c6:73:74:e5:71:88 actions=output:2
cookie=0x0, duration=245.128s, table=1, n_packets=3, n_bytes=126, priority=1 actions=CONTROLLER:65535
```

