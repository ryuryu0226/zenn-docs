---
title: "UnityからPythonモジュールを呼び出す【Python.NET】"
emoji: "📘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Unity", "python", "csharp"]
published: true
---

# はじめに

[pythonnet (Python.NET)](http://pythonnet.github.io/) を使用して，Unity上でPythonモジュールを呼び出す方法をまとめました．Pythonの豊富な機械学習系のライブラリとUnityのゲームエンジンを組み合わせて使いたいときに有用です．

pythonnetをUnity上で使用する解説記事がないのが現状で，[クラッシュ報告](https://github.com/pythonnet/pythonnet/issues/701)もあるため，記事を作成しました．

# Python.NETとは？
pythonnetは，.NETからPython (CPython) コードを呼び出すことができます．反対に，Pythonコードから.NETを呼び出すことも可能です．一方，[IronPython](https://ironpython.net/)は，python3.x系の開発が進んでいなかったり，[CPythonとの互換性](https://github.com/IronLanguages/ironpython3/blob/v3.4.0-beta1/Documentation/differences-from-c-python.md)も十分でなかったりと，本用途には適していません．

pythonnetの特徴は，Pythonコードから中間言語を生成せず，CPythonと.NET / Monoを統合するため，Pythonコードのネイティブ実行速度を維持したまま.NETの機能を使用することができます．（IronPythonはマネージコード実装）

# 準備
まずは，pythonnetをビルドします．[githubリポジトリ](https://github.com/pythonnet/pythonnet)からcloneして，ソリューションファイルpythonnet.slnを開きます．

pythonnet.slnから，`Python.Runtime.dll`をビルドします．ビルド方法やその他の環境構築については，[こちらの記事](https://qiita.com/hogegex/items/3743225a7af13e93df78)に詳しくまとめられています．

作成した`Python.Runtime.dll`をUnityプロジェクトにインポートし，`Api Compatibility Level`を`.NET 4.x`に変更します．

# サンプルコード
以下は，numpyモジュールをUnity上で実行した例です．
`PythonEngine.Initialize()`の引数を設定しない場合，この[issue](https://github.com/pythonnet/pythonnet/issues/701)にあるように，二回目実行時にクラッシュします．

```cs
using System.Collections.Generic;
using UnityEngine;
using Python.Runtime;

void Start()
{
    PythonEngine.Initialize(mode: ShutdownMode.Reload); // 初期化
    using (Py.GIL()) // Global Interpreter Lockを取得
    {
        dynamic np = Py.Import("numpy");

        // ndim: 1
        dynamic a = np.array(new List<float> {1.0f, 2.0f});
        Debug.Log(a.shape);
        Debug.Log(a*a);
        
        // ndim: 2
        dynamic b = np.array(new List<List<float>> {new List<float> {1.0f, 2.0f}, new List<float> {3.0f, 4.0f}}, dtype: np.float64);
        Debug.Log(b.shape);
        Debug.Log(b*b);
    }
    PythonEngine.Shutdown(); // python環境を破棄
}
```

次の記事では，PythonモジュールとUnityの3Dオブジェクトを組み合わせた実装を紹介したいと思います．

# 参考文献
[pythonnet.github.io](http://pythonnet.github.io/)
[.NET (C#)からpythonを呼び出す](https://qiita.com/hogegex/items/3743225a7af13e93df78)
