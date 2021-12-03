# TyranoScriptV5Perser
ティラノスクリプトV5のシナリオファイルを読み込んで、タグやデータに分解するパーサーを作成する手順のメモです。

ティラノビルダーで作成した会話シーンを、Unityなどへ移植する時にシナリオファイルをどのように分析すればよいかの例を示します。


## 想定環境
- ティラノスクリプトV5
- Unity2019.4.x

## ティラノスクリプトの基本をおさえる
[TYRANO SCRIPT. 文字を表示する](https://tyrano.jp/usage/tutorial/moji)を読んで、基本的な文法を把握します。

- シナリオはプロジェクトフォルダーの`data\scenario`フォルダー内にある
- `;`から始まる行は行末までコメント
- `*`から始まる行はラベル定義
- `[]`で囲まれた部分はタグで、何らかの動きを表す
- タグは`@`で始めることも可能。`[]`は文章中に埋め込むことができるのに対して、`@`は1行で完結させる必要がある
- タグには属性を持つものがある。属性は半角スペースで区切り、パラメーターを持つ場合は`=`に続けて書く。パラメーターは`"`で囲む場合と、囲まない場合がある
- `#`で始まる行は、キャラ名の指定。詳しくは https://tyrano.jp/usage/tech/chara
- 空行は無視
- 上記以外の文字はセリフ

まずは以上を前提に、スクリプトファイルをコマンドとパラメーターに分割する機能を作成する。

## 分割の仕方
渡されたテキストファイルを上記のルールに基づいて分解して、コマンドと属性と属性の値に分解します。例えば、以下のデフォルトスクリプトを渡したとします。

```ks
;チュートリアル用スクリプトファイル
*start

[wait time=200]

吾輩わがはいは猫である。名前はまだ無い。[l][r]

どこで生れたかとんと見当けんとうがつかぬ。[l][cm]

何でも薄暗いじめじめした所でニャーニャー泣いていた事だけは記憶している。[l]

吾輩はここで始めて人間というものを見た。[l][r]
```

これを以下のように出力します。

```
label value=start
wait time=200
serif value=吾輩わがはいは猫である。名前はまだ無い。
l
r
serif value=どこで生れたかとんと見当けんとうがつかぬ。
l
cm
serif value=何でも薄暗いじめじめした所でニャーニャー泣いていた事だけは記憶している。
l
serif value=吾輩はここで始めて人間というものを見た。
l
r
```

もう一つ例示します。

```ks
[position layer=message0 width=800 height=300 top=380 left=70 ]
[position layer=message0 page=fore frame="frame.png" margint="65" marginl="50" marginr="70" marginb="60"]
[cm]
メッセージウィンドウが下に表示されましたね？[r][l]
ここにメッセージが表示されています。[r][l]
ここにメッセージが表示されています。[r][l]
```

以下のように出力します。

```
position layer=message0 width=800 height=300 top=380 left=70
position layer=message0 page=fore frame=frame.png margint=65 marginl=50 marginr=70 marginb=60
cm
serif value=メッセージウィンドウが下に表示されましたね？
r
l
serif value=ここにメッセージが表示されています。
r
l
serif value=ここにメッセージが表示されています。
r
l
```

このように、コマンドや空行などを取り除いて、最初にコマンドを表す文字列、属性があれば半角スペースで区切って続けます。属性の名前がない場合は、`value`を名前として、`=`に続けて値を記載します。値がない場合は`=`以降は省略します。

このように分解することで、最初の文字列で処理を分岐して、分岐先で属性を読み取って必要な処理を実行するようにすれば、プレイヤーを実装することができます。

## Unityでのテキストファイルの扱い方
UnityはテキストファイルをTextAssetを使って簡単に扱うことができます。以下のようなテキストファイルを用意して読み込む例を示します。文字エンコードは UTF-8 である必要があります。

messages.txt
```
今は9:00です。
おはようございます、デジタルアーツ太郎さん。
１日頑張りましょう
```

スクリプトは以下の通り。

TextLog.cs
```cs
using UnityEngine;

public class TextLog : MonoBehaviour
{
    [SerializeField]
    TextAsset messages = default;

    void Start()
    {
        var mes = messages.text.Split('\n');
        for (int i = 0; i < mes.Length; i++)
        {
            Debug.Log(mes[i].Trim());
        }
    }
}
```

上記のスクリプトを何らかのゲームオブジェクトにアタッチして、Inspectorウィンドウで `messages.txt` を Messages 欄にドラッグ&ドロップすることで動かすことができます。テキストファイルの読み込み方と、何らかの区切り文字(今回は改行である\n)で分割する方法が分かれば、おおよそのテキストファイルは扱えます。

- [Split('区切り文字')](https://docs.microsoft.com/ja-jp/dotnet/api/system.string.split?view=net-6.0) 文字列を指定の区切り文字で分割して、文字配列を返します
- [Trim()](https://docs.microsoft.com/ja-jp/dotnet/api/system.string.trim?view=net-6.0) 空白や改行などの空白文字を削除した文字列を返します
- [IndexOf("文字列")](https://docs.microsoft.com/ja-jp/dotnet/api/system.string.indexof?view=net-6.0) 指定の文字列が始まる位置を返します。戻り値が0なら先頭が指定の文字列、1以上なら文字列が含まれている、-1なら文字列が含まれないことを示します



## 参考URL
- [TYRANO SCRIPT V5](https://tyrano.jp/usage/tutorial/ready_v5)

## ライセンス
MITライセンス
Copyright (C) 2021 たなかゆう
