---
title: "【Unity】VRシーンをスクリプトからキャプチャして透視投影変換もする"
emoji: "🎈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Unity", "VR"]
published: true
---

VRシーンをスクリプトからキャプチャして，画像上でオブジェクトの位置を取得しようとしたときに，至る所でハマったので要点をまとめました．キャプチャする方法だけ知りたい人は前半部分だけ読んでください．

下の実装例では，ユーザが見ているVRシーンをキャプチャしています．球の中心座標を透視投影変換し，キャプチャ画像上に赤い点でプロットしています．

![](/images/20210730/movie.gif =250x)


# カメラの制御を切る
キャプチャ用のカメラをシーンに配置すると，勝手にVRカメラとして認識され，カメラの姿勢やFoVがVRデバイスによって制御されてしまいます．そのため，VRデバイスからの制御を切る必要があります．

```cs
XRDevice.DisableAutoXRCameraTracking(target, false);
```

上記で制御を切れますが，シーンを再生する一瞬だけ制御が働き，カメラの姿勢やFoVが更新されてしまいます．VRデバイスからの制御を切った後，姿勢とFoVを再設定しなければなりません．

また，VRシーンをテクスチャにレンダリングするために，カメラに`RenderTexture`を割り当てても，VRデバイスからの制御を切ることができます．

```cs
CaptureCamera.targetTexture = new RenderTexture(width, height, 0);
CaptureCamera.transform.localPosition = Vector3.zero;
CaptureCamera.transform.localRotation = Quaternion.identity;
CaptureCamera.fieldOfView = fov;
```

# VRシーンをキャプチャする
フレーム毎に，`RenderTexture`に格納されているピクセル情報を`Texture2D`に変換し，画像にエンコードして保存します．画像の保存はシーンのレンダリングが完了するまで待つ必要があります．下記にキャプチャ用のソースコードを載せています．

:::details コード
```cs
[SerializeField] private Camera CaptureCamera;
Texture2D tex;
int frame = 0;

private IEnumerator VR_Capture(Texture2D tex, Camera camera)
{
    yield return new WaitForEndOfFrame();

    RenderTexture.active = camera.targetTexture;
    tex.ReadPixels(new Rect(0, 0, camera.targetTexture.width, camera.targetTexture.height), 0, 0);
    tex.Apply();

    byte[] bytes = tex.EncodeToJPG(75);
    File.WriteAllBytes(@"capture\"+ frame +".jpg", bytes);

    yield break;
}

void Start()
{
    int width = 800;
    int height = 800;
    tex = new Texture2D(width, height, TextureFormat.RGB24, false);

    CaptureCamera.targetTexture = new RenderTexture(width, height, 0);

    CaptureCamera.transform.localPosition = Vector3.zero;
    CaptureCamera.transform.localRotation = Quaternion.identity;
    CaptureCamera.fieldOfView = 55;
}

void Update()
{
    StartCoroutine(VR_Capture(tex, CaptureCamera));
    frame++;
}
```
:::


# 焦点距離の単位をmmからpxに変換する
CameraのPysical Cameraプロパティから，焦点距離をミリメートル単位で取得することができます．シーン中のオブジェクトが，画像上でどの位置にあるかをを求めるには，焦点距離をピクセルに変換する必要があります．

単位の変換式は，解像度$\textrm{Resolution}$とセンサーサイズ$\textrm{SensorSize}$を用いて，次のように表されます．

$$ f^{\prime}~\textrm{[px]} = f~\textrm{[mm]} \times \frac{\textrm{Resolution~[px]}}{\textrm{SensorSize~[mm]}} $$

$f$と$\textrm{SensorSize}$はそれぞれ，`Cameara.focalLength`と`Cameara.sensorSize`で取得可能です．また，Pysical Cameraプロパティで，焦点距離やセンサーサイズなどを変更できます．$\textrm{Resolution}$は，`RenderTexture(width, height, 0)`で設定した値になります．


# 透視投影変換
あるオブジェクトのカメラ座標を$[x_c, y_c, z_c]^T$とすると，画像上でのオブジェクトの位置$[u, v]^T$は，次のようになります．

$$ \begin{bmatrix} x\\ y\\ z \end{bmatrix} = \begin{bmatrix} f^{\prime}_x & 0 & c_x\\ 0 & f^{\prime}_y & c_y\\ 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} x_c\\ y_c\\ z_c \end{bmatrix} $$

$$ \begin{bmatrix} u\\ v \end{bmatrix} = \begin{bmatrix} \dfrac{x}{z}\\[2.0ex] \dfrac{y}{z} \end{bmatrix} $$

ただし，$[c_x,c_y]^T$は画像の中心座標です．


# 参考文献
[[Unity] VRのカメラトラッキングを切る](https://qiita.com/kleus_balut/items/952299f7b27edd423b38)
[UnityでxR有効時のカメラのFOVが固定されるのを防止する](https://qiita.com/kleus_balut/items/952299f7b27edd423b38)
[【Unity】Cameraに写るものをPNGファイルとして保存](https://www.kemomimi.dev/unity/1161/)