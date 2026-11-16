# Жизненный цикл, фон и push: вопросы на собеседовании

Холодный старт, scene phase, фоновые режимы, BGTaskScheduler, APNs, silent push, universal links. Middle · 24 вопроса.

**[Тренировать тему на irbisa.com →](https://irbisa.com/ru/topic/ios-lifecycle-background/)** · [Все темы по iOS](../README.ru.md#все-темы)

### 1. В каких состояниях может быть iOS-приложение (not running, inactive, active, background, suspended), и что переводит его из одного в другое?

Приложение бывает not running (не запущено), inactive (на экране, но без событий), active (на экране и получает события), background (не на экране, код ещё работает) или suspended (в памяти, ничего не выполняет). Между ними его переводят запуск, прерывания, уход из приложения и нехватка памяти.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67125)

### 2. Чем `AppDelegate` отличается от `SceneDelegate`, и зачем в iOS 13 жизненный цикл UI перенесли в сцены?

`AppDelegate` отвечает за процесс: запуск, push-токены, события фонового `URLSession`, выбор конфигурации сцены. Каждый `SceneDelegate` владеет одним окном и его жизненным циклом на переднем плане и в фоне. В iOS 13 их разделили, чтобы одно приложение могло показывать несколько окон, каждое в своём состоянии.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67126)

### 3. Как отслеживать жизненный цикл приложения в SwiftUI через `scenePhase` и когда всё ещё нужен `@UIApplicationDelegateAdaptor`?

Читайте `@Environment(\.scenePhase)` и реагируйте в `.onChange(of:)`: `.active`, `.inactive` и `.background` соответствуют состояниям сцены в UIKit, а `@UIApplicationDelegateAdaptor` возвращает `AppDelegate` для колбэков, для которых в SwiftUI нет модификатора.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67118)

### 4. Гарантированно ли вызывается `applicationWillTerminate` и где тогда сохранять состояние?

Нет. Он вызывается, только если приложение завершают, пока оно выполняется, а большинство умирает замороженными, без всякого колбэка. Сохраняйте в `sceneDidEnterBackground` (или на `scenePhase == .background`): это последний момент, когда ваш код гарантированно выполняется.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67102)

### 5. Система выгрузила приложение в фоне, и пользователь возвращается на корневой экран вместо наполовину заполненной формы. Как восстановить состояние UI в приложении на сценах?

Сохраняйте короткое описание того, где был пользователь, а не сами данные: UIKit просит у каждой сцены `NSUserActivity` в `stateRestorationActivity(for:)` и возвращает его как `session.stateRestorationActivity` при следующем подключении; SwiftUI делает то же для каждого окна через `@SceneStorage`.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67127)

### 6. Чем локальные уведомления отличаются от удалённых и как запросить у пользователя разрешение?

Локальное уведомление планирует само приложение на устройстве с помощью триггера; удалённое отправляет ваш сервер через APNs на device token. Для пользователя они выглядят одинаково и используют одно разрешение, которое запрашивается через `UNUserNotificationCenter.requestAuthorization(options:)`.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67119)

### 7. После выхода этого кода число сессий удвоилось. Почему и в каком колбэке должна начинаться сессия?

```swift
final class SceneDelegate: UIResponder, UIWindowSceneDelegate {
  var window: UIWindow?

  func sceneDidBecomeActive(_ scene: UIScene) {
    Analytics.log("session_start")
  }

  func sceneDidEnterBackground(_ scene: UIScene) {
    Analytics.log("session_end")
  }
}
```

`sceneDidBecomeActive` срабатывает заново после каждого прерывания, которое лишь делает приложение inactive (запроса Face ID, Control Center, системного алерта), поэтому логирует лишние старты без парного конца. Начинайте сессию в `sceneWillEnterForeground`, парном к `sceneDidEnterBackground`.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67101)

### 8. Что происходит между тапом пользователя по иконке и вызовом `application(_:didFinishLaunchingWithOptions:)`?

Система создаёт процесс, `dyld` загружает динамические библиотеки, отрабатывают рантайм Objective-C и статические инициализаторы, затем `main` вызывает `UIApplicationMain`, который создаёт `UIApplication` и ваш app delegate и только после этого вызывает `willFinishLaunching` и `didFinishLaunching`.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67120)

