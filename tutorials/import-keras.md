# 如何將 Keras 模型導入 TensorFlow.js

Keras 模型（通常透過 Python API 建立）的多種格式可以壓縮成一檔案儲存，而「整個模型」的格式可以轉換成 TensorFlow.js Layers 格式，並且可以直接加載到 TensorFlow.js 中進行推測或進一步的訓練。

TensorFLow.js Layers 格式是一個目錄包含 `model.json` 檔案以及一組二進位制碎片式權重的檔案，而 `model.json` 文件包含拓樸（Topology）模型（又名「結構」、「圖形」：描述各圖層以及它們如何連接）及權重檔案的清單。

## 要求

轉換的過程需要 Python 的環境，而你可以使用 [pipenv](https://github.com/pypa/pipenv) 或 [virtualenv](https://virtualenv.pypa.io/en/stable/) 來保持一個獨立的環境。若要安裝轉換器，請使用`pip install tensorflow`。

將 Keras 模型導入 TensorFlow.js 是一個兩步驟的程序，首先，將現有的 Keras 模型轉換成 TensorFlow.js Layers 格式，然後再將其加載到 TensorFlow.js 中


## 步驟一：將現有的 Keras 模型轉換為 TF.js Layers 格式

Keras 模型通常通過 `model.save(filepath)` 生成一個包含拓樸模型和權重的單一 HDF5（.h5）檔案來儲存。若要將這樣的檔案轉換成 TF.js Layers 格式，執行以下指令，其中 `path/to/my_model.h5` 是 Keras.h5 檔案存取的路徑，而 `path/to/tfjs_target_dir` 是 TF.js 輸出所存放的目錄：
```
#bath 

tensorflowjs_converter --input_format keras \
                       path/to/my_model.h5 \
                       path/to/tfjs_target_dir
```
## 替代方案：利用 Python API 直接輸出成 TF.js Layers 格式
如果你在 Python 中使用 Keras 模型，可以利用以下指令直接輸出成 Tensorflow.js Layers 格式：
```Python

import tensorflowjs as tfjs

def train(...):
    model = keras.models.Sequential()   # for example
    ...
    model.compile(...)
    model.fit(...)
    tfjs.converters.save_keras_model(model, tfjs_target_dir)
```
## 步驟二：將模型加載到 Tensorflow.js 中
使用網頁伺服器來為步驟一中所轉換的模型檔案提供服務，請注意你可能需要設定網頁伺服器以允許跨域資源共享（CORS），允許使用 JavaScript 取得檔案。

透過提供 URL 給 model.json 檔案，將模型加載到 TensorFlow.js 當中：

``` JavaScript

import * as tf from '@tensorflow/tfjs';

const model = await tf.loadModel('https://foo.bar/tfjs_artifacts/model.json');
```

現在該模型已經準備進行推測、評估或者是重新訓練。例如：已載入的模型可以被立即用來進行預測：

```JavaScript

const example = tf.fromPixels(webcamElement);  // for example
const prediction = model.predict(example);
```

許多 [TensorFlow.js 範例](https://github.com/tensorflow/tfjs-examples) 都採用此方法，利用那些已經轉換完成且在 Google 雲端硬碟上的待訓練模型。

請注意你使用 `model.json` 文件名來引用整個模型，`loadModel(...)` 取得 `model.json`，然後發出額外的 HTTP(S) 來請求獲得 `model.json` 權重清單中所提及的權重檔案，這個方法可以透過瀏覽器來快取所有這些檔案（也可以透過其他網路上的快取伺服器），因為 `model.json` 檔案和權重碎片都小於每個典型的快取檔案大小限制，因此，模型可能會在隨後的時間加載的更快。

## 支援的功能

TensorFlow.js Layers 目前只支援使用標準 Keras 構造的 Keras 模型。使用不支援的運算子或圖層，例如：自定義圖層（custom layers）、Lambda 圖層、自定義損失量（custom losses）、自定義指標（custom metrics）將無法自動導入，因為它們取決於 Python 程式碼且無法確實地轉換成 JavaScript。
