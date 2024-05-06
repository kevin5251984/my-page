---
layout: post
title:  "GCP Cloud Storage上傳相同檔名時，強制取代"
date:   2024-05-06 11:42:00 +0900
categories: chatgpt
---

## 網址
[官方文件](https://cloud.google.com/storage/docs/uploading-objects?hl=zh-cn#storage-upload-object-nodejs)

``` javascript
const options = {
    destination: destFileName,
    // Optional:
    // Set a generation-match precondition to avoid potential race conditions
    // and data corruptions. The request to upload is aborted if the object's
    // generation number does not match your precondition. For a destination
    // object that does not yet exist, set the ifGenerationMatch precondition to 0
    // If the destination object already exists in your bucket, set instead a
    // generation-match precondition using its generation number.
    // 可選：
    // 如果目标對象的世代号與您的前提條件不匹配，上傳請求将被終止。如果目标對象已存在于您的數據桶中，則使用其世代号設置世代匹配前提條件。
    preconditionOpts: {ifGenerationMatch: generationMatchPrecondition},
  };

```

## 問題

如果你上傳了相同檔名的文件時，系統會報錯，因為你檔案的版本號沒有調整

``` json
{"code":412,"message":"At least one of the pre-conditions you specified did not hold.","errors":[{"message":"At least one of the pre-conditions you specified did not hold.","domain":"global","reason":"conditionNotMet","locationType":"header","location":"If-Match"}]}
```

## 解決辦法

``` json

preconditionOpts: {
    "ifMatch": 0, //相同檔名則取代
}

```