### 9. Как iOS ограничивает фоновую работу и какими легальными способами можно выполнить код после того, как пользователь ушёл из приложения?

После ухода пользователя приложение получает несколько секунд в фоне, а затем замораживается. Для всего, что дольше, нужен разрешённый механизм: фоновая задача, задача `BGTaskScheduler`, фоновая `URLSession`, push или фоновый режим, используемый строго по назначению.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67104)

### 10. Этот код должен дать загрузке завершиться после ухода пользователя из приложения. Что с ним не так?

```swift
func sceneDidEnterBackground(_ scene: UIScene) {
  let app = UIApplication.shared
  let id = app.beginBackgroundTask(withName: "upload")

  uploader.upload(draft) { result in
    if case .success = result {
      app.endBackgroundTask(id)
    }
  }
}
```

Нет expiration handler, а задача завершается только при успехе, поэтому медленная или упавшая загрузка оставляет идентификатор открытым; когда бюджет заканчивается, watchdog убивает приложение с кодом `0x8badf00d`, и пользователь видит это как краш при переключении на другое приложение.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67103)

### 11. Найдите баг: посреди тренировки пользователь на несколько минут переключается в другое приложение, а по возвращении этот таймер отстаёт на эти минуты.

```swift
struct WorkoutTimerView: View {
  @State private var elapsed = 0
  private let ticker = Timer.publish(every: 1, on: .main, in: .common).autoconnect()

  var body: some View {
    Text(Duration.seconds(elapsed).formatted(.time(pattern: .minuteSecond)))
      .monospacedDigit()
      .onReceive(ticker) { _ in
        elapsed += 1
      }
  }
}
```

Вью считает тики, а не измеряет время. В фоне приложение замораживается через несколько секунд и таймер перестаёт срабатывать, поэтому каждая минута вне приложения теряется, и тренировка оказывается короче ровно на это время.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67123)

### 12. Как запланировать работу через `BGTaskScheduler` и почему зарегистрированная задача может так и не запуститься?

Зарегистрируйте обработчик до возврата из `didFinishLaunching`, добавьте идентификатор в `Info.plist`, затем вызовите `submit` с `BGAppRefreshTaskRequest` или `BGProcessingTaskRequest`. Когда запускать, решает система, а тихие ошибки в настройке делают так, что задача не запускается никогда.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67105)

### 13. Как протестировать задачу `BGTaskScheduler`, не дожидаясь, пока её запустит система?

На устройстве уведите приложение в фон после того, как оно отправило запрос, поставьте паузу в Xcode и вызовите из LLDB приватный `_simulateLaunchForTaskWithIdentifier:`, и зарегистрированный обработчик выполнится сразу. `_simulateExpirationForTaskWithIdentifier:` так же вызывает expiration handler.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67106)

### 14. Как push-уведомления работают от начала до конца: APNs, device token и payload?

Приложение запрашивает у APNs device token и отправляет его на ваш сервер; сервер шлёт JSON-payload для этого токена в APNs по HTTP/2, аутентифицируясь ключом `.p8` или сертификатом, а APNs доставляет его, чтобы iOS показала уведомление или разбудила приложение.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67107)

### 15. Что нужно silent push, чтобы разбудить приложение, и почему он всё равно может не дойти?

В payload нужен `"content-available": 1` и ничего видимого, в запросе нужны `apns-push-type: background` и `apns-priority: 5`, а приложению нужен фоновый режим `remote-notification`. Но даже тогда система считает такой push подсказкой и может задержать или отбросить его.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67108)

### 16. Найдите баг в этом обработчике silent push: пуш приходит, но когда пользователь открывает приложение, входящие всё ещё устаревшие.

```swift
func application(_ application: UIApplication,
                 didReceiveRemoteNotification userInfo: [AnyHashable: Any],
                 fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {
  guard userInfo["reason"] as? String == "inbox" else {
    completionHandler(.noData)
    return
  }
  Task {
    try await inbox.sync()
  }
  completionHandler(.newData)
}
```

