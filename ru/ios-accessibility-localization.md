# Доступность и локализация: вопросы на собеседовании

VoiceOver, traits, Dynamic Type, Reduce Motion, String Catalog, плюрализация, RTL, форматы локали. Middle · 25 вопросов.

**[Тренировать тему на irbisa.com →](https://irbisa.com/ru/topic/ios-accessibility-localization/)** · [Все темы по iOS](../README.ru.md#все-темы)

### 1. Что входит в доступность iOS-приложения: для каких пользователей вы его делаете и какие системные функции поддерживаете?

Это значит, что приложением может пользоваться человек, который не видит экран, нуждается в крупном тексте, не различает цвета, плохо переносит анимацию или не может точно попадать пальцем. На практике это означает VoiceOver, Dynamic Type, контраст, Reduce Motion и альтернативный ввод.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67517)

### 2. Что такое accessibility label, value, traits и hint у элемента и в каком порядке VoiceOver их читает?

VoiceOver читает label, value, traits, hint: label говорит, что это за элемент, value передаёт его текущее состояние, traits описывают, какого он типа и как себя ведёт, а hint сообщает, что произойдёт при активации.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67501)

### 3. Что произносит VoiceOver, проходя по этой строке свайпами, и что бы вы изменили?

```swift
struct UserRow: View {
  let user: User
  let onDelete: () -> Void

  var body: some View {
    HStack {
      Image("avatar_placeholder")
      Text(user.name)
      Spacer()
      Button(action: onDelete) {
        Image("ic_trash")
      }
    }
  }
}
```

Три остановки, две из них создают шум: имя ассета заглушки как картинка, имя пользователя и имя ассета корзины как кнопка. Если ничего лучше нет, SwiftUI использует имя ассета картинки как её label.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67518)

### 4. Что такое Dynamic Type и как поддержать его в SwiftUI и UIKit?

Dynamic Type представляет собой системную настройку размера текста; текст, заданный текстовым стилем вроде `.body` или `.headline`, масштабируется вместе с ней автоматически: через семь обычных размеров и пять увеличенных размеров для доступности.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67519)

### 5. Как локализовать iOS-приложение и что нельзя хардкодить?

Добавьте языки в проект, держите каждую строку для пользователя в String Catalog (`Localizable.xcstrings`), который Xcode наполняет из кода при каждой сборке, и позвольте локали пользователя форматировать числа, даты и цены.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67520)

### 6. Карточка товара состоит из пяти остановок VoiceOver: картинка, название, цена, бейдж, кнопка-сердечко. Как сделать одну, и когда этого делать не стоит?

Оберните карточку в `.accessibilityElement(children: .combine)`, чтобы она стала одним элементом с объединённым label, а сердечко отдайте как именованное кастомное действие; в UIKit поставьте контейнеру `isAccessibilityElement = true` и напишите ему label.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67502)

### 7. VoiceOver читает цену карточки раньше названия, хотя название нарисовано выше. Как управлять порядком чтения?

В SwiftUI сгруппируйте вью через `.accessibilityElement(children: .contain)` и упорядочьте их через `.accessibilitySortPriority` (чем больше, тем раньше читается); в UIKit задайте контейнеру массив `accessibilityElements` в нужном порядке.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67503)

### 8. В строке есть swipe-to-delete и кастомный регулятор громкости. Как пользователь VoiceOver доберётся до них без этих жестов?

У каждого жеста должен быть аналог в доступности: кастомные действия (`.accessibilityAction(named:)`, `accessibilityCustomActions`) появляются на роторе Actions в VoiceOver, а контрол со значением становится adjustable, чтобы свайпы вверх и вниз его меняли.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67504)

### 9. После сохранения VoiceOver прыгает в начало экрана и читает его заново. Какое уведомление отправили по ошибке и какое нужно было?

Он отправил `.screenChanged`: это сообщает VoiceOver, что появился новый экран, и переносит фокус на его первый элемент; статусное сообщение должно быть `.announcement`, которое произносится без перемещения фокуса.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67505)

