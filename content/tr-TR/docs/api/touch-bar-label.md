## Sınıf: DokunmatikÇubukEtiketi

> Dokunmatik çubukta yerel macOS uygulamaları için bir etiket oluşturun

Süreç: [Ana](../tutorial/quick-start.md#main-process)

### `yeni DokunmatikÇubukEtiketi(seçenekler)` *Deneysel*

* `seçenekler` Nesne 
  * `label` Dize (opsiyonel) - Görüntülenecek metin.
  * `textColor` Dize (opsiyonel) - Metinin Hex rengi, örn `#ABCDEF`.

### Örnek Özellikler

Aşağıdaki özellikler `TouchBarLabel` örnekleri için uygundur:

#### `touchBarLabel.label`

Etiketin geçerli metnini temsil eden bir `String`. Bu değeri değiştirmek dokunmatik çubuktaki etiketi hemen güncelleştirir.

#### `touchBarLabel.textColor`

Etiketin geçerli rengini değiştiren bir `String` hex kodu. Bu değeri değiştirmek dokunmatik çubuktaki etiketi hemen güncelleştirir.