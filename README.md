# SynthCore для Android

Android-приложение (Kotlin, Jetpack Compose) поверх порта SynthCore. Все данные синтетические.

## Что внутри
- `:core` - чистый Kotlin/JVM-модуль, порт Python-проекта без смены архитектуры:
  `Model` -> `Contracts` (`SnapshotSource`, `ActionSink`) -> `Esp` / `Aim` -> `Synthetic` (синтетические адаптеры, `SimulatedWorld`).
  Исходный Python-проект лежит в `reference/python-synthcore`.
- `:app` - интерфейс: тёмная тема, боковая навигация (на экранах >= 840dp постоянная панель), разделы ESP, Aim, Радар, Настройки, О проекте.
  Настройки хранятся в SharedPreferences и восстанавливаются после перезапуска. На экранах >= 600dp контент идёт в две колонки.
- `app/src/main/AndroidManifest.xml` не содержит разрешений: нет оверлея, служб доступности, захвата экрана и сети.

## Free Fire
Интеграции нет. Официального API для ESP и автонаведения у игры нет, а чтение памяти, обход защиты и работа поверх игры
в проект не входят. `SnapshotSource` и `ActionSink` остаются интерфейсами, реализованными только синтетически.

## Как получить APK

### Вариант 1. GitHub Actions (ничего не нужно ставить)
1. Создайте репозиторий на GitHub и загрузите содержимое архива (папка `.github` должна попасть в репозиторий).
2. Откройте Actions -> Build APK -> Run workflow (или просто сделайте push в `main`).
3. После зелёной сборки скачайте артефакт `synthcore-debug-apk` (внутри `app-debug.apk`). Workflow заодно запускает все unit-тесты.

### Вариант 2. Android Studio
1. Установите Android Studio (Ladybug 2024.2 или новее) с Android SDK 35.
2. File -> Open -> выберите папку проекта, дождитесь Gradle Sync.
3. Build -> Build APK(s). Файл: `app/build/outputs/apk/debug/app-debug.apk`.

### Вариант 3. Командная строка
Нужны JDK 17, Android SDK (platforms;android-35, build-tools) и переменная `ANDROID_HOME`.

    gradle wrapper --gradle-version 8.9      # один раз, создаёт gradlew
    ./gradlew :core:test :app:testDebugUnitTest :app:assembleDebug

## Установка на телефон (Android 8.0+)
1. Скопируйте `app-debug.apk` на телефон.
2. Откройте файл и разрешите установку из этого источника (Настройки -> Установка неизвестных приложений).
3. Нажмите "Установить". Либо по USB с включённой отладкой: `adb install -r app-debug.apk`.
Debug-APK подписан отладочным ключом: годится для личной установки, но не для публикации в Google Play.

## Тесты
- `core/src/test` - 23 JVM-теста порта (проекция, фильтры, углы, обёртка yaw, режимы выбора цели, сходимость, контракты):
  `gradle :core:test`
- `app/src/test` - JVM-тест настроек: `gradle :app:testDebugUnitTest`
- `app/src/androidTest` - инструментальный тест сохранения настроек (нужен эмулятор или устройство):
  `gradle :app:connectedDebugAndroidTest`
