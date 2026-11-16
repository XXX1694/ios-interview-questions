# Состояние в SwiftUI: вопросы на собеседовании

@State, @Binding, @StateObject, @ObservedObject, @EnvironmentObject, Observation. Middle · 24 вопроса.

**[Тренировать тему на irbisa.com →](https://irbisa.com/ru/topic/swiftui-state/)** · [Все темы по iOS](../README.ru.md#все-темы)

### 1. Чем `@State` отличается от `@Binding` в SwiftUI?

`@State` объявляет хранилище, которым владеет сама вью: SwiftUI держит значение вне структуры вью и перерисовывает её, когда оно меняется. `@Binding` работает как ссылка на чтение и запись к состоянию, которым владеет кто-то другой: ребёнок может менять значение, но не хранит его.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-2901)

### 2. Что в этой вью означают `count`, `$count` и `_count`?

```swift
struct CounterView: View {
  @State private var count: Int

  init(start: Int = 0) {
    _count = State(initialValue: start)
  }

  var body: some View {
    VStack {
      Text("\(count)")
      Stepper("Count", value: $count)
      Button("Reset") { count = 0 }
    }
  }
}
```

`count` возвращает текущее значение (`wrappedValue` обёртки), `$count` возвращает `Binding<Int>` к нему (`projectedValue`), а `_count` ссылается на саму обёртку `State<Int>`, к которой обращаются только в `init`.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-62915)

### 3. Чем отличаются `@Environment`, `@EnvironmentObject` и `@ObservedObject`, и когда брать environment вместо `init`?

`@ObservedObject` получает объект явно через инициализатор, а `@EnvironmentObject` и `@Environment` читают то, что внедрено выше по дереву, поэтому промежуточным вью не нужно передавать это дальше (prop drilling).

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-2903)

### 4. Какие значения `@Environment` вы реально используете? Например, как дочерняя вью закрывает свой экран?

Она читает `@Environment(\.dismiss)` и вызывает его. Environment знает, показана ли вью как sheet или запушена в навигационный стек, поэтому ребёнку не нужен биндинг обратно к тому, кто его показал.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-62905)

### 5. Кнопка Done на клавиатуре формы ничего не делает, а Next не переводит на следующее поле. Какой property wrapper забыли?

Не хватает `@FocusState` (вместе с `.onSubmit`). `@FocusState` отражает фокус клавиатуры: поля привязывают к нему через `.focused(...)`, присваивание поля переводит туда фокус, а `nil` прячет клавиатуру.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-62907)

### 6. Родитель меняет `start` с 0 на 5, а `Counter` продолжает показывать 0. Почему и как исправить?

```swift
struct Counter: View {
  @State private var count: Int

  init(start: Int) {
    _count = State(initialValue: start)
  }

  var body: some View {
    Button("\(count)") { count += 1 }
  }
}

// In the parent:
// Counter(start: selectedPreset.start)
```

`State(initialValue:)` используется только при первом появлении вью с этой identity. Дальше SwiftUI хранит своё значение и игнорирует каждое новое начальное значение от родителя, поэтому `count` остаётся 0.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-62917)

### 7. Когда использовать `@StateObject`, а когда `@ObservedObject`, и что сломается, если выбрать не тот?

`@StateObject` значит, что вью владеет объектом: SwiftUI создаёт его один раз и держит живым всё время жизни вью. `@ObservedObject` лишь наблюдает за объектом, которым владеет кто-то другой, поэтому созданный прямо в нём объект пересоздаётся при каждой перерисовке родителя.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-2902)

### 8. Что делает `@Published` и как `ObservableObject` сообщает об изменениях?

`@Published` оборачивает свойство и отправляет событие в паблишер `objectWillChange` объекта непосредственно перед изменением значения; вью, наблюдающие объект (`@StateObject`, `@ObservedObject`, `@EnvironmentObject`), после этого заново выполняют `body`.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-62921)

### 9. Найдите баг: по нажатию товар добавляется в корзину, но счётчик корзины в тулбаре не меняется.

```swift
final class Cart: ObservableObject { @Published var items: [Product] = [] }
final class ShopModel: ObservableObject { @Published var cart = Cart() }

struct ShopScreen: View {
  let products: [Product]
  @StateObject private var model = ShopModel()
  var body: some View {
    List(products) { p in
      Button(p.name) { model.cart.items.append(p) }
    }
    .toolbar { Text("Cart: \(model.cart.items.count)") }
  }
}
```

`Cart` объявлен как класс, поэтому `@Published var cart` хранит ссылку, которая не меняется. Добавление в `cart.items` дёргает собственный `objectWillChange` у `Cart`, на который не подписана ни одна вью, поэтому `ShopModel` молчит, а в тулбаре остаётся старое число.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-62922)

### 10. Что меняет макрос `@Observable` по сравнению с `ObservableObject` и `@Published`?

`@Observable` делает наблюдаемым каждое хранимое свойство класса без `@Published`, а SwiftUI отслеживает, какие свойства прочитала каждая вью, поэтому вью перерисовывается только при изменении этих свойств, а не при любом изменении объекта.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-2904)

### 11. Что такое `@Bindable` и когда он нужен с моделью `@Observable`?

`@Bindable` превращает `@Observable`-объект в источник биндингов: `$profile.name` становится `Binding<String>` для `TextField`. Он нужен только во вью, которая не создавала модель, но должна отдать контролам двусторонние биндинги.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-62918)

### 12. Вы нажимаете кнопку, а текст не меняется. Почему и как исправить?

