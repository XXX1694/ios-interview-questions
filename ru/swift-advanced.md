# Swift: дженерики и продвинутое: вопросы на собеседовании

Generics, opaque types, property wrappers, result builders, key paths. Middle · 25 вопросов.

**[Тренировать тему на irbisa.com →](https://irbisa.com/ru/topic/swift-advanced/)** · [Все темы по iOS](../README.ru.md#все-темы)

### 1. Что делает `Codable` и как декодировать JSON, ключи которого не совпадают с именами свойств?

`Codable` означает `Encodable & Decodable`: объявите соответствие, и компилятор сгенерирует код, который по именам переводит хранимые свойства в JSON (или другой формат) и обратно. Если имена расходятся, ключи переименовывают через enum `CodingKeys` или задают стратегию декодирования ключей.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-2606)

### 2. Что такое generics в Swift и что может выразить `where`, чего не могут угловые скобки?

Generics позволяют написать одну функцию или тип для любого типа, удовлетворяющего ограничениям, при этом компилятор проверяет каждое использование: никаких `Any` и приведений типов. `where` добавляет ограничения на associated types и связи между параметрами типа.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-2601)

### 3. Что такое associated type, какие ограничения он накладывает на протокол и что меняют primary associated types?

Под associated type понимают тип-заполнитель внутри протокола, который каждый соответствующий тип подставляет сам, как `Element` в `Collection`. Он делает протокол обобщённым, и имя протокола само по себе уже не описывает один законченный тип.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-62615)

### 4. Какая строка этого кода не компилируется и при чём здесь conditional conformance?

```swift
struct Box<Value> {
  let value: Value
}
extension Box: Equatable where Value: Equatable {}

struct Token { let raw: String }

print(Box(value: 1) == Box(value: 1))
print([Box(value: "a")] == [Box(value: "a")])
print(Box(value: Token(raw: "a")) == Box(value: Token(raw: "a")))
```

Третий `print`. `Box` является `Equatable` только `where Value: Equatable`, а `Token` не соответствует `Equatable`, поэтому у `Box<Token>` нет `==`: «operator function '==' requires that 'Token' conform to 'Equatable'». Первые две строки компилируются, потому что `Box<Int>` и `Box<String>` выполняют условие.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-62616)

### 5. Чем отличаются `some` и `any` в Swift и как они связаны с generics?

`some Shape` обозначает непрозрачный тип (opaque type): один конкретный тип, который компилятор знает, но скрывает от вызывающего. `any Shape` обозначает экзистенциальный тип (existential): коробка, в которой может лежать значение любого соответствующего типа.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-2602)

### 6. Эта функция не компилируется. Почему и как это исправить?

```swift
protocol Shape { func area() -> Double }
struct Circle: Shape { let r: Double; func area() -> Double { .pi * r * r } }
struct Square: Shape { let side: Double; func area() -> Double { side * side } }

func makeShape(rounded: Bool) -> some Shape {
  if rounded {
    return Circle(r: 1)
  }
  return Square(side: 2)
}
```

`some Shape` обещает один конкретный тип на всю функцию, а два `return` возвращают `Circle` и `Square`. Компилятор сообщает, что return statements «do not have matching underlying types».

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-62620)

### 7. Что означает `@escaping` и почему параметры-замыкания по умолчанию non-escaping?

`@escaping` говорит, что замыкание может быть вызвано после возврата из функции (его сохраняют или передают в асинхронную работу). Non-escaping выбрано по умолчанию, потому что замыкание, которое не переживает вызов, дешевле и безопаснее: оно не может создать цикл сильных ссылок.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-62602)

### 8. Что делает `@autoclosure` и почему `&&`, `??` и `assert` принимают аргументы именно так?

`@autoclosure` сам оборачивает выражение-аргумент в замыкание: вызывающий пишет обычное значение, а функция решает, вычислять ли его и когда. Это ленивое вычисление без фигурных скобок: `??` вычисляет правую часть, только если левая равна `nil`.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-62626)

