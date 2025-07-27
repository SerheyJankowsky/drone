# 🔧 Исправление ошибки PSRAM "Insufficient PSRAM: 0 bytes"

## 🚨 Проблема

Если вы видите ошибку:

```
[E][ov2640.cpp:432] checkMemoryConstraints(): [OV2640] Insufficient PSRAM: 0 bytes
```

Это означает, что ESP32-S3 не может получить доступ к PSRAM памяти, которая критически важна для работы камеры.

## ✅ Решение

### 1. Проверьте правильность платы в `platformio.ini`

Убедитесь, что используете плату **с PSRAM**:

```ini
[env:esp32s3_fpv_drone]
platform = espressif32
board = freenove_esp32_s3_wroom     ; ✅ 8MB PSRAM
# НЕ ИСПОЛЬЗУЙТЕ:
# board = esp32-s3-devkitc-1        ; ❌ No PSRAM
```

### 2. Добавьте флаги для поддержки PSRAM

В `platformio.ini` должны быть флаги:

```ini
build_flags =
    -DBOARD_HAS_PSRAM                   ; Плата имеет PSRAM
    -DCONFIG_ESP32S3_SPIRAM_SUPPORT=1   ; Поддержка SPIRAM
    -DCONFIG_SPIRAM_MODE_OCT=1          ; Октальный режим SPIRAM
    -DCONFIG_SPIRAM_USE_MALLOC=1        ; Используть PSRAM для malloc
    -DCONFIG_SPIRAM_USE_CAPS_ALLOC=1    ; CAPS allocator для PSRAM
    -DPSRAM_MODE=1                      ; Включить PSRAM
```

### 3. Инициализируйте PSRAM в коде

В `src/main.cpp` добавьте проверку PSRAM:

```cpp
void setup() {
    Serial.begin(115200);
    delay(2000);

    // Инициализация PSRAM - критично для работы камеры
    if (!psramInit()) {
        Serial.println("[ERROR] ❌ PSRAM initialization failed!");
        while(1) delay(1000); // Останавливаем выполнение
    }

    Serial.printf("[PSRAM] ✅ PSRAM initialized: %lu bytes available\n", ESP.getFreePsram());

    // ... остальная инициализация
}
```

## 🎯 Рекомендуемые платы для проекта

### ✅ Подходящие платы с PSRAM:

- `freenove_esp32_s3_wroom` - 8MB Flash + 8MB PSRAM (рекомендуется)
- `adafruit_feather_esp32s3` - 4MB Flash + 2MB PSRAM
- `rymcu-esp32-s3-devkitc-1` - 8MB Flash + 2MB PSRAM

### ❌ НЕ подходящие платы без PSRAM:

- `esp32-s3-devkitc-1` - No PSRAM
- `adafruit_feather_esp32s3_nopsram` - No PSRAM
- `adafruit_qtpy_esp32s3_nopsram` - No PSRAM

## 🔍 Проверка PSRAM после загрузки

После загрузки прошивки, в Serial мониторе должно появиться:

```
[PSRAM] ✅ PSRAM initialized: 8388608 bytes available
[HEAP] 💾 Free heap: 320000 bytes
```

Если PSRAM = 0 bytes, значит плата не поддерживает PSRAM или неправильно настроена.

## 🛠️ Пошаговое исправление

1. **Измените плату в platformio.ini:**

   ```ini
   board = freenove_esp32_s3_wroom
   ```

2. **Очистите старую сборку:**

   ```bash
   pio run --target clean
   ```

3. **Соберите заново:**

   ```bash
   pio run
   ```

4. **Загрузите и проверьте:**
   ```bash
   pio run --target upload --target monitor
   ```

## 💡 Дополнительные советы

- **PSRAM критична для камеры** - без неё система не запустится
- **Минимум 1MB PSRAM** требуется для стабильной работы
- **8MB PSRAM** оптимально для высокого качества и FPS
- Если у вас нет платы с PSRAM, рассмотрите покупку ESP32-S3 CAM модуля

---

После применения этих изменений ошибка "Insufficient PSRAM" исчезнет и камера будет работать корректно! 🎯
