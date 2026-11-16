# Objective-C runtime и интероп: вопросы на собеседовании

Отправка сообщений, swizzling, бриджинг, KVO изнутри, когда ObjC всё ещё нужен. Senior · 18 вопросов.

**[Тренировать тему на irbisa.com →](https://irbisa.com/ru/topic/objc-runtime-interop/)** · [Все темы по iOS](../README.ru.md#все-темы)

### 1. Что такое Objective-C runtime и что значит, что Objective-C отправляет сообщения, а не вызывает методы?

Objective-C runtime (`libobjc`) представляет собой библиотеку, которая хранит классы, методы и селекторы как данные во время работы приложения; `[obj doThing]` компилируется в `objc_msgSend(obj, @selector(doThing))`, и реализация ищется только в момент вызова.

[Полный ответ →](https://irbisa.com/ru/topic/objc-runtime-interop/#q-71417)

### 2. Что произойдёт, если в Objective-C отправить сообщение `nil`, и как Swift ведёт себя в той же ситуации?

Ничего не произойдёт и ничего не упадёт: `objc_msgSend` первым делом проверяет `nil` и возвращает ноль, то есть `nil`, `0`, `NO` или `0.0`. Swift делает этот случай явным через опционалы: `obj?.run()` пропускается и возвращает опционал; force unwrap от `nil` падает.

[Полный ответ →](https://irbisa.com/ru/topic/objc-runtime-interop/#q-71418)

### 3. Чем отличаются `@objc`, `@objcMembers` и `dynamic`, и почему одного `@objc` не хватает для KVO или swizzling?

`@objc` публикует член в Objective-C runtime (селектор и запись в списке методов), `@objcMembers` делает это для всего класса, а `dynamic` заставляет и сам Swift вызывать член через `objc_msgSend`. Без `dynamic` вызовы из Swift идут в обход рантайма.

[Полный ответ →](https://irbisa.com/ru/topic/objc-runtime-interop/#q-71419)

### 4. Что такое KVC и KVO, как наблюдать за свойством из Swift через `observe(_:options:changeHandler:)` и что заменяет KVO в новом Swift-коде?

KVC (key-value coding) читает и записывает свойства по строковому ключу во время выполнения; KVO (key-value observing) уведомляет наблюдателей, когда такое свойство меняется. Из Swift вы вызываете `observe(\.status)` на свойстве `@objc dynamic` и храните токен; для собственных новых моделей вместо этого берут `@Observable`.

[Полный ответ →](https://irbisa.com/ru/topic/objc-runtime-interop/#q-71433)

### 5. Чем категория в Objective-C отличается от class extension, и что будет, если две категории добавят метод с одинаковым именем?

Категория представляет собой именованное дополнение (`@interface UIView (Badge)`), которое добавляет методы любому классу при загрузке бинарника, даже без доступа к его исходникам; class extension (`@interface Foo ()`) является анонимной частью самого класса, компилируется с его `@implementation` и может добавлять ivar.

[Полный ответ →](https://irbisa.com/ru/topic/objc-runtime-interop/#q-71420)

### 6. Чем `+load` отличается от `+initialize`, и почему Swift-класс не может переопределить ни один из них?

`+load` выполняется сразу, один раз для каждого класса и каждой категории, где он реализован, при загрузке бинарника и до `main()`; `+initialize` выполняется лениво, прямо перед первым сообщением классу, и наследуется. Swift запрещает переопределять оба метода ошибкой компиляции.

[Полный ответ →](https://irbisa.com/ru/topic/objc-runtime-interop/#q-71421)

### 7. Что означают атрибуты свойств Objective-C `atomic`, `nonatomic`, `strong`, `weak`, `copy` и `assign`, и делает ли `atomic` свойство потокобезопасным?

Они говорят компилятору, как синтезировать аксессоры: `strong` удерживает объект, `weak` не удерживает и становится `nil`, `copy` хранит копию, `assign` хранит значение как есть, а `atomic` (по умолчанию) делает атомарным каждый отдельный get или set. Это не потокобезопасность: `visits += 1` остаётся незащищённым.

[Полный ответ →](https://irbisa.com/ru/topic/objc-runtime-interop/#q-71434)

### 8. Как добавить хранимое свойство классу, который вам не принадлежит, и чего стоят associated objects?

Extension и категории не умеют добавлять хранение, поэтому используют associated objects: `objc_setAssociatedObject` сохраняет значение для пары «объект + уникальный ключ» в отдельной таблице, а рантайм освобождает его при деаллокации объекта.

[Полный ответ →](https://irbisa.com/ru/topic/objc-runtime-interop/#q-71422)

### 9. Зачем в блоке Objective-C писать связку `weakSelf`/`strongSelf`, и что меняет `__block` под ARC?

Блок удерживает каждый захваченный объект, поэтому блок, который хранится в `self` и упоминает `self`, создаёт retain cycle; захват `__weak weakSelf` разрывает цикл, а превращение его в `strongSelf` внутри блока держит объект живым до конца этого выполнения блока.

[Полный ответ →](https://irbisa.com/ru/topic/objc-runtime-interop/#q-71423)

### 10. Как сделать заголовок Objective-C удобным для Swift: nullability, `NS_SWIFT_NAME`, `NS_REFINED_FOR_SWIFT` и другие макросы?

Аннотируйте заголовок, чтобы импортёр выдал нативные Swift-типы: nullability превращает `T!` в `T` или `T?`, lightweight generics превращают `NSArray *` в `[String]`, а `NS_SWIFT_NAME` и `NS_REFINED_FOR_SWIFT` исправляют имена и формы, которые плохо переводятся.

[Полный ответ →](https://irbisa.com/ru/topic/objc-runtime-interop/#q-71424)

### 11. Что происходит, когда объект получает сообщение, для которого у него нет метода, и как можно вмешаться в этот процесс?

Прежде чем упасть с «unrecognized selector sent to instance», рантайм даёт объекту три шанса: добавить метод (`resolveInstanceMethod:`), назвать другой объект, который его обработает (`forwardingTargetForSelector:`), или принять весь вызов как `NSInvocation` (`forwardInvocation:`).

[Полный ответ →](https://irbisa.com/ru/topic/objc-runtime-interop/#q-71425)

### 12. Что такое method swizzling, как делать его безопасно и чем он опасен в продакшене?

Swizzling меняет местами реализации двух селекторов во время работы приложения, и каждый вызов, например, `viewWillAppear:` сначала выполняет ваш код. Работает только для методов, которые вызываются через `objc_msgSend`, и меняет поведение для всего процесса.

[Полный ответ →](https://irbisa.com/ru/topic/objc-runtime-interop/#q-71426)

### 13. Что такое `isa`, и как `objc_msgSend` с его помощью, через кеш методов и цепочку суперклассов находит реализацию?

Каждый объект начинается с поля `isa`, которое указывает на его класс; `objc_msgSend` читает его, ищет селектор в кеше методов этого класса, а при промахе идёт по спискам методов вверх по цепочке суперклассов, кладёт найденный `IMP` в кеш и прыгает на него.

[Полный ответ →](https://irbisa.com/ru/topic/objc-runtime-interop/#q-71427)

### 14. Что выведет этот код, и что KVO делает с объектом, из-за чего вывод именно такой?

```swift
class Upload: NSObject {
  @objc dynamic var progress = 0.0
  @objc var status = "idle"
}

let upload = Upload()
let a = upload.observe(\.progress) { u, _ in print("progress", u.progress) }
let b = upload.observe(\.status) { u, _ in print("status", u.status) }

upload.progress = 0.5
upload.status = "running"
upload.setValue("done", forKey: "status")
```

Выведет `progress 0.5`, а затем `status done`. KVO подменяет класс объекта сгенерированным подклассом, сеттеры которого рассылают уведомления, а Swift отправляет сеттер через `objc_msgSend`, где подкласс может его перехватить, только для `@objc dynamic`.

[Полный ответ →](https://irbisa.com/ru/topic/objc-runtime-interop/#q-71428)

### 15. Найдите баг: аналитика экранов работает, пока пользователь iPad не откроет второе окно, а потом пропадает для всех экранов. Что не так с этим swizzling?

```swift
extension UIViewController {
  static func enableScreenTracking() {
    let original = class_getInstanceMethod(UIViewController.self, #selector(viewDidAppear(_:)))!
    let tracked = class_getInstanceMethod(UIViewController.self, #selector(tracked_viewDidAppear(_:)))!
    method_exchangeImplementations(original, tracked)
  }
  @objc dynamic func tracked_viewDidAppear(_ animated: Bool) {
    tracked_viewDidAppear(animated)  // runs the original viewDidAppear
    Analytics.screen(String(describing: type(of: self)))
  }
}
final class SceneDelegate: UIResponder, UIWindowSceneDelegate {
  func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
    UIViewController.enableScreenTracking()
  }
}
```

Swizzling выполняется при каждом подключении сцены, а не один раз за процесс: второе окно меняет реализации обратно, `viewDidAppear` снова оригинальный, и трекинг молча перестаёт работать, пока третье окно не включит его снова.

[Полный ответ →](https://irbisa.com/ru/topic/objc-runtime-interop/#q-71429)

### 16. Что такое toll-free bridging, и как bridge-приведения ARC и `Unmanaged` определяют, кто освобождает объект Core Foundation?

Toll-free пары вроде `CFStringRef` и `NSString` имеют одинаковое устройство в памяти, поэтому приведение использует тот же самый объект. ARC не управляет CF-объектами, поэтому в Objective-C bridge-приведение, а в Swift `Unmanaged` говорят, кто владеет ссылкой +1.

[Полный ответ →](https://irbisa.com/ru/topic/objc-runtime-interop/#q-71430)

### 17. Чего стоит бриджинг между `String` и `NSString`, и почему `NSRange`, построенный из `text.count`, пропускает совпадения?

Swift-`String` хранит UTF-8 и считает `Character` (графемные кластеры), а `NSString` работает в кодовых единицах UTF-16, поэтому бридж или перевод индексов может означать перекодирование и проход O(n), а `text.count` возвращает не ту длину, которую ждёт `NSRange`.

[Полный ответ →](https://irbisa.com/ru/topic/objc-runtime-interop/#q-71431)

### 18. Когда в 2026 году в iOS-проекте ещё нужен Objective-C, и как бы вы перевели большой Objective-C-модуль на Swift?

Новый код пишется на Swift, но Objective-C всё ещё оправдан для перехвата `NSException`, работы с рантаймом вроде `forwardInvocation:` и `+load`, обвязки на Objective-C++ для C++-кода, который C++-интероп Swift не умеет импортировать, и больших легаси-модулей и SDK, которые уже работают.

[Полный ответ →](https://irbisa.com/ru/topic/objc-runtime-interop/#q-71432)
