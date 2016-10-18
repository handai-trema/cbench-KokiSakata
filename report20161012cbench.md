# 情報ネットワーク学演習Ⅱ 10/12 レポート課題

## 課題内容 (Cbench のボトルネック調査)

Ruby のプロファイラで Cbench のボトルネックを解析しよう。

以下に挙げた Ruby のプロファイラのどれかを使い、Cbench や Trema のボトルネック部分を発見し遅い理由を解説してください。

##解答
まず初めに、１つの端末から以下のコマンドを実行した。

	gem install ruby-prof

これにより、ruby-profの実行環境ができる。

次に、以下のコマンドを実行した。

	ruby-prof ./bin/trema run ./lib/cbench.rb &> result.txt

これにより、実行終了後にresult.txtファイルにプロファイルが書き込まれる。

次に、別の端末から以下のコマンドを実行した。

	./bin/cbench --port 6653 --switches 1 --loop 10 --ms-per-test 10000 --delay 1000 --throughput

これにより、cbenchのボトルネックの調査を行うことができる。
このプログラムが終了してから、１つ目の端末でCtrl-Cを押してプログラムの実行を終了させる。
このようにして、cbenchのボトルネック調査の結果がresult.txtに書き込まれる。
その結果の一部分を以下に示す。

	 %self      total      self      wait     child     calls  name
	 　2.66     31.038     4.373     0.001    26.664   427841   BinData::Struct#instantiate_obj_at
	  2.26     15.849     3.714     0.000    12.135   617119  *BinData::BasePrimitive#_value
	  1.88      3.257     3.086     0.000     0.170   383634   BinData::Base#get_parameter
	  1.83      4.689     3.014     0.000     1.675   373383   Kernel#define_singleton_method
	  1.83      3.006     3.006     0.000     0.000   266311   String#sub
	  1.82     27.962     2.993     0.000    24.969   266303   BinData::Struct#find_obj_for_name
	  1.80      2.965     2.965     0.000     0.000   266303   Array#index
	  1.71      9.768     2.805     0.000     6.963   266303   BinData::Struct#base_field_name
	  1.53     29.862     2.521     0.000    27.340   232051   BinData::SanitizedPrototype#instantiate
	  1.51      7.360     2.487     0.000     4.873   170939   Kernel#dup

同様にして、fast_cbench.rbのcbenchのボトルネックの調査を行った結果を以下に示す。

	 %self      total      self      wait     child     calls  name
	  3.47     25.154     3.984     0.000    21.170   350694   BinData::Struct#instantiate_obj_at
	  3.24      3.848     3.712     0.000     0.136   369434   BinData::Base#get_parameter
	  2.73      5.273     3.126     0.000     2.146   360183   Kernel#define_singleton_method
	  2.24      8.309     2.573     0.000     5.736   226903   BinData::Struct#base_field_name
	  2.19     15.779     2.517     0.000    13.262   226903   BinData::Struct#find_obj_for_name
	  2.14      4.014     2.458     0.000     1.555   306835  *BinData::BasePrimitive#_value
	  2.13      2.443     2.443     0.000     0.000   226911   String#sub
	  2.08      2.383     2.383     0.000     0.000   226903   Array#index
	  1.99      2.282     2.282     0.000     0.000   226903   String#to_sym
	  1.94      2.228     2.228     0.000     0.000   263861   Hash#[]=

各行の意味を以下に示す。
* %self：全体に対してこのメソッドが占める時間の割合
* total：このメソッドとその子メソッドが占める時間
* self：このメソッドが占める時間
* wait：待ち時間
* child：子メソッドが占める時間
* calls：このメソッドが呼び出された回数
* name：メソッドの名前

以上より、cbench.rbでは呼び出し回数が400000を超えている、"BinData::Struct#instantiate_obj_at"と"*BinData::BasePrimitive#_value"の呼び出された回数が大幅に減少している。
この部分がボトルネックであると考えられ、遅い理由はメソッドの呼び出し回数が多いからである考えられる。
