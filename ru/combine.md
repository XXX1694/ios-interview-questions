# Combine: вопросы на собеседовании

Publisher, Subscriber, операторы, Subjects, мост с async/await. Senior · 24 вопроса.

**[Тренировать тему на irbisa.com →](https://irbisa.com/ru/topic/combine/)** · [Все темы по iOS](../README.ru.md#все-темы)

### 1. Что такое Combine и как в нём связаны `Publisher`, `Subscriber` и операторы?

Combine представляет собой реактивный фреймворк Apple (iOS 13+): `Publisher` выдаёт значения во времени и затем одно завершение, операторы преобразуют этот поток, а `Subscriber` в конце цепочки запрашивает и получает значения.

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-3601)

### 2. Чем `PassthroughSubject` отличается от `CurrentValueSubject`?

`PassthroughSubject` передаёт каждое значение только тем, кто подписан в этот момент, и ничего не хранит; `CurrentValueSubject` хранит последнее значение, отдаёт его через `.value` и сразу передаёт каждому новому подписчику.

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-3602)

### 3. Чем `debounce` отличается от `throttle`?

`debounce` ждёт, пока апстрим замолчит на заданный интервал, и затем выдаёт последнее значение; `throttle` выдаёт не больше одного значения за интервал, пока значения продолжают приходить.

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-63618)

### 4. Что такое `AnyCancellable` и как управлять временем жизни и памятью подписки?

`AnyCancellable` служит токеном, который возвращает подписка: он отменяет её при вызове `cancel()` или при деаллокации, поэтому пайплайн живёт ровно столько, сколько кто-то держит этот токен.

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-3606)

### 5. Почему эта view model никогда не печатает `deinit` и чем `assign(to:on:)` отличается от `assign(to:)`?

```swift
final class ProfileViewModel {
  @Published var title = ""
  private var bag = Set<AnyCancellable>()

  init(user: AnyPublisher<User, Never>) {
    user
      .map(\.name)
      .assign(to: \.title, on: self)
      .store(in: &bag)
  }

  deinit { print("deinit") }
}
```

`assign(to:on:)` сильно держит объект, в который пишет, а возвращённый cancellable хранится в том же объекте, поэтому `self → bag → subscription → self` образует цикл удержания, который живёт, пока апстрим не завершится. `assign(to: &$title)` вместо этого привязывает время жизни к свойству и цикла не создаёт.

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-63615)

### 6. Найдите баг: после нескольких переходов в пост и обратно каждое обновление ленты рендерится и логируется по несколько раз.

```swift
final class FeedViewController: UIViewController {
  private let viewModel = FeedViewModel()
  private var bag = Set<AnyCancellable>()

  override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    viewModel.$posts
      .receive(on: DispatchQueue.main)
      .sink { [weak self] posts in
        self?.render(posts)
        Analytics.log("feed_shown", count: posts.count)
      }
      .store(in: &bag)
  }
}
```

Подписка создаётся в `viewWillAppear`, который вызывается при каждом возврате на экран, и каждая новая добавляется в `bag`, а старые продолжают жить. После трёх возвратов работают три sink, и каждое обновление рендерится и логирует `feed_shown` трижды.

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-63620)

### 7. Что делает `eraseToAnyPublisher()` и почему API возвращают `AnyPublisher`?

`eraseToAnyPublisher()` оборачивает паблишер в `AnyPublisher<Output, Failure>` и скрывает вложенные типы операторов, так что API показывает только то, что он выдаёт и как падает, а не то, как устроена цепочка.

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-63616)

### 8. Чем отличаются `merge`, `combineLatest` и `zip`?

`merge` перемешивает значения одного типа из нескольких паблишеров; `combineLatest` при изменении любого входа выдаёт кортеж из последних значений каждого; `zip` склеивает значения по порядковому номеру и выдаёт, только когда у каждого входа есть новое значение.

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-3603)

### 9. Чем `flatMap` отличается от `switchToLatest` и что из них нужно полю поиска?

`flatMap` держит живыми все внутренние паблишеры и сливает их результаты, поэтому медленный старый запрос может затереть более новый; `map` с последующим `switchToLatest` отменяет предыдущий внутренний паблишер при новом значении, и именно это нужно полю поиска.

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-63601)

### 10. Что выведет этот код и как сохранить поток живым после ошибки?

```swift
let subject = PassthroughSubject<Int, Never>()
let cancellable = subject
  .tryMap { value -> Int in
    if value == 2 { throw URLError(.badURL) }
    return value * 10
  }
  .replaceError(with: -1)
  .sink { print($0) }

subject.send(1)
subject.send(2)
subject.send(3)
```

Он выведет `10` и `-1`: брошенная ошибка означает завершение, поэтому `replaceError` выдаёт последнее значение и поток заканчивается, а `3` не приходит. Чтобы пережить ошибку, ловите её внутри `flatMap`.

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-63606)

### 11. Что такое горячие и холодные паблишеры и какие паблишеры Combine к каким относятся?

Холодный паблишер запускает работу отдельно для каждого подписчика в момент подписки; горячий выдаёт значения по своему расписанию независимо от того, слушает ли кто-то, поэтому опоздавший подписчик видит только то, что пришло после подписки.

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-63617)

### 12. Найдите баг: после одного тоста «Сохранено» каждый новый экран, который открывает пользователь, снова показывает тот же тост.

