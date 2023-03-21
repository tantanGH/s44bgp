# S44BGP.X

16bit PCM background player for X680x0/Human68k

.S44/.A44 データをハイメモリに抱えて常駐し、Mercury-UNITでバックグラウンド再生を行うプレーヤーです。
KMD歌詞の簡易表示にも対応しています。

---

### 前提条件

本プログラムは Mercury-UNIT + ハイメモリ専用です。以下が必須になります。

Mercury-UNIT V3 以上 および PCM8PP ドライバ

- [PCM8PP.X](http://retropc.net/x68000/software/hardware/mercury/pcm8pp/)

16MB以上のハイメモリ および 下記いずれかのハイメモリドライバ

- [060turbo.sys](http://retropc.net/x68000/software/hardware/060turbo/060turbo_sys/)
- [TS16DRVp.X](https://t.co/qJDbBEiJsS)


動作確認は以下でのみ行なっています。

- X68030 (25MHz) + 060turbo + ハイメモリ128MB + PCM8PP.X + Mercury-UNIT V3 (実機)
- XM6 Type G + X68030 (25MHz) + TS-6BE16 ハイメモリ16MB + TS16DRVp.X + PCM8PP.X + Mercury-UNIT V3.5 (エミュレータ)

おそらく Phantom-X + Kepler-X の環境でも動作すると思いますが、未確認です。無事に動作した場合はぜひお知らせください。


---

### インストール方法

S44BPxxx.ZIP をダウンロードして展開し、S44BGP.X をパスの通ったディレクトリに置きます。

PCM8PP.X および ハイメモリドライバが有効になっていることを確認してください。

---

### 割り込みに関して

S44BGP.X は Timer-D の割り込みを使用します。Timer-Dを使うアプリケーションと一緒に動作させることはできません。

また、Timer-Dを使うHuman68k のバックグラウンド機能も利用できません。CONFIG.SYS で PROCESS= の行が有効になっている場合は、本プログラム利用時はコメントアウトする必要があります。

---

### バックグラウンド再生中の注意点

Mercury-UNIT やPCM8PPを利用するアプリケーションとは競合しますので同時に動作させないようにしてください。

重い処理はお勧めしません。テキストエディタでの文書編集、ビュワーでのパソコン通信のログ確認などはokです。RS232Cはおそらく取りこぼします。

ディスクアクセスを多用するとDMACアクセスが競合してプチノイズが乗る場合があります。

何らかの異常状態になった場合、大音量でノイズが再生される可能性があります。ヘッドフォンの利用時は特にご注意ください。

なお、S44BGP.X 自体は一旦常駐した後にディスクアクセスは一切行いません。

---

### ファンクションキー表示部

S44BGP.X は最下段のファンクションキー表示行の一部にインフォメーションを表示します。ファンクションキー表示を使う他アプリケーションを動作させた場合は競合してそのアプリケーションの期待する正しい表示になりませんのでご注意ください。

これはオプションで無効にすることも可能です。

---

### 起動方法

    s44bgp [オプション] <PCMファイル名(.s44|.a44)> [PCMファイル名(.s44|.a44) ...]

      -r    ... 再生を停止してハイメモリエリアを開放すると共に常駐解除します
      -h    ... ヘルプメッセージを表示します

      -i <file> ... インダイレクトファイルを使用します

      -v<n> ... ボリューム指定(1-12, default:8)
      -s    ... シャッフルモード
      -q    ... インフォメーション表示・KMD歌詞表示を行いません

      -2    ... 22.05kHz mode
      -8    ... 8bit PCM mode
      -m    ... mono mode


コマンドラインで再生対象とするPCMファイル名を指定して起動します。複数指定可能です。ただしサポートしているのは .s44 (16bit big endian raw PCM 44.1kHz stereo)と .a44 (YM2608 ADPCM 44.1kHz stereo) 形式のみです。これら以外には対応していません。

必要に応じて NOZ氏の PCM3PCM.X などのツールで変換してください。

インダイレクトファイルは、1行に1ファイル名を記述したテキストファイルです。長いコマンドラインで複数のPCMファイル名を与える代わりに利用できます。

22.05kHzモードはハードウェアとして未対応のMercury-UNIT V3.5未満であってもPCM8PPがその差異を吸収してくれるので利用可能です。

---

### 操作方法

常駐完了後再生がすぐに始まります。常駐している間は以下のキー操作に対応しています。

    CTRL + XF4 ... 一時停止および再開
    CTRL + XF5 ... 次の曲へ

完全に停止させるには常駐解除してください。

    s44bgp -r

---

### 必要なメモリサイズの計算

44.1kHz 16bit stereo 240秒のPCMデータサイズは

    44100 * 2(16bit) * 2(stereo) * 240(秒) = 42,336,000

と計算でき、約42MBということになります。これはS44のデータサイズそのものになります。
ADPCM圧縮されたA44であっても、展開した状態でメモリに置くことになるのでフットプリントは変わりません。

そのまま060turboの128MBハイメモリに置くとなると、目一杯使っても3曲までです。
そこで再生周波数を半分の22.05kHz(`-2`オプション)とすれば、必要サイズは半分で済むので最大6曲程度まではいけることになります。

PhantomXの384MBハイメモリであれば、アルバム丸ごとも可能かと思います。ただし最初のロード時間はその分掛かります。

TS-6BE16の16MBハイメモリには、これでは1曲すら収まりません。
さらにbit数を半分の8bit(`-8`オプション)にすれば、サイズは約10MBとなるので、何とか1曲は置くことが可能です。

なお、RAMDISKに置くわけではないので、ハイメモリを既にRAMDISKとして使っている場合は、当然ながら利用可能な領域は減ります。
PCMデータロード時に利用可能なハイメモリエリアのサイズが表示されますので参考にしてください。

---

### KMD歌詞簡易再生

PCMファイルと同じ主ファイル名で、拡張子.kmdのKMD歌詞データファイルが存在した場合、それを自動的に読み込んで歌詞表示を行うことが可能です。

ただしあくまで簡易的な表示で以下のような制限があります。
 - y位置は無視されすべて1行に表示される
 - 歌詞の消去は行わない
 - 曲の出だし500ms分の歌詞表示は行わない(S44BGPのファイル名表示のため)

KMDについて詳しくは [KMDED.X](https://github.com/tantanGH/kmded)を参照してください。

---

### Special Thanks

* PCM8PP.X thanks to たにぃさん
* ADPCMLIB thanks to Otankonas さん
* 060turbo.sys thanks to M.Kamadaさん
* TS16DRVp.X thanks to M.Kamadaさん / はうさん / みゆ🌹ฅ^•ω•^ฅ さん
* XM6 TypeG thanks to PI.さん / GIMONSさん
* xdev68k thanks to ファミべのよっしんさん
* HAS060.X on run68mac thanks to YuNKさん / M.Kamadaさん / GOROmanさん
* HLK301.X on run68mac thanks to SALTさん / GOROmanさん

---

### History

* 0.3.0 (2023/03/20) ... 初版