### 10. Форма не прошла валидацию. Как в SwiftUI перевести фокус VoiceOver на сообщение об ошибке?

Повесьте `.accessibilityFocused(_:)` на вью с ошибкой, привяжите его к свойству `@AccessibilityFocusState` и выставьте это свойство, когда ошибка уже на экране; VoiceOver переведёт курсор туда и прочитает её.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67521)

### 11. QA сообщает, что эта строка игнорирует размер текста пользователя, а когда начинает масштабироваться, обрезается на самых крупных размерах. Что вы измените?

```swift
HStack(spacing: 12) {
  Image(systemName: "envelope")
    .frame(width: 40, height: 40)
  Text(item.title)
    .font(.system(size: 17))
    .lineLimit(1)
}
.frame(height: 44)
```

Используйте текстовый стиль, чтобы текст масштабировался, и уберите вокруг него все фиксированные размеры: `minHeight` вместо `height`, размер иконки через `@ScaledMetric`, без ограничения в одну строку и с вертикальной раскладкой на размерах для доступности.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67506)

### 12. Найдите баг: пользователь увеличивает размер текста через Пункт управления, и подписи в этой шапке растут, но остаются в ряд и обрезаются, пока экран не открыть заново.

```swift
final class ContactHeaderViewController: UIViewController {
  private let stack = UIStackView(arrangedSubviews: [UILabel(), UILabel()])
  override func viewDidLoad() {
    super.viewDidLoad()
    for case let label as UILabel in stack.arrangedSubviews {
      label.font = .preferredFont(forTextStyle: .body)
      label.adjustsFontForContentSizeCategory = true
    }
    view.addSubview(stack)
    stack.frame = view.bounds
  }
  override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    stack.axis = traitCollection.preferredContentSizeCategory.isAccessibilityCategory
      ? .vertical : .horizontal
  }
}
```

Ось выбирается только во `viewWillAppear`, который не вызывается, когда размер текста меняется при открытом экране: подписи следуют за Dynamic Type сразу, а стек остаётся горизонтальным, и крупный текст обрезается.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67524)

### 13. Что такое Reduce Motion, Reduce Transparency, Increase Contrast и Bold Text и как учитывать их в своём UI?

Это системные настройки доступности, которые UIKit и SwiftUI учитывают в своих контролах; для собственных анимаций, размытий, цветов и шрифтов вы читаете их (как значения окружения SwiftUI или флаги `UIAccessibility`) и подстраиваетесь.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67507)

### 14. Дизайн помечает оплаченные заказы зелёной точкой, а просроченные красной. Что с этим не так и как это исправить?

Смысл передаётся только цветом, поэтому статус теряется для людей с нарушением цветовосприятия, а для VoiceOver теряется полностью; добавьте второй сигнал (форму или слово) и вынесите статус в accessibility label или value.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67509)

### 15. Как тестировать доступность приложения: что находят Accessibility Inspector и `performAccessibilityAudit()`, а что они пропускают?

Автоматические аудиты ловят механические проблемы одного экрана (отсутствующие labels, маленькие зоны нажатия, низкий контраст, обрезанный текст), а понять, есть ли в labels смысл и можно ли довести задачу до конца, может только человек с VoiceOver.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67508)

### 16. Что такое String Catalog и что значат состояния New, Needs Review и Stale после сборки?

String Catalog (`.xcstrings`) представляет собой JSON-файл, который Xcode синхронизирует с кодом: каждая сборка извлекает в него локализуемые литералы и отслеживает состояние каждого перевода, а в бандл он компилируется в обычные файлы `.strings` и `.stringsdict`.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67510)

### 17. Строка задана как `"\(count) items"`, а в русской сборке выходит «1 товаров». Как правильно работать с множественным числом?

Через вариацию по числу (plural variation) в String Catalog: система выбирает для числа категорию множественного числа по CLDR, а каждый язык даёт свои формы, так что ваш код никогда не выбирает между единственным и множественным.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67511)

### 18. В немецкой сборке заголовок навигации переведён, а заголовки секций остались английскими. Почему и какой тип это исправляет?