```swift
@MainActor final class ToastCenter {
  static let shared = ToastCenter()
  let messages = CurrentValueSubject<String?, Never>(nil)
  func show(_ text: String) { messages.send(text) }
}

class BaseViewController: UIViewController {
  private var bag = Set<AnyCancellable>()
  override func viewDidLoad() {
    super.viewDidLoad()
    ToastCenter.shared.messages
      .compactMap { $0 }
      .sink { [weak self] in self?.presentToast($0) }
      .store(in: &bag)
  }
}
```

`CurrentValueSubject` хранит последнее значение и отдаёт его каждому новому подписчику, поэтому каждый экран, подписавшийся в `viewDidLoad`, сразу получает старое сообщение и снова показывает тост. Для одноразового события нужен `PassthroughSubject`, который ничего не хранит, и показывать его должен только один экран.

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-63621)

### 13. Что такое `Future` в Combine и почему его обычно оборачивают в `Deferred`?

`Future` является паблишером, который выдаёт ровно одно значение или одну ошибку через колбэк `promise`; он выполняет своё замыкание сразу при создании и кеширует результат, поэтому нужен `Deferred`, чтобы сделать его ленивым и повторяемым.

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-63607)

### 14. Чем `receive(on:)` отличается от `subscribe(on:)`?

`receive(on:)` задаёт планировщик, на котором значения и завершение доставляются всему, что ниже него; `subscribe(on:)` задаёт, где апстрим получает подписку, запросы и отмену, то есть где начинается работа, и не управляет доставкой (синхронный апстрим всё равно выдаёт значения на этой очереди).

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-63608)

### 15. Что выведет этот код и почему sink видит старое значение `items`?

```swift
final class Cart: ObservableObject {
  @Published var items: [String] = []
  private var bag = Set<AnyCancellable>()

  init() {
    $items
      .sink { [weak self] newItems in
        print(newItems.count, self?.items.count ?? -1)
      }
      .store(in: &bag)
  }
}

let cart = Cart()
cart.items = ["apple", "pear"]
```

Он выведет `0 0`, а затем `2 0`: `$items` выдаёт текущее значение при подписке, а `@Published` публикует из `willSet`, поэтому sink выполняется до записи свойства и `self.items` ещё содержит старый массив.

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-63609)

### 16. Какие паблишеры Combine дают Foundation и UIKit и как получить паблишер для `UIButton`?

Apple даёт короткий список (`NotificationCenter`, KVO, `Timer`, `URLSession` и ваши собственные `@Published`-свойства), а встроенного паблишера для `UIControl` нет, поэтому для кнопки вы сами оборачиваете `UIAction` в субъект.

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-63605)

### 17. Почему обратный отсчёт на `Timer.publish` замирает, пока пользователь скроллит, и как это исправить?

Таймер запланирован в `RunLoop.Mode.default`, а пока палец тащит scroll view, главный run loop работает в режиме tracking, и таймеры режима default не срабатывают; запланируйте его с `in: .common`.

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-63610)

### 18. Как Combine и async/await работают вместе и когда Combine всё ещё лучший инструмент?

Паблишер становится `AsyncSequence` через `.values`, а асинхронная работа становится паблишером через `Deferred { Future { ... } }` или субъект, который наполняется из `Task`; Combine по-прежнему лучше там, где нужно комбинировать несколько потоков во времени.

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-3605)

### 19. Пайплайн Combine выдаёт два значения и замолкает. Как найти место, где останавливаются события?

Пометьте каждую стадию через `print("name")` или `handleEvents` и прочитайте лог жизненного цикла: первая стадия, которая перестала логировать значения, указывает на место, где цепочка умерла, а её последнее событие говорит, как именно.

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-63614)

### 20. У одного дорогого паблишера три подписчика. Сколько раз выполнится работа и как её разделить?

Три раза, потому что каждая подписка заново запускает всю цепочку; `share()`, `multicast` или `makeConnectable()` заставляют подписчиков использовать одно выполнение апстрима.

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-63602)

### 21. Что такое backpressure в Combine и почему он не спасает, когда `sink` встречает быстрый источник?

Backpressure означает, что подписчик сообщает паблишеру через `Subscribers.Demand`, сколько значений готов принять; `sink` и `assign` сразу запрашивают `.unlimited`, так что протокол фактически выключен и каждое значение проталкивается насквозь.

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-63611)

### 22. Как написать собственный оператор Combine и когда нужны свои `Publisher` и `Subscription`?

Собственный оператор почти всегда оказывается расширением `Publisher`, которое комбинирует существующие операторы и возвращает `AnyPublisher` или `some Publisher`; свой `Publisher` с собственной `Subscription` оправдан только для нового источника, который сам должен соблюдать спрос и отмену.

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-63619)

### 23. Как протестировать пайплайн с `debounce`, не засыпая в тесте?

Внедряйте планировщик вместо жёстко прописанного `DispatchQueue.main` и передавайте в тесте планировщик с виртуальными часами, которые тест двигает вперёд вручную.

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-63603)

### 24. Что ломается, когда модуль, плотно использующий Combine, переходит на языковой режим Swift 6, и как это чинить?

Combine появился раньше `Sendable` и изоляции акторов, поэтому Swift 6 отвергает значения и замыкания, которые через него пересекают границы изоляции, и добавляет проверки во время выполнения, роняющие main-actor `sink`, который кормится из фоновой очереди.

[Полный ответ →](https://irbisa.com/ru/topic/combine/#q-63604)