`completionHandler(.newData)` вызывается сразу после создания `Task`, до того как синхронизация что-то сделала. Этот вызов сообщает iOS, что работа закончена, поэтому приложение вскоре замораживается и запрос застывает на полпути; входящие обновятся только при следующем выходе на передний план.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67124)

### 17. Как показать или скрыть уведомление, пришедшее при открытом приложении, и как тапы и кнопки действий попадают в `userNotificationCenter(_:didReceive:)`?

Реализуйте `userNotificationCenter(_:willPresent:)`, чтобы решать, показывать ли уведомление на переднем плане, и зарегистрируйте при запуске `UNNotificationCategory` с действиями; ключ `category` в payload выбирает кнопки, а `userNotificationCenter(_:didReceive:)` обрабатывает тапы и действия.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67110)

### 18. Что даёт provisional-разрешение на уведомления и почему пользователь со статусом `.authorized` может ничего не видеть?

Если добавить `.provisional` в `requestAuthorization`, разрешение выдаётся без диалога, но уведомления тихо попадают в Центр уведомлений, без баннера и звука, пока пользователь не нажмёт «Оставить». А `.authorized` значит лишь, что приложению можно отправлять уведомления, а не что баннеры включены.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67111)

### 19. Чем universal links отличаются от кастомных URL-схем и что нужно universal link, чтобы открыть приложение?

Кастомная схема вроде `myapp://` ничем не подтверждена (её может заявить любое приложение, и она не срабатывает, если приложение не установлено), а universal link работает как обычный `https`-URL, который открывает приложение только после того, как iOS проверила домен, а иначе открывает сайт.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67112)

### 20. Universal links открывают нужный экран, пока приложение запущено, но после холодного старта приводят на главный экран. Чего здесь не хватает?

```swift
final class SceneDelegate: UIResponder, UIWindowSceneDelegate {
  var window: UIWindow?

  func scene(_ scene: UIScene, willConnectTo session: UISceneSession,
             options connectionOptions: UIScene.ConnectionOptions) {
    guard let windowScene = scene as? UIWindowScene else { return }
    window = UIWindow(windowScene: windowScene)
    window?.rootViewController = RootViewController()
    window?.makeKeyAndVisible()
  }

  func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
    guard let url = userActivity.webpageURL else { return }
    router.open(Route(url) ?? .home)
  }
}
```

При холодном старте ссылка приходит не через `scene(_:continue:)`: она лежит в `connectionOptions.userActivities` внутри `scene(_:willConnectTo:options:)`, а этот код её игнорирует.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67122)

### 21. Как iOS может запустить приложение сразу в фоне и чего `didFinishLaunching` должен избегать на этом пути?

Silent push, задача `BGTaskScheduler`, завершившаяся фоновая `URLSession`, событие геолокации или Bluetooth могут запустить процесс без UI. Выполняется тот же `didFinishLaunching`, но сцена не подключается и интерфейс не показывается.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67115)

### 22. Какие фоновые пробуждения ещё доходят до приложения после того, как пользователь смахнул его в переключателе приложений?

Принудительное закрытие говорит iOS, что пользователь хочет остановить приложение: silent push, задачи `BGTaskScheduler` и передачи фоновой `URLSession` прекращаются, пока пользователь снова не откроет приложение, а alert-push, VoIP push и события геолокации по-прежнему доходят.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67116)

### 23. Что такое prewarming приложения и как он искажает метрики времени запуска?

Начиная с iOS 15 система может запустить процесс заранее, перед предсказанным запуском, выполнить `dyld` и статические инициализаторы и остановиться перед `UIApplicationMain`. Тогда таймер запуска, привязанный к старту процесса, считает минуты, которые пользователь не ждал.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67114)

### 24. Что такое `BGContinuedProcessingTask` в iOS 26 и когда его выбирать вместо `BGProcessingTask`?

Позволяет работе, которую пользователь явно запустил (экспорт, загрузка, большая правка), продолжаться после ухода из приложения, с системным индикатором прогресса и кнопкой отмены. В отличие от `BGProcessingTask`, стартует сразу, с переднего плана.

[Полный ответ →](https://irbisa.com/ru/topic/ios-lifecycle-background/#q-67121)