### 9. Что такое key paths в Swift и где их использовать?

Key path представляет собой проверяемую компилятором ссылку на свойство, записываемую как `\User.name`. Её можно хранить и передавать, а потом читать или записывать это свойство у любого экземпляра.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-2605)

### 10. Как работает property wrapper и как написать свой?

Property wrapper определяется как тип с атрибутом `@propertyWrapper`, который владеет хранилищем свойства: для `@Clamped var volume` компилятор хранит скрытое `_volume` и проводит каждое чтение и запись через его `wrappedValue`.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-2603)

### 11. Найдите баг: `finished` в итоге меньше числа завершённых загрузок, хотя каждое обращение к нему идёт через блокировку.

```swift
@propertyWrapper
final class Locked<Value>: @unchecked Sendable {
  private let lock = NSLock()
  private var value: Value
  init(wrappedValue: Value) { value = wrappedValue }

  var wrappedValue: Value {
    get { lock.withLock { value } }
    set { lock.withLock { value = newValue } }
  }
}

final class DownloadStats: @unchecked Sendable {
  @Locked var finished = 0
  func markFinished() { finished += 1 }  // called from many download tasks
}
```

`finished += 1` выполняется не как одна операция под блокировкой: сначала вызывается геттер (lock, чтение, unlock), потом сеттер (lock, запись, unlock). Два потока могут прочитать одно и то же значение и оба записать его плюс один, и инкременты теряются.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-62623)

### 12. Что такое result builder и почему `if` или `for` внутри своего билдера иногда не компилируется?

Result builder представляет собой тип с атрибутом `@resultBuilder`, чьи статические методы `build…` компилятор вызывает, чтобы превратить инструкции замыкания в одно значение. К ним относится и `@ViewBuilder` в SwiftUI. Каждая конструкция управления работает, только если билдер реализует соответствующий метод.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-2604)

### 13. Чем `Sequence` отличается от `Collection` и как заставить свой тип работать в цикле `for-in`?

`Sequence` обещает только, что по элементам можно пройти итератором, возможно, лишь один раз. `Collection` добавляет индексы, сабскрипт и повторяемый обход без разрушения данных. Для `for-in` достаточно `Sequence`.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-62605)

### 14. Чем отличаются `#if`, `#available` и `@available` в Swift?

`#if` проверяет условие во время компиляции: код в ложной ветке вообще не попадает в бинарник. `if #available` проверяет во время выполнения версию ОС, на которой запущено приложение, а `@available` помечает объявление, чтобы компилятор заставил каждого вызывающего сделать эту проверку.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-62619)

### 15. Как использовать Objective-C-код из Swift и Swift-код из Objective-C в одном таргете и что не может пересечь эту границу?

Objective-C доступен в Swift через bridging header (в фреймворке через umbrella header). Swift доступен в Objective-C через сгенерированный `YourModule-Swift.h`, но только объявления с `@objc` в подклассах `NSObject` и типы, которые Objective-C умеет представить.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-62622)

### 16. Что выведет этот код и почему `map` вызывается чаще, чем можно ожидать?

```swift
let numbers = [1, 2, 3, 4, 5, 6]
let squares = numbers.lazy
  .map { n -> Int in
    print("map \(n)")
    return n * n
  }
  .filter { $0 % 2 == 0 }

print("start")
print(squares.first!)
print(squares.first!)
```

Выведет `start`, затем `map 1`, `map 2`, `map 2`, `4`, и то же ещё раз для второго `first`. Ленивая цепочка хранит замыкания, а не результаты, поэтому каждое обращение выполняет их заново, а `first` вызывает `map` для найденного элемента дважды: при поиске начального индекса и при обращении по нему.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-62621)

### 17. Что такое type erasure и когда нужен самописный type eraser, если есть `any P<T>`?

