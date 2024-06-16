
有沒有曾經為 AMXX 的 set_rendering 而煩惱?

因為太多的插件同時在使用 set_rendering, 變得很難管理?

我針對以上這個問題寫了一個 set_rendering 的管理系統

## set_rendering 有什麼情況下會發生衝突?
**舉個例子:**

喪屍模式有凍結彈, 當你被凍結被會使用 set_rendering 為玩家加上一層像冰面的 淺藍色光, 凍結解除後重設 rendering

你的喪屍有血性暴走, 使用時會為自身加上一層紅色光, 暴走結束後重設 rendering

當以上兩個情況一起發生時, 就會出現衝突

譬如 你正在暴走時 被凍結住, 然後在凍結完之前暴走就使用結束, 那就會出現 凍結的藍色光提前消失的情況

另外一種情況就是一樣正在暴走時被凍結住, 在你暴走未完結之前 凍結就解除了, 暴走的發光效果會提前消失, 正確的狀況應該會是凍結解除後還是 紅色光, 等到暴走真正結束後才消失

(當然這只是舉個例, 因為可能你會寫被凍結時就取消暴走的狀態)


再來就是你的特殊感染者設定了身體發出基礎綠色光, 如果你再碰到上面兩種情況就會很麻煩

## 使用說明:
```sourcepawn
// 加入新的 rendering 層級
// 前 5 個參數跟 set_rendering 差不多
// duration 是 這個發光的持續時間, 設 0.0 代表永久 直到下一次重生
// class 是 給這個層級一個名稱, 若相同的名稱已經存在 則改變該層級的 rendering 樣式
// return 插入的 index
native render_push(id, fx, Float:color[3], mode, Float:amount, Float:duration=0.0, const class[]="");

// 移除指定一個 rendering 層級
// id 是玩家
// pop_index 是 要移除的 index
// class 是 從玩家現在的 rendering 列表中尋找這個名稱的層級再移除(找不到則不動作), 若class為空則使用pop_index
native render_pop(id, pop_index, const class[]="");

// 移除所有層級
native render_clear(id);
```

### 注意
- 這個 API 會自動在玩家重生時 移除所有層級
- 層級會根據 render_push 的先後決定層級的優先度
- 預設最多只能有 16 個層級, 如果想增加請修改 sma 裡的 MAX_LAYERS

### 以下是針對以上說的情況而寫的範例
首先, 設定特殊感染者的基礎綠色光

在玩家成為特殊感染者時 加入以下代碼:

```sourcepawn
render_push(id, kRenderFxGlowShell, Float:{0.0, 255.0, 0.0}, kRenderNormal, 16.0, 0.0, "base");
```

在冰彈凍結玩家時 加入以下代碼:

以下假設凍結時間是 3 秒

```sourcepawn
render_push(id, kRenderFxGlowShell, Float:{0.0, 200.0, 200.0}, kRenderNormal, 16.0, 3.0, "freeze");
```

在玩家使用暴走時 加入以下代碼:

以下假設暴走時間是 10 秒

```sourcepawn
render_push(id, kRenderFxGlowShell, Float:{255.0, 0.0, 0.0}, kRenderNormal, 16.0, 10.0, "sprint");
```

在一些情況中, 你可能想提前或手動取消暴走效果, 可以使用以下代碼:

```sourcepawn
render_pop(id, -1, "sprint");
```

## TODO (將會加入的功能):
- 讓 render_push 可以指定層級的優先度

---

post link: [https://bbs.mychat.to/reads.php?tid=1081146](https://bbs.mychat.to/reads.php?tid=1081146)
