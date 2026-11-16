# Расширения и WidgetKit: вопросы на собеседовании

Таймлайны WidgetKit, Live Activities, App Intents, share- и notification-расширения, App Groups. Middle · 20 вопросов.

**[Тренировать тему на irbisa.com →](https://irbisa.com/ru/topic/ios-extensions-widgets/)** · [Все темы по iOS](../README.ru.md#все-темы)

### 1. Что такое app extension, какие виды встречаются чаще всего и чем расширение отличается от основного приложения по процессу, жизненному циклу и памяти?

App extension представляет собой отдельный таргет со своим bundle id и своим процессом, который система запускает, когда он нужен хосту, например домашнему экрану или share sheet. Он делит с приложением исходный код, но не его состояние в памяти.

[Полный ответ →](https://irbisa.com/ru/topic/ios-extensions-widgets/#q-67401)

### 2. Из каких частей состоит виджет WidgetKit и что на самом деле где выполняется?

Виджет состоит из `Widget` с конфигурацией, timeline provider, который выдаёт записи с датами, и SwiftUI-view, рисующей каждую запись. WidgetKit рендерит эти view заранее и показывает сохранённый результат, поэтому, пока виджет виден, в view ничего не выполняется.

[Полный ответ →](https://irbisa.com/ru/topic/ios-extensions-widgets/#q-67417)

### 3. Что делают `placeholder`, `snapshot` и `timeline` в timeline provider виджета и когда WidgetKit вызывает каждый из них?

`placeholder` мгновенно возвращает заглушку-скелет, которую рисуют серыми плашками; `snapshot` возвращает одну запись для галереи виджетов и других временных показов; `timeline` возвращает записи с датами, которые виджет реально покажет, и указывает, когда спросить снова.

[Полный ответ →](https://irbisa.com/ru/topic/ios-extensions-widgets/#q-67402)

### 4. Как сделать переход из виджета на конкретный экран приложения и чем `widgetURL` отличается от `Link`?

`.widgetURL(_:)` делает весь виджет одной зоной тапа с одним URL; `Link` даёт отдельные адреса частям виджета, но только в семействах medium, large и extra-large; у маленьких виджетов и аксессуаров на экране блокировки есть только `widgetURL`.

[Полный ответ →](https://irbisa.com/ru/topic/ios-extensions-widgets/#q-67407)

### 5. Что такое Live Activity и чем она отличается от виджета и от push-уведомления?

Live Activity представляет собой карточку одного идущего события (доставка, поездка, матч): приложение запускает её и обновляет на месте на экране блокировки и в Dynamic Island. В отличие от виджета она привязана к одному событию и обновляется сразу и явно; в отличие от пуша это одна меняющаяся карточка, а не россыпь сообщений.

[Полный ответ →](https://irbisa.com/ru/topic/ios-extensions-widgets/#q-67420)

### 6. Эта view виджета должна отсчитывать время каждую секунду. Почему этот отсчёт никогда не тикает на домашнем экране и как это исправить?

```swift
struct CountdownView: View {
  let entry: CountdownEntry
  @State private var secondsLeft = 0

  var body: some View {
    Text("\(secondsLeft) s left")
      .onAppear {
        secondsLeft = Int(entry.deadline.timeIntervalSinceNow)
      }
      .onReceive(Timer.publish(every: 1, on: .main, in: .common).autoconnect()) { _ in
        secondsLeft = Int(entry.deadline.timeIntervalSinceNow)
      }
  }
}
```

Отсчёт не пойдёт: WidgetKit рендерит body один раз и показывает заархивированный результат, поэтому ни `onAppear`, ни таймер не могут изменить `@State` на экране, и число никогда не меняется. Обратный отсчёт в виджете должен быть view, которую анимирует сама система, например `Text(timerInterval:countsDown:)`.

[Полный ответ →](https://irbisa.com/ru/topic/ios-extensions-widgets/#q-67418)

### 7. Как работают политики перезагрузки `.atEnd`, `.after(_:)` и `.never` и как приложение обновляет виджет сразу после изменения данных?

Политика перезагрузки сообщает WidgetKit самое раннее время, когда можно попросить у provider новый таймлайн; это просьба, а не расписание. Чтобы обновить виджет сразу, приложение записывает новые данные туда, откуда их читает виджет, и вызывает `WidgetCenter.shared.reloadTimelines(ofKind:)`.

[Полный ответ →](https://irbisa.com/ru/topic/ios-extensions-widgets/#q-67403)

### 8. Найдите баг: виджет шагов всегда показывает 0, хотя приложение показывает правильное число и перезагружает виджет после каждого сохранения.

```swift
// Compiled into both the app and the widget extension.
enum StepsStore {
  private static var defaults: UserDefaults { .standard }
  static func save(_ steps: Int) { defaults.set(steps, forKey: "todaySteps") }
  static func todaySteps() -> Int { defaults.integer(forKey: "todaySteps") }
}

// App: called when HealthKit delivers new samples.
func didUpdateSteps(_ steps: Int) {
  StepsStore.save(steps)
  WidgetCenter.shared.reloadTimelines(ofKind: "StepsWidget")
}

func makeEntry() -> StepsEntry {   // widget extension, in getTimeline
  StepsEntry(date: .now, steps: StepsStore.todaySteps())
}
```

`UserDefaults.standard` у каждого процесса свой, внутри его sandbox, поэтому приложение сохраняет в свой контейнер, а widget extension читает свой, пустой, и `integer(forKey:)` возвращает 0 для отсутствующего ключа. Обоим таргетам нужен suite App Group.

[Полный ответ →](https://irbisa.com/ru/topic/ios-extensions-widgets/#q-67421)

### 9. Этот provider должен показывать каждый из четырёх почасовых прогнозов, когда наступает его час. Что происходит вместо этого и как исправить?

```swift
func timeline(for configuration: CityIntent,
              in context: Context) async -> Timeline<WeatherEntry> {
  let forecast = await WeatherCache.hourly(for: configuration.city)

  let entries = forecast.prefix(4).map { hour in
    WeatherEntry(date: .now, temperature: hour.temperature)
  }
  return Timeline(entries: entries, policy: .atEnd)
}
```

У каждой записи дата `.now`, поэтому все четыре становятся актуальными в один и тот же момент, и виджет никогда не проходит по часам. А раз дата последней записи уже прошла, `.atEnd` сразу же просит новый таймлайн, снова и снова.

[Полный ответ →](https://irbisa.com/ru/topic/ios-extensions-widgets/#q-67419)

### 10. Как часто виджет может обновляться и что расходует его бюджет обновлений?

Каждый добавленный виджет получает скользящий бюджет (обычно 40–70 перезагрузок в сутки, примерно раз в 15–60 минут), и его размер зависит от того, как часто пользователь реально смотрит на виджет. Когда бюджет исчерпан, система молча перестаёт выполнять вашу политику перезагрузки, пока он не восстановится.

[Полный ответ →](https://irbisa.com/ru/topic/ios-extensions-widgets/#q-67404)

### 11. Как сделать настраиваемый пользователем виджет на App Intents и откуда берётся список вариантов для выбора?

Используйте `AppIntentConfiguration` с `WidgetConfigurationIntent`, у которого `@Parameter` имеет тип `AppEntity`; варианты выдаёт `EntityQuery` этой сущности, и выполняется он внутри widget extension, пока пользователь редактирует виджет.

[Полный ответ →](https://irbisa.com/ru/topic/ios-extensions-widgets/#q-67405)

### 12. Как работают интерактивные виджеты начиная с iOS 17 и почему `onTapGesture` в виджете ничего не делает?

Интерактивны только `Button(intent:)` и `Toggle(isOn:intent:)`: тап выполняет `AppIntent` в фоне, интент обновляет общие данные, а затем WidgetKit перезагружает виджет. Жесты и замыкания кнопок не выполняются никогда: view на экране остаётся заархивированным снимком.

[Полный ответ →](https://irbisa.com/ru/topic/ios-extensions-widgets/#q-67406)

### 13. Что такое режимы рендеринга виджета и почему цвета виджета пропадают на экране блокировки или на тонированном домашнем экране?

Виджет рендерится в режиме `.fullColor`, `.accented` или `.vibrant`, и два последних отбрасывают ваши цвета, поэтому всё, что передаёт смысл только оттенком, исчезает. Читайте режим из environment, дополняйте цвет иконкой или словом и объявляйте фон через `containerBackground`.

[Полный ответ →](https://irbisa.com/ru/topic/ios-extensions-widgets/#q-67411)

### 14. Что в ActivityKit относится к `ActivityAttributes`, а что к его `ContentState`, и как запустить, обновить и завершить Live Activity?

В `ActivityAttributes` лежит то, что не меняется за всю жизнь activity, например id заказа и ресторан; во вложенном `ContentState` лежит то, что меняется: этап и ETA. Запускают через `Activity.request`, состояние заменяют через `update`, завершают через `end`.

[Полный ответ →](https://irbisa.com/ru/topic/ios-extensions-widgets/#q-67408)

### 15. Какие представления есть у Live Activity в Dynamic Island и когда система использует каждое из них?

Compact (левая и правая view) показывается, когда ваша activity единственная запущенная, minimal показывается, когда activity есть у двух и более приложений, expanded открывается по долгому нажатию; плюс view для экрана блокировки, которая используется на устройствах без Island и для баннеров при обновлении.

[Полный ответ →](https://irbisa.com/ru/topic/ios-extensions-widgets/#q-67410)

### 16. Что такое App Intent и что нужно добавить, чтобы действие работало из Siri и Spotlight без настройки шортката пользователем?

`AppIntent` представляет собой Swift-тип, описывающий одно действие (заголовок, `@Parameter` и `perform()`), которое система может выполнить без вашего UI. Сам по себе он виден только в приложении Shortcuts; если добавить его в `AppShortcutsProvider` с голосовыми фразами, он сразу после установки доступен в Siri и Spotlight.

[Полный ответ →](https://irbisa.com/ru/topic/ios-extensions-widgets/#q-67413)

### 17. Найдите баг: текстовые push-уведомления, которые обрабатывает этот notification service extension, приходят примерно на 30 секунд позже, а push с картинкой приходит сразу.

```swift
final class NotificationService: UNNotificationServiceExtension {
  var handler: ((UNNotificationContent) -> Void)?
  var best: UNMutableNotificationContent?
  override func didReceive(_ request: UNNotificationRequest,
                           withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void) {
    handler = contentHandler
    best = request.content.mutableCopy() as? UNMutableNotificationContent
    guard let best, let link = best.userInfo["image"] as? String,
          let url = URL(string: link) else { return }
    Task {
      if let attachment = try? await ImageAttachment.download(url) { best.attachments = [attachment] }
      contentHandler(best)
    }
  }
  override func serviceExtensionTimeWillExpire() { if let handler, let best { handler(best) } }
}
```

Когда в payload нет URL `image`, `guard` выходит, не вызвав `contentHandler`, поэтому система ждёт, пока истечёт лимит времени расширения (около 30 секунд), вызывает `serviceExtensionTimeWillExpire()` и только тогда показывает push.

[Полный ответ →](https://irbisa.com/ru/topic/ios-extensions-widgets/#q-67422)

### 18. Как share-расширение получает файл, которым поделились, и как надёжно загрузить большой файл на сервер?

Прочитайте вложения `NSItemProvider` из `extensionContext.inputItems`, скопируйте файл в контейнер App Group и передайте загрузку фоновой `URLSession`. Процесс расширения завершается сразу после `completeRequest` и уносит с собой любую загрузку, идущую внутри процесса.

[Полный ответ →](https://irbisa.com/ru/topic/ios-extensions-widgets/#q-67412)

### 19. Как запускать и обновлять Live Activity с сервера через push-уведомления и какие токены нужны бэкенду?

Бэкенду нужен push-to-start токен, который принадлежит типу activity и существует до запуска любой activity, и update-токен для каждой работающей activity. Оба приходят как async-последовательности и могут меняться, поэтому приложение должно отправлять на сервер каждое новое значение, а не только первое.

[Полный ответ →](https://irbisa.com/ru/topic/ios-extensions-widgets/#q-67409)

### 20. На устройстве виджет пустой, хотя приложение работает нормально, а вывод `print` нигде не появляется. Как отлаживать widget extension?

Отлаживайте его как отдельный процесс, которым он и является: запустите схему widget extension или подключите Xcode к расширению по имени, логируйте через `Logger` вместо `print` и читайте краш-репорты самого расширения; пустой виджет чаще всего означает убитое расширение.

[Полный ответ →](https://irbisa.com/ru/topic/ios-extensions-widgets/#q-67415)
