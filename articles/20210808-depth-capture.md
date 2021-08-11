---
title: "【Unity】深度画像を保存する"
emoji: "🎞"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Unity","csharp"]
published: true
---

深度画像を実装に利用したり，Game Viewに描画する方法はネット上に散見されるのですが，画像ファイルとして保存するのに苦労したので，その方法を共有したいと思います．


# 深度値を取得する
`RenderTextureFormat.Depth`は，深度値を`RenderTexture`にレンダリングするために使用されます．また，画像ファイルとして保存するため，カメラのレンダリング先を`RenderTexture`に変更します．以下に，深度値取得に必要なスクリプトを記載します．今回は，キャプチャ用のカメラをメインカメラとは別に用意することを想定しています．
```cs
CaptureCamera.depthTextureMode = DepthTextureMode.Depth;
CaptureCamera.targetTexture = new RenderTexture(width, height, 16, RenderTextureFormat.Depth);
```


# 画像ファイルとして保存する
深度バッファは直接アクセスできないため，Shaderを使って`ReadPixels`でエンコードできる形に変換する必要があります．変換には，`Graphics.Blit`を利用します．保存用のソースコードは次のようになります．

```cs
private IEnumerator Depth_Capture(Texture2D tex, Camera camera)
{
    yield return new WaitForEndOfFrame();

    // RenderTexture.active = camera.targetTexture; // これは無理
    RenderTexture.active = new RenderTexture(tex.width, tex.height, 0);
    Graphics.Blit(camera.targetTexture, RenderTexture.active);

    tex.ReadPixels(new Rect(0, 0, camera.targetTexture.width, camera.targetTexture.height), 0, 0);
    tex.Apply();

    byte[] bytes = tex.EncodeToPNG();
    File.WriteAllBytes(@"capture\fixation" + frame + ".png", bytes);

    yield break;
}
```
このコルーチンの使い方は，スクリプトからシーンをキャプチャする方法と同様です．詳しくは[以前のブログ](https://zenn.dev/ryuryu/articles/20210730-vr-capture)に書いてます．

# （おまけ）Shaderでポストエフェクト

`Graphics.Blit`では，Shaderを使ってテクスチャにポストエフェクトをかけることもできます．
```cs
Graphics.Blit(camera.targetTexture, RenderTexture.active, material);
```






# 参考文献
[Using Depth Textures](http://www.cis.sojo-u.ac.jp/~izumi/Unity_Documentation_jp/Documentation/Components/SL-DepthTextures.html)
[How to access rendered depth buffer properly?](https://forum.unity.com/threads/how-to-access-rendered-depth-buffer-properly.158237/)
[【Unity】カメラのレンダリング対象の変更と、Blitによるレンダリングについて](https://light11.hatenadiary.com/entry/2018/04/05/195745)
[【Unity】【シェーダ】カメラから見た深度を描画する](https://light11.hatenadiary.com/entry/2018/05/08/012149)
