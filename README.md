# VdLimit SDK

## 1. Требования для работы с VdLimit

Минимальная версия Android SDK - API 21.

Наличие в манифесте основного приложения разрешения на работу с камерой:

```xml
// Permission
<uses-permission android:name="android.permission.CAMERA" />
```

Наличие в манифесте основного приложения объявленной activity для работы с UI SDK:

```xml
// SDK Activity
<activity 
android:name="com.da.vdlimit.presentation.activity.VdLimitMainActivity" 
android:launchMode="singleTask" 
android:screenOrientation="portrait"
android:windowSoftInputMode="stateHidden|adjustResize" />
```

Подключенный Data Binding в build.gradle основного проекта

```groovy
android {
    dataBinding {
        enabled = true
    }
}
```

## 2. Подключение библиотеки

Содержимое архива [vdlimit.zip](http://vdlimit.zip) необходимо распаковать в директорию libs основного проекта.

В build.gradle основного проекта добавить локальный репозиторий:

```groovy
repositories {
    maven {
        url uri("${projectDir}/libs")
    }
}
```

После чего добавить зависимость:

```groovy
implementation ("com.da.vdlimit:vdlimit:1.0.0") {
        transitive = true
}
```

## 3. Инициализация SDK

В Application классе основного приложения создать экземпляр конфигурационного класса, в конструктор которого отдать API ключ, для работы SDK.

Опционально: 

- Передать в конфигурацию номер телефона (userPhoneNumber) текущего пользователя приложения и его идентификатор(userId), в этом случае эти данные не нужно будет указывать при запросе лимита без вызова UI SDK;
- Передать в конфигурацию стили текстов и акцентный цвет для кастомизации интерфейса UI SDK под стиль основного приложения. В стилях текстов, передаваемых в конфигурацию, так же может использоваться кастомный шрифт.

```kotlin
val sdkConfig = VdLimit.Config(
		apiKey = "someKey",
    apiSecret = "someKey",
    userPhoneNumber = "999999999999",
    userId = "hgd123-hgd123-hgd123"
)
VdLimit.init(sdkConfig)
```

## 3. Вызов экрана анкеты для получения кредитного лимита

Для вызова экрана заполнения анкеты и получения итогового результата после запроса лимита, необходимо создать интент VdLimit.getLimitsIntent( и использовать его в startActivity().

```kotlin
val intent = VdLimit.getLimitsIntent(
                context = this,
                userId = "hgd123-hgd123-hgd123", // опционально
                userPhone = "999999999999" // опционально
            )
startActivityForResult(intent)
```

## 4. Получение статуса кредитного лимита без вызова экрана анкеты

Для получения данных о статусе кредитного лимита без вызова экрана анкеты, необходимо вызвать метод VDLimit.getStatus() и реализовать интерфейс колбека для получения результата. В метод колбека onResult будет приходить модель результата VDLimitStatus. В метод колбека onError будет приходить строка с ошибкой.

При вызове VDLimit.getStatus() параметры userPhoneNumber и userId необходимо заполнять если они не были указаны при конфигурации SDK:

```kotlin

VdLimit.getStatus(
                userId = configData[3],
                userPhone = 999999999999,
                statusResultListener = object : VdLimit.StatusResultListener {
                    override fun onError(error: String) {
                        writeToLog("Check SDK limit error: $error")
                    }

                    override fun onResult(status: VDLimitStatus) {
                        writeToLog("Check SDK limit result: $status")
                    }
                }
            )
```