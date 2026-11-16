# Swift: ООП и протоколы: вопросы на собеседовании

Классы, struct, enum, протоколы, расширения, protocol-oriented programming. Junior · 26 вопросов.

**[Тренировать тему на irbisa.com →](https://irbisa.com/ru/topic/swift-oop-protocols/)** · [Все темы по iOS](../README.ru.md#все-темы)

### 1. Чем структура отличается от класса в Swift?

Помимо копирования и разделения, у структуры нет наследования, `deinit` и идентичности, а у класса нет поэлементного (memberwise) инициализатора, `mutating` и автоматического `Sendable`.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-2501)

### 2. Назовите основные принципы ООП и покажите, как каждый из них выражается в Swift.

Инкапсуляция, наследование, полиморфизм и абстракция; Swift поддерживает все четыре, но там, где классические ООП-языки опираются на иерархии классов, Swift опирается на протоколы и значимые типы.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-62517)

### 3. Чем перегрузка (overloading) отличается от переопределения (overriding), и что может переопределить подкласс?

Перегрузка означает несколько функций с одним именем и разными сигнатурами, нужную выбирает компилятор в точке вызова; переопределение означает замену унаследованной реализации в подклассе, и выбирается она во время выполнения по реальному классу объекта.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-62510)

### 4. Что такое протокол в Swift и чем он отличается от интерфейса или абстрактного класса?

Протокол представляет собой контракт из методов, свойств, инициализаторов и ассоциированных типов, который могут принять классы, структуры и enum, в том числе чужие типы; в отличие от абстрактного класса он не хранит состояние.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-2502)

### 5. Чем raw values отличаются от associated values в enum Swift?

Raw value задаёт для каждого case одну фиксированную константу одного и того же типа, известную на этапе компиляции; associated value представляет собой данные, которые прикрепляются к case при создании значения, и у разных case они могут быть разных типов.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-2503)

### 6. Что можно добавить к типу через extension в Swift, а что нельзя?

Extension добавляет вычисляемые свойства, методы, инициализаторы, сабскрипты, вложенные типы и соответствие протоколам к любому типу, даже чужому; добавить хранимые свойства или переопределить существующие члены он не может.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-2504)

### 7. Что делает `typealias` и создаёт ли он новый тип?

`typealias` даёт существующему типу второе имя, но не создаёт новый тип. Псевдоним и исходный тип взаимозаменяемы везде, поэтому псевдоним улучшает читаемость, но не добавляет типобезопасности.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-62522)

### 8. Какие уровни доступа есть в Swift и чем `public` отличается от `open`?

В Swift шесть уровней: `open`, `public`, `package`, `internal` (по умолчанию), `fileprivate` и `private`; `public` и `open` видны из других модулей, но только `open` позволяет этим модулям наследоваться от класса и переопределять его члены.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-2506)

### 9. Что такое `lazy`-свойство, чем оно отличается от вычисляемого и наблюдаемого свойства и может ли оно вычислиться больше одного раза?

`lazy var` выполняет свой инициализатор при первом чтении и запоминает результат, а `willSet`/`didSet` выполняют код вокруг каждого присваивания; `lazy` вычисляется один раз, если только два потока не прочитают его впервые одновременно.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-2507)

### 10. Что делает `mutating` и почему методам структуры он нужен, а методам класса нет?

`mutating` помечает метод структуры или enum, который меняет `self`: `self` передаётся как `inout`, и изменение записывается обратно в переменную вызывающего кода. Метод класса меняет объект через ссылку, а сама ссылка при этом не меняется.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-62506)

### 11. Что требуют `Equatable` и `Hashable` и что произойдёт, если у двух разных значений одинаковый хэш?

`Equatable` требует `==`, а `Hashable` добавляет `hash(into:)` с одним правилом: равные значения должны давать равные хэши. Одинаковый хэш у двух разных значений означает обычную коллизию: `Set` и `Dictionary` различают их через `==`, просто немного медленнее.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-62502)

### 12. Это объявление делегата не компилируется. Какая ошибка и как её исправить?

```swift
protocol DownloaderDelegate {
  func downloader(_ downloader: Downloader, didFinish text: String)
}

final class Downloader {
  weak var delegate: DownloaderDelegate?

  func finish(with text: String) {
    delegate?.downloader(self, didFinish: text)
  }
}
```

`weak` работает только с экземплярами классов, а этому протоколу может соответствовать и структура, поэтому компилятор выдаёт `'weak' must not be applied to non-class-bound 'any DownloaderDelegate'`. Протокол нужно ограничить классами: `protocol DownloaderDelegate: AnyObject`.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-62503)

### 13. Что такое `deinit`, когда он вызывается и в каком потоке выполняется?

`deinit` служит методом очистки класса, который вызывается автоматически прямо перед тем, как ARC освободит экземпляр, то есть когда исчезает последняя сильная ссылка. Он выполняется синхронно в том потоке, который отпустил эту последнюю ссылку, и это не обязательно главный поток.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-62518)

### 14. Чем отличаются `static`- и `class`-члены и что меняет `final`?

Оба объявляют члены типа, а не экземпляра, но `class`-член подкласс может переопределить, а `static` нет: в классе `static` означает `class final`.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-62505)

### 15. Чем отличаются `self` и `Self` в Swift?

`self` означает значение: текущий экземпляр, а внутри `static`- или `class`-метода сам тип. `Self` означает тип: в протоколе он обозначает тип, который этому протоколу соответствует, в структуре или enum обозначает сам этот тип, а в классе обозначает динамический тип во время выполнения.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-62523)