Стирание типа (type erasure) прячет конкретный тип за обёрткой с фиксированным типом, чтобы разные типы, соответствующие одному протоколу, можно было хранить и передавать как один тип. `AnyPublisher`, `AnySequence`, `AnyHashable` и `AnyView` из SwiftUI относятся к стирающим обёрткам (type erasers).

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-62601)

### 18. Как Swift выполняет generic-код и почему параметр `any` может быть медленнее generic-параметра?

Оптимизатор может специализировать generic-функцию, создав копию под каждый встреченный конкретный тип, и тогда вызовы прямые и могут инлайниться. Если не может, работает одно общее тело, которое получает метаданные типа и witness tables; значение `any` всегда идёт этим косвенным путём.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-62611)

### 19. Что такое phantom type и от какой ошибки он защищает?

Фантомным типом (phantom type) называют generic-параметр, который тип никогда не хранит; он нужен только для того, чтобы компилятор различал в остальном одинаковые значения. `ID<User>` и `ID<Order>` оба оборачивают `String`, но передать один вместо другого уже не получится.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-62618)

### 20. Что такое typed throws (`throws(MyError)`) и когда использовать их вместо обычного `throws`?

Начиная со Swift 6 функция может объявить точный тип ошибки, `throws(ParseError)`, и тогда `catch` получает `ParseError`, а не `any Error`, и вы можете разобрать его в `switch` исчерпывающе. Обычный `throws` по-прежнему правильный выбор по умолчанию для большей части кода.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-62617)

### 21. Передача большого `Array` в Swift выглядит бесплатной. Как работает copy-on-write и как реализовать его для своего типа?

Буфер общий для всех копий, пока кто-то не начнёт писать; перед записью проверка уникальности решает, нужно ли сначала клонировать буфер.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-62604)

### 22. Найдите баг: очистка копии этого `Playlist` опустошает и оригинал.

```swift
struct Playlist {
  private final class Storage {
    var ids: [Int]
    init(ids: [Int]) { self.ids = ids }
  }
  private var storage = Storage(ids: [])
  var ids: [Int] { storage.ids }

  mutating func removeAll() { storage.ids.removeAll() }

  mutating func append(_ id: Int) {
    if !isKnownUniquelyReferenced(&storage) { storage = Storage(ids: storage.ids) }
    storage.ids.append(id)
  }

  mutating func insert(_ id: Int, at index: Int) {
    if !isKnownUniquelyReferenced(&storage) { storage = Storage(ids: storage.ids) }
    storage.ids.insert(id, at: index)
  }
}
```

`removeAll()` пишет в общий `Storage` без проверки уникальности. После `var draft = saved` оба значения указывают на один объект, поэтому очистка `draft` очищает и `saved`: структура незаметно получает ссылочную семантику.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-62624)

### 23. Как `weak` и `unowned` устроены в Swift изнутри (side table, три счётчика ссылок), и почему память объекта может пережить его `deinit`?

Каждый объект Swift хранит счётчики strong и unowned прямо в заголовке, а первая `weak`-ссылка переносит счётчики в side table, на которую и указывают weak-ссылки. `deinit` выполняется, когда strong-счётчик доходит до нуля, но память объекта освобождается, только когда обнулится и unowned-счётчик.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-62625)

### 24. Нужна обёртка над сырым файловым дескриптором, которую нельзя скопировать, закрыть дважды или использовать после закрытия. Что даёт `~Copyable`?

Отказ от `Copyable` убирает неявные копии, поэтому у значения в каждый момент ровно один владелец, и компилятор, а не код-ревью, доказывает, что дескриптор закрывается один раз и после этого не используется.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-62612)

### 25. Когда писать макрос, а не property wrapper, и почему `@Observable` сделан макросом?

Property wrapper меняет то, как читается и пишется одно хранимое свойство во время выполнения. Макрос генерирует код во время компиляции и может добавить члены, аксессоры и соответствия протоколам целому объявлению. `@Observable` должен переписать каждое свойство класса и добавить соответствие.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-62603)
