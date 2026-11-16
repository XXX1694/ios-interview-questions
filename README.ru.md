# Вопросы на собеседовании по iOS с ответами

[English version](README.md)

685 вопросов на собеседовании по iOS, Swift, SwiftUI и UIKit в 31 теме, от junior до senior, к каждому краткий ответ. Каждый вопрос ведёт к полному ответу и коду на **[irbisa.com](https://irbisa.com/ru/ios/)**: там их можно тренировать с интервальным повторением, на mock-интервью на время и пройти пятиминутную диагностику, которая покажет слабые темы.

## Содержание

- [Самые частые вопросы](#самые-частые-вопросы)
- [Все темы](#все-темы)
- [Тренировка](#тренировка)

## Самые частые вопросы

Вопросы по iOS, которые чаще всего задают на реальных собеседованиях.

### 1. В чём разница между `let` и `var` в Swift?

Константе `let` значение присваивается ровно один раз, а переменную `var` можно переприсваивать сколько угодно; по умолчанию пишут `let` и переходят на `var`, только когда значение действительно должно меняться.

[Полный ответ →](https://irbisa.com/ru/topic/swift-basics/#q-2401) · [Основы Swift](ru/swift-basics.md)

### 2. Что такое Optional в Swift, как он устроен и как его безопасно развернуть?

Optional либо хранит значение, либо хранит `nil`, поэтому «значения может не быть» проверяет компилятор, а не падение в рантайме; `T?` служит сокращением для обобщённого enum `Optional<T>` с кейсами `.some(T)` и `.none`.

[Полный ответ →](https://irbisa.com/ru/topic/swift-basics/#q-2402) · [Основы Swift](ru/swift-basics.md)

### 3. В чём разница между типами-значениями и ссылочными типами в Swift?

Тип-значение копируется при присваивании и передаче в функцию, поэтому у каждой переменной свои данные; ссылочный тип разделяется, поэтому несколько переменных указывают на один экземпляр и видят изменения друг друга.

[Полный ответ →](https://irbisa.com/ru/topic/swift-basics/#q-2403) · [Основы Swift](ru/swift-basics.md)

### 4. Что такое замыкание в Swift и это тип-значение или ссылочный тип?

Замыкание представляет собой блок кода, который можно сохранить в переменную и передавать дальше, и он помнит переменные той области, где был создан; замыкания, как и функции, относятся к ссылочным типам.

[Полный ответ →](https://irbisa.com/ru/topic/swift-basics/#q-2404) · [Основы Swift](ru/swift-basics.md)

### 5. В чём разница между `weak` и `unowned` и когда в замыкании нужен `[weak self]`?

Ни одна из них не удерживает объект: `weak` работает как optional, который становится `nil`, когда объект освобождён, а `unowned` неопционален и падает, если обратиться к нему после того, как объект исчез. Используйте `[weak self]` в escaping-замыкании, которым владеет сам `self`.

[Полный ответ →](https://irbisa.com/ru/topic/swift-basics/#q-62415) · [Основы Swift](ru/swift-basics.md)

### 6. Как устроена обработка ошибок в Swift: `throws`, `try`, `try?`, `try!`, `do`/`catch` и `Result`?

Функция, которая может завершиться ошибкой, помечается `throws`, каждый её вызов помечается `try`, а ошибка либо обрабатывается в `do`/`catch`, либо передаётся вызывающему, так что возможность сбоя видна прямо в сигнатуре.

[Полный ответ →](https://irbisa.com/ru/topic/swift-basics/#q-2406) · [Основы Swift](ru/swift-basics.md)

### 7. Чем структура отличается от класса в Swift?

Помимо копирования и разделения, у структуры нет наследования, `deinit` и идентичности, а у класса нет поэлементного (memberwise) инициализатора, `mutating` и автоматического `Sendable`.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-2501) · [Swift: ООП и протоколы](ru/swift-oop-protocols.md)

### 8. Чем raw values отличаются от associated values в enum Swift?

Raw value задаёт для каждого case одну фиксированную константу одного и того же типа, известную на этапе компиляции; associated value представляет собой данные, которые прикрепляются к case при создании значения, и у разных case они могут быть разных типов.

[Полный ответ →](https://irbisa.com/ru/topic/swift-oop-protocols/#q-2503) · [Swift: ООП и протоколы](ru/swift-oop-protocols.md)

### 9. Что такое SwiftUI и в чём его плюсы и минусы по сравнению с UIKit?

SwiftUI представляет собой декларативный UI-фреймворк Apple: вы описываете, как выглядит экран при текущем состоянии, а фреймворк сам решает, какие изменения применить. В императивном UIKit вы создаёте view и затем сами их изменяете.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-2801) · [Основы SwiftUI](ru/swiftui-fundamentals.md)

### 10. Как работает `NavigationStack` и как переходить между экранами из кода или по диплинку?

`NavigationStack` хранит свои экраны как путь (path) из значений: `NavigationLink(value:)` добавляет значение, `.navigationDestination(for:)` превращает значение этого типа в экран, а кнопка «Назад» удаляет последнее. Привяжите path к состоянию, и навигация сводится к изменению массива.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-2804) · [Основы SwiftUI](ru/swiftui-fundamentals.md)

### 11. Расскажите о жизненном цикле `UIViewController`: какие колбэки вызываются один раз, а какие при каждом появлении, и где начинать загрузку данных?

`loadView` и `viewDidLoad` вызываются один раз на экземпляр контроллера; `viewWillAppear`, `viewIsAppearing`, `viewDidAppear` и два колбэка исчезновения вызываются каждый раз, когда экран показывается или скрывается.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-3002) · [Основы UIKit](ru/uikit-fundamentals.md)

### 12. Чем отличаются `frame` и `bounds` и когда `bounds.origin` не равен нулю?

`frame` задаёт положение и размер вью в системе координат superview; `bounds` описывает тот же прямоугольник в собственных координатах вью. `bounds.origin` не равен нулю, когда вью показывает смещённую часть своего содержимого: у scroll view `bounds.origin` совпадает с его `contentOffset`.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-3003) · [Основы UIKit](ru/uikit-fundamentals.md)

### 13. Что такое responder chain и как `hitTest(_:with:)` определяет, какая вью получит касание?

UIKit сначала выполняет hit-testing от окна вниз, чтобы найти самую глубокую вью под пальцем, и доставляет касание ей; всё, что эта вью не обработала, поднимается по responder chain: superview, view controller, окно, приложение.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-3004) · [Основы UIKit](ru/uikit-fundamentals.md)

### 14. Расскажите о своём опыте в iOS: какие приложения вы выпускали и чем занимаетесь в Xcode изо дня в день?

Начните с **приложения, которое выпускаете сейчас**, и своей роли в нём, добавьте один прошлый шаг, который объясняет ваш путь, и закончите тем, почему эта iOS-команда подходит под ваши планы.

[Полный ответ →](https://irbisa.com/ru/topic/ios-behavioral-interview/#q-76401) · [Поведенческое интервью](ru/ios-behavioral-interview.md)

### 15. Что заставляет вас искать место за пределами текущей iOS-работы и чем вас зацепила эта компания?

Назовите одну **честную причину** как движение к чему-то, а затем свяжите её с конкретной деталью, которую вы заметили в iOS-продукте этой компании.

[Полный ответ →](https://irbisa.com/ru/topic/ios-behavioral-interview/#q-76402) · [Поведенческое интервью](ru/ios-behavioral-interview.md)

### 16. Расскажите об iOS-фиче, которой вы гордитесь. За что вы отвечали лично?

Выберите фичу, которая хорошо использовала **возможность платформы**, например виджет, Live Activity или App Intents, и объясните, почему это было важно пользователям, что решили вы и что изменилось после.

[Полный ответ →](https://irbisa.com/ru/topic/ios-behavioral-interview/#q-76403) · [Поведенческое интервью](ru/ios-behavioral-interview.md)

### 17. Какое вознаграждение вы ожидаете на этой iOS-позиции?

Назовите **вилку из исследования**, а не от текущей зарплаты, и уточните, идёт ли речь о сумме до налогов за год или на руки за месяц и что итог зависит от всего пакета.

[Полный ответ →](https://irbisa.com/ru/topic/ios-behavioral-interview/#q-76405) · [Поведенческое интервью](ru/ios-behavioral-interview.md)

### 18. Чем отличаются `some` и `any` в Swift и как они связаны с generics?

`some Shape` обозначает непрозрачный тип (opaque type): один конкретный тип, который компилятор знает, но скрывает от вызывающего. `any Shape` обозначает экзистенциальный тип (existential): коробка, в которой может лежать значение любого соответствующего типа.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-2602) · [Swift: дженерики и продвинутое](ru/swift-advanced.md)

### 19. Передача большого `Array` в Swift выглядит бесплатной. Как работает copy-on-write и как реализовать его для своего типа?

Буфер общий для всех копий, пока кто-то не начнёт писать; перед записью проверка уникальности решает, нужно ли сначала клонировать буфер.

[Полный ответ →](https://irbisa.com/ru/topic/swift-advanced/#q-62604) · [Swift: дженерики и продвинутое](ru/swift-advanced.md)

### 20. Чем конкурентность отличается от параллельности, а чем синхронное выполнение отличается от асинхронного?

Конкурентность означает, что несколько задач находятся в работе одновременно и могут чередоваться; параллельность означает, что они буквально выполняются в один момент на разных ядрах. Синхронный вызов заставляет вызывающего ждать результата, асинхронный даёт продолжить работу и отдаёт результат позже.

[Полный ответ →](https://irbisa.com/ru/topic/swift-concurrency/#q-62715) · [Swift Concurrency](ru/swift-concurrency.md)

### 21. Что такое GCD, чем отличаются serial- и concurrent-очереди и чем `sync` отличается от `async`?

GCD (Grand Central Dispatch) представляет собой библиотеку Apple: вы отправляете замыкания в очереди, а система выполняет их на своём пуле потоков. Serial-очередь выполняет по одной задаче в порядке FIFO, concurrent запускает несколько сразу; `async` возвращается сразу, `sync` блокирует до конца замыкания.

[Полный ответ →](https://irbisa.com/ru/topic/swift-concurrency/#q-62716) · [Swift Concurrency](ru/swift-concurrency.md)

### 22. Как работает `async`/`await` в Swift и чем он отличается от completion handler’ов?

`async`-функция может приостановиться на каждом `await` и продолжить позже, а пока она приостановлена, она не держит поток. По сравнению с completion handler’ами код читается сверху вниз, ошибки идут через `try`/`catch`, а компилятор гарантирует, что результат придёт ровно один раз.

[Полный ответ →](https://irbisa.com/ru/topic/swift-concurrency/#q-2701) · [Swift Concurrency](ru/swift-concurrency.md)

### 23. Что такое актор, как он предотвращает гонки данных и почему его методы снаружи вызываются через `await`?

Актор представляет собой ссылочный тип, который защищает своё изменяемое состояние: в нём одновременно выполняется только один фрагмент кода, на собственном последовательном executor’е. Снаружи к нему обращаются через `await`, потому что вызову, возможно, придётся подождать, пока актор освободится.

[Полный ответ →](https://irbisa.com/ru/topic/swift-concurrency/#q-2703) · [Swift Concurrency](ru/swift-concurrency.md)

### 24. Чем `@State` отличается от `@Binding` в SwiftUI?

`@State` объявляет хранилище, которым владеет сама вью: SwiftUI держит значение вне структуры вью и перерисовывает её, когда оно меняется. `@Binding` работает как ссылка на чтение и запись к состоянию, которым владеет кто-то другой: ребёнок может менять значение, но не хранит его.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-2901) · [Состояние в SwiftUI](ru/swiftui-state.md)

### 25. Чем отличаются `@Environment`, `@EnvironmentObject` и `@ObservedObject`, и когда брать environment вместо `init`?

`@ObservedObject` получает объект явно через инициализатор, а `@EnvironmentObject` и `@Environment` читают то, что внедрено выше по дереву, поэтому промежуточным вью не нужно передавать это дальше (prop drilling).

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-state/#q-2903) · [Состояние в SwiftUI](ru/swiftui-state.md)

### 26. Что такое MVC в UIKit и почему он так часто превращается в Massive View Controller?

В MVC от Apple контроллер связывает модель и view, но `UIViewController` ещё и владеет жизненным циклом своей view, поэтому вёрстка, состояние, сеть и навигация оседают в одном классе, который нельзя протестировать без view.

[Полный ответ →](https://irbisa.com/ru/topic/ios-architecture/#q-3202) · [Архитектура iOS](ru/ios-architecture.md)

### 27. Что такое MVVM, какую проблему он решает и как выглядит в UIKit и в SwiftUI?

MVVM выносит логику представления и состояние экрана из view во view model, которая отдаёт готовые к показу значения; view только привязывается к ним и передаёт события пользователя, поэтому логику можно покрыть юнит-тестами без всякого UI.

[Полный ответ →](https://irbisa.com/ru/topic/ios-architecture/#q-63203) · [Архитектура iOS](ru/ios-architecture.md)

### 28. Из каких частей состоит стек Core Data и за что отвечает каждая?

Четыре части: `NSManagedObjectModel` описывает схему, `NSPersistentStoreCoordinator` связывает эту модель с файлами хранилища, `NSManagedObjectContext` служит черновиком, в котором вы получаете и меняете объекты, а `NSPersistentContainer` создаёт их все и владеет ими.

[Полный ответ →](https://irbisa.com/ru/topic/ios-persistence/#q-63418) · [Хранилище iOS](ru/ios-persistence.md)

### 29. Как реализовать синглтон в Swift, потокобезопасен ли он и что не так с синглтонами?

`static let shared` плюс `private init()`. Swift создаёт статическое свойство лениво и ровно один раз, даже если до него одновременно добираются несколько потоков. Проблема в дизайне: синглтон по сути глобальное состояние, до которого может дотянуться любая строка, поэтому зависимости прячутся, а тесты влияют друг на друга.

[Полный ответ →](https://irbisa.com/ru/topic/ios-design-patterns/#q-75503) · [ООП, SOLID и паттерны проектирования](ru/ios-design-patterns.md)

### 30. Как работает ARC в Swift и чем он отличается от сборщика мусора?

ARC освобождает экземпляр класса в тот момент, когда счётчик сильных ссылок на него падает до нуля; вызовы retain и release вставляет компилятор. Сборщик мусора, наоборот, ищет недостижимые объекты во время работы: это стоит пауз или фоновой работы, зато он освобождает и циклы ссылок, чего ARC не делает никогда.

[Полный ответ →](https://irbisa.com/ru/topic/ios-performance/#q-3501) · [Производительность iOS и память](ru/ios-performance.md)

## Все темы

| Тема | Уровень | Вопросов |
|---|---|---:|
| [Основы Swift](ru/swift-basics.md) | Junior | 26 |
| [Swift: ООП и протоколы](ru/swift-oop-protocols.md) | Junior | 26 |
| [Основы SwiftUI](ru/swiftui-fundamentals.md) | Junior | 25 |
| [Основы UIKit](ru/uikit-fundamentals.md) | Junior | 25 |
| [Auto Layout и UIKit-вёрстка](ru/uikit-layout.md) | Junior | 25 |
| [Git и командная работа](ru/ios-git-workflow.md) | Junior | 18 |
| [Поведенческое интервью](ru/ios-behavioral-interview.md) | Junior | 17 |
| [Swift: дженерики и продвинутое](ru/swift-advanced.md) | Middle | 25 |
| [Swift Concurrency](ru/swift-concurrency.md) | Middle | 25 |
| [Состояние в SwiftUI](ru/swiftui-state.md) | Middle | 24 |
| [Архитектура iOS](ru/ios-architecture.md) | Middle | 24 |
| [Сеть в iOS](ru/ios-networking.md) | Middle | 25 |
| [Хранилище iOS](ru/ios-persistence.md) | Middle | 25 |
| [Тестирование в iOS](ru/ios-testing.md) | Middle | 25 |
| [Жизненный цикл, фон и push](ru/ios-lifecycle-background.md) | Middle | 24 |
| [Расширения и WidgetKit](ru/ios-extensions-widgets.md) | Middle | 20 |
| [Доступность и локализация](ru/ios-accessibility-localization.md) | Middle | 25 |
| [UICollectionView и diffable data source](ru/uikit-collections.md) | Middle | 23 |
| [GCD и потоки](ru/gcd-threading.md) | Middle | 18 |
| [Безопасность, приватность и Keychain](ru/ios-security-keychain.md) | Middle | 20 |
| [Live coding на Swift](ru/swift-coding-round.md) | Middle | 18 |
| [StoreKit и релиз в App Store](ru/ios-storekit-release.md) | Middle | 19 |
| [ООП, SOLID и паттерны проектирования](ru/ios-design-patterns.md) | Middle | 18 |
| [Производительность iOS и память](ru/ios-performance.md) | Senior | 25 |
| [Combine](ru/combine.md) | Senior | 24 |
| [Строгая конкурентность Swift 6](ru/swift-strict-concurrency.md) | Senior | 19 |
| [Анимации и Core Animation](ru/ios-animations.md) | Senior | 20 |
| [Сборка Xcode и модуляризация](ru/ios-build-modularization.md) | Senior | 22 |
| [SwiftUI: layout, идентичность и отрисовка](ru/swiftui-layout-rendering.md) | Senior | 21 |
| [Objective-C runtime и интероп](ru/objc-runtime-interop.md) | Senior | 18 |
| [System Design на iOS](ru/ios-system-design.md) | Senior | 16 |

## Тренировка

Прочитать ответ и ответить самому: не одно и то же. На [irbisa.com](https://irbisa.com/ru/ios/) ты сначала отвечаешь, потом сверяешься с ответом, а вопросы, на которых споткнулся, возвращаются, пока не запомнятся.

---

Вопросы и ответы пишутся и обновляются на [irbisa.com](https://irbisa.com/ru/). Этот список собран из того же источника.