### 16. Как устроена инициализация классов в Swift: designated и convenience-инициализаторы, две фазы и failable-инициализаторы?

Designated-инициализатор присваивает значения всем хранимым свойствам своего класса и затем вызывает designated-инициализатор суперкласса; convenience-инициализатор обязан вызвать другой инициализатор того же класса. Так `self` не читается до полной инициализации цепочки.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-62501)

### 17. Что такое протокол-ориентированное программирование и чем оно отличается от наследования классов?

Протокол-ориентированное программирование (POP) собирает поведение из маленьких протоколов с реализациями по умолчанию в extension, а не из дерева базовых классов. Тип берёт ровно те возможности, которые ему нужны, и участвовать могут структуры и enum, а не только классы.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-2505)

### 18. Что выведет этот код и почему два вызова ведут себя по-разному?

```swift
protocol Greeter {
  func greet() -> String
}

extension Greeter {
  func greet() -> String { "default greet" }
  func shout() -> String { "default shout" }
}
struct Robot: Greeter {
  func greet() -> String { "beep" }
  func shout() -> String { "BEEP" }
}

let robot: any Greeter = Robot()
print(robot.greet())
print(robot.shout())
```

Выведет `beep`, а затем `default shout`. `greet()` объявлен в протоколе, значит, это требование, которое вызывается через witness table, и побеждает версия типа; `shout()` есть только в extension, поэтому компилятор выбирает его статически по объявленному типу `any Greeter`.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-62504)

### 19. Могут ли у протокола в Swift быть необязательные методы? Сравните `@objc optional` и реализацию по умолчанию.

Помечать требования как `optional` можно только в `@objc`-протоколе, и тогда соответствовать ему могут только классы; в чистом Swift тот же эффект даёт реализация по умолчанию в расширении протокола.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-62515)

### 20. Почему эта структура не компилируется и как вместо неё смоделировать связный список или дерево?

```swift
struct Node {
  let value: Int
  var next: Node?
}

var head = Node(value: 1, next: Node(value: 2, next: nil))
```

Структура хранится inline, поэтому `Node`, содержащий опциональный `Node`, имел бы бесконечный размер; компилятор сообщает «value type 'Node' cannot have a stored property that recursively contains it». `Optional` не спасает, потому что это тоже значимый тип, хранящийся inline.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-62519)

### 21. Когда варианты лучше моделировать через enum с associated values, а когда через протокол и отдельный тип на каждый вариант?

Enum выбирают, когда весь набор вариантов принадлежит вам и вы хотите, чтобы компилятор нашёл каждое место, которое надо поменять; протокол выбирают, когда новые варианты должны добавляться без правки существующего кода, например из другого модуля.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-62509)

### 22. Найдите баг: после обновления 2.3 заказы, закэшированные предыдущей версией, показывают неверный статус.

```swift
enum OrderStatus: Int, Codable {
  case draft      // added in 2.3
  case paid
  case shipped
  case delivered
}

struct Order: Codable {
  let id: Int
  var status: OrderStatus
}

// Orders are cached on disk between launches
func loadCachedOrder(from data: Data) throws -> Order {
  try JSONDecoder().decode(Order.self, from: data)
}
```

Неявные `Int` raw values задаются позициями: вставка `draft` в начало сдвигает все кейсы на единицу, поэтому сохранённая `1`, означавшая `shipped` в 2.2, теперь декодируется как `paid`, а старые заказы `delivered` показываются как `shipped`. Ничего не падает, данные просто молча неверные.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-62520)

### 23. Почему не-final класс должен помечать инициализатор, выполняющий требование `init` из протокола, как `required`?

Каждый подкласс наследует соответствие протоколу, значит, у каждого подкласса тоже должен быть этот инициализатор, и через `required` компилятор это гарантирует. У `final`-класса, структуры или enum подклассов нет, и ключевое слово не нужно.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-62511)

### 24. Как устроена диспетчеризация методов в Swift (static, vtable, witness table и message dispatch) и сколько стоит каждая?

Swift использует самую дешёвую диспетчеризацию, которую допускает объявление: прямые вызовы для значимых типов и `final`-членов, vtable для переопределяемых методов классов, witness table для требований протоколов и сообщения Objective-C только для `@objc dynamic`.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-62514)

### 25. Найдите баг: pull-to-refresh вызывает `refreshAll`, но экран ленты так и не перезагружается.

```swift
@MainActor protocol Refreshable: AnyObject {
  func refresh()
}
extension Refreshable {
  func refresh() {}  // default: nothing to reload
}

class BaseViewController: UIViewController, Refreshable {}

final class FeedViewController: BaseViewController {
  func refresh() { reloadFeed() }
}

@MainActor func refreshAll(_ screens: [any Refreshable]) {
  screens.forEach { $0.refresh() }
}
```

`BaseViewController` соответствует протоколу через пустую реализацию по умолчанию из extension, поэтому witness table протокола указывает на неё. `FeedViewController.refresh()` оказывается просто новым методом, `refreshAll` его никогда не вызывает, и pull-to-refresh на экране ленты ничего не делает.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-62521)

### 26. Тип соответствует двум протоколам, и каждый даёт свою реализацию `refresh()` по умолчанию. Что сделает Swift и как разрешить конфликт?

Если ни одна реализация по умолчанию не специфичнее другой, Swift не угадывает: либо соответствие протоколу не компилируется, либо вызов неоднозначен, а исправить это можно, реализовав `refresh()` в самом типе.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-62512)
