# 情報ネットワーク学演習Ⅱ 10/12 レポート課題

## 課題内容 (複数スイッチ対応版 ラーニングスイッチ)

複数スイッチに対応したラーニングスイッチ (multi_learning_switch.rb) の動作を説明しよう。

* 複数スイッチの FDB をどのように実現しているか、コードと動作を解説する
* 動作の様子やフローテーブルの内容もステップごとに確認すること
* 必要に応じて図解すること

![](multi_learning_switch.jpeg)

## 解答

### コード解説
multi_learning_switch.rbの
* startメソッド
'''ruby
  def start(_argv)
    @fdbs = {}
    logger.info "#{name} started."
  end
'''

* switch_readyメソッド

* packet_inメソッド

* age_fdbsメソッド

* flow_mod_and_packet_outメソッド

* flow_modメソッド

* packet_outメソッド

### 動作確認