```swift
struct LibrarySection: Identifiable {
  let id = UUID()
  let title: String
}

let sections = [LibrarySection(title: "Recent"),
                LibrarySection(title: "Favourites")]

List(sections) { section in
  Text(section.title)
}
.navigationTitle("Library")
```

`Text` локализует только литералы: `Text(section.title)` со `String` использует дословный инициализатор, а «Recent», сохранённое как `String`, так и не было извлечено. Храните заголовок как `LocalizedStringResource`, который извлекается и резолвится в момент отрисовки.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67512)

### 19. Как форматировать даты, числа и цены для пользователя и кто выбирает формат, если язык английский, а регион немецкий?

Используйте API `.formatted()` и дайте решать `Locale.current`. Форматы чисел, дат, единиц и календаря задаёт регион, а не язык, поэтому английский с немецким регионом показывает английские слова с числами в немецком стиле (`1.234,50`) и 24-часовыми часами.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67513)

### 20. Найдите баг: во французской и немецкой сборках пикера стран несколько стран оказываются в самом конце списка.

```swift
struct CountryPicker: View {
  let regionCodes: [String]            // "AE", "AT", "DE", "ZW", …
  @Binding var selection: String

  private var countries: [(code: String, name: String)] {
    regionCodes
      .map { ($0, Locale.current.localizedString(forRegionCode: $0) ?? $0) }
      .sorted { $0.name < $1.name }
  }

  var body: some View {
    Picker("Country", selection: $selection) {
      ForEach(countries, id: \.code) { Text(verbatim: $0.name).tag($0.code) }
    }
  }
}
```

`<` у `String` сравнивает кодовые точки Unicode, а не алфавит пользователя, поэтому названия на букву с диакритикой (Égypte, Österreich) уходят после Z, а строчные идут после заглавных. Списку, который читает человек, нужно сравнение с учётом локали, например `localizedStandardCompare(_:)`.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67525)

### 21. Продукт хочет переключатель языка внутри приложения: что уже даёт iOS и почему запись `AppleLanguages` в `UserDefaults` считается хаком?

С iOS 13 в системе есть язык для отдельного приложения: если приложение поставляется с несколькими локализациями, на его странице в Настройках появляется пункт «Язык», а «переключатель» в приложении должен просто открывать эту страницу через `UIApplication.openSettingsURLString`.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67522)

### 22. Как поддержать языки с письмом справа налево, например арабский: что зеркалится автоматически, что переворачивать самому и что нельзя зеркалить никогда?

При вёрстке через leading и trailing UIKit и SwiftUI зеркалят интерфейс автоматически; вы решаете для каждой картинки, направленная ли она, оставляете незеркальными медиаконтролы и логотипы, а порядок символов в тексте доверяете алгоритму двунаправленного текста Unicode (bidi).

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67514)

### 23. Столбчатый график нарисован в одном `Canvas` или в переопределённом `draw(_:)`. Как сделать каждый столбец доступным для VoiceOver?

Нарисованный график остаётся одним непрозрачным элементом, поэтому дерево доступности для него строится вручную: `.accessibilityChildren` в SwiftUI или `UIAccessibilityElement` с фреймами в UIKit, плюс сводка и, в идеале, дескриптор графика для Audio Graphs.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67523)

### 24. Переводы приходят за две недели до релиза. Как поймать обрезанный текст и захардкоженные строки в CI задолго до этого?

Гоняйте UI-тесты на псевдоязыках Xcode и на самом крупном размере текста: они растягивают, снабжают диакритикой и берут в скобки каждую локализованную строку, так что обрезка, склеенные предложения и неизвлечённые литералы видны без всякого переводчика.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67515)

### 25. Помимо UI приложения, где локализуются страница в App Store, push-уведомления и App Shortcuts?

Всё это живёт вне кода вью: описание в магазине лежит в App Store Connect, текст пуша приходит как `loc-key`, который резолвится на устройстве или в Notification Service Extension, а фразы App Shortcuts хранятся в отдельном `AppShortcuts.xcstrings`.

[Полный ответ →](https://irbisa.com/ru/topic/ios-accessibility-localization/#q-67516)
