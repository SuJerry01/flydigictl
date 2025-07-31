## 遇到問題 「does not have endpoint with address 0x05」的

```
執行：flydigictl info

Error: connect to gamepad: open xinput device: open interface: failed to claim interface 0 on vid=045e,pid=028e,bus=1,addr=60,config=1: libusb: device or resource busy [code -6]
```

### Xbox 360 控制器「端點不符」的問題，修改了 pkg/flydigi/protocol/xinput/xinput.go 找不到 0x05 這個端點時，會自動 fallback 去找 0x02 這個端點（控制器實際提供的 OUT 端點）。

## 原本的程式碼：

```
outep, err := intf.OutEndpoint(5)
if err != nil {
    return nil, fmt.Errorf("open out endpoint: %w", err)
}
```

## 修改後的程式碼：

```
// Try endpoint 5 first (standard Xbox 360 controller), then fallback to endpoint 2
outep, err := intf.OutEndpoint(5)
if err != nil {
    // Fallback to endpoint 2 for some Xbox 360 controllers
    outep, err = intf.OutEndpoint(2)
    if err != nil {
        return nil, fmt.Errorf("open out endpoint: %w", err)
    }
}


```


## 這樣做的原因
1. 有些 Xbox 360 控制器的 OUT 端點是 0x02，不是 0x05。
2. 修改後，flydigictl 會自動支援這兩種端點配置，不會因為端點號不同就報錯。

## 小結

1. 確認這段程式碼已經在你的 pkg/flydigi/protocol/xinput/xinput.go 裡。
2. 重新編譯並安裝。
3. 重啟 flydigid 服務。

```
   sudo systemctl restart flydigid

   flydigictl info
```