```swift
final class Draft {
  var title = ""
}

struct Editor: View {
  @State private var draft = Draft()

  var body: some View {
    VStack {
      Text(draft.title.isEmpty ? "empty" : draft.title)
      Button("Set") { draft.title = "Hello" }
    }
  }
}
```

`@State` перерисовывает вью, когда заменяется хранимое в нём значение, а `draft.title = "Hello"` меняет объект за ссылкой, не заменяя саму ссылку, поэтому SwiftUI не видит изменений.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-62908)

### 13. Как выбрать правильный property wrapper для куска состояния в SwiftUI?

Состоянием владеет вью, которая его создала: `@State` для значения или `@Observable`-модели. Остальные получают обычный `let` для чтения, `@Binding` (или `@Bindable` для модели) для записи либо environment для всего поддерева.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-62919)

### 14. Как передать дочерней вью `Binding`, если значение не хранится в `@State`?

`Binding` представляет собой просто геттер плюс сеттер, поэтому его можно собрать вручную через `Binding(get:set:)`, когда значение лежит там, куда не дотянуться через `$`: за преобразованием типа или внутри опционала.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-62902)

### 15. Правки на экране деталей не появляются в списке позади него, хотя оба показывают одну и ту же структуру `Order`. Что пошло не так?

Экран деталей получил копию. `Order` объявлен как структура, поэтому передача как `let order` или заполнение из неё локального `@State` даёт ребёнку собственное значение, и все правки попадают в эту копию.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-62910)

### 16. Когда SwiftUI заново выполняет `body` вью и когда сбрасывается её `@State`?

SwiftUI заново выполняет `body`, когда изменилось то, что вью прочитала: её состояние, биндинг, значение environment, наблюдаемое свойство или входные данные от родителя. `@State` сбрасывается при смене identity вью, потому что состояние принадлежит identity.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-2905)

### 17. Состояние «развёрнуто/свёрнуто» у строки сбрасывается, когда пользователь прокручивает её за экран и обратно. Что на самом деле происходит?

Флаг живёт в `@State` строки, а `List` или `LazyVStack` вправе уничтожать строки, ушедшие далеко за экран. SwiftUI не обещает сохранять их состояние, поэтому строка возвращается собранной заново со значением по умолчанию.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-62911)

### 18. Где во вью SwiftUI место для побочных эффектов и асинхронной работы и что происходит с `.task`, когда вью исчезает?

Только не в `body`: SwiftUI может выполнить его когда угодно и сколько угодно раз. Побочные эффекты кладут в замыкания действий и событийные модификаторы (`.task`, `.onChange`, `.onAppear`), а `.task` автоматически отменяется, когда его вью исчезает.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-2906)

### 19. Что делают `@AppStorage` и `@SceneStorage` и где они перестают быть подходящим инструментом?

`@AppStorage` объявляет свойство поверх `UserDefaults`, которое перерисовывает вью при изменении ключа; `@SceneStorage` хранит UI-состояние отдельного окна, которое система сохраняет и восстанавливает при перезапуске приложения.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-62904)

### 20. Как `@Observable` узнаёт, какие вью инвалидировать, и какие изменения это отслеживание молча пропускает?

Макрос превращает каждое хранимое свойство в вычисляемое, которое сообщает о чтениях и записях в `ObservationRegistrar`, а SwiftUI выполняет `body` под отслеживанием наблюдения, поэтому вью зависит ровно от тех свойств, которых коснулась за этот прогон.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-62913)

### 21. Модель пишет свои свойства из фонового сетевого колбэка, и список обновляется с опозданием, глючит или падает. Чего на самом деле требует SwiftUI?

Каждая запись, которую может увидеть вью, должна происходить на main actor. Вью SwiftUI, `@State` и рендеринг, читающий вашу модель, изолированы `@MainActor`, поэтому запись из другого потока означает гонку данных (data race), а не просто медленное обновление.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-62914)

### 22. Найдите баг: чат-сервер фиксирует новое подключение при каждой перерисовке родительского экрана, хотя у пользователя открыт один чат.

```swift
// ChatSocket wraps a WebSocket: connect() opens it, disconnect() closes it, messages() streams incoming text
@MainActor @Observable final class ChatModel {
  var messages: [String] = []
  private let socket = ChatSocket(url: supportURL)

  init() {
    Task {                               // bug: side effects in init
      socket.connect()
      for await message in socket.messages() { messages.append(message) }
    }
  }
}

struct SupportChatScreen: View {
  @State private var model = ChatModel()
  var body: some View {
    List(model.messages, id: \.self) { Text($0) }
  }
}
```

`@State` вычисляет начальное значение при каждом создании структуры вью, а SwiftUI пересоздаёт `SupportChatScreen` при каждой перерисовке родителя. Каждый раз новый `ChatModel` открывает сокет в `init`; `@State` оставляет первую модель и выбрасывает остальные, но они уже подключились.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-62923)

### 23. Экран на SwiftUI подтормаживает, и вы подозреваете слишком частые вычисления body. Как это подтвердить и что менять?

Сначала докажите, какие вью перерисовываются и почему, потом устраняйте причину. Инструменты: `Self._printChanges()` внутри подозрительного `body` и инструмент SwiftUI в Instruments, запущенный на release-сборке на реальном устройстве.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-62906)

### 24. Как организовать владение состоянием и поток данных в большом SwiftUI-приложении?

У каждого куска состояния ровно один владелец на самом узком уровне, которому оно нужно. Данные идут вниз как значения или наблюдаемые модели, а изменения возвращаются наверх как вызовы методов владельца, а не как запись в копию.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-62920)
