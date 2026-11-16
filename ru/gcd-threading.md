# GCD и потоки: вопросы на собеседовании

Очереди, QoS, DispatchGroup и семафоры, deadlock на main.sync, Operation, связь с async/await. Middle · 18 вопросов.

**[Тренировать тему на irbisa.com →](https://irbisa.com/ru/topic/gcd-threading/)** · [Все темы по iOS](../README.ru.md#все-темы)

### 1. Что такое `Thread` (`NSThread`) и почему в iOS-приложениях потоки почти никогда не создают вручную?

`Thread` представляет собой обёртку Foundation над POSIX-потоком: вы сами его создаёте, запускаете и отвечаете за его жизнь. В приложениях так делают редко, потому что GCD и Swift Concurrency держат пул переиспользуемых потоков и сами решают, сколько их запускать; это дешевле, и в этом сложнее ошибиться.

[Полный ответ →](https://irbisa.com/ru/topic/gcd-threading/#q-71517)

### 2. На каком потоке вызывается completion handler у `URLSession` и как из него обновить UI?

На фоновом: сессия вызывает completion handler на своей delegate-очереди, а у `URLSession.shared` это приватная serial `OperationQueue`, а не главная очередь. Данные разбирайте там же, а на главную очередь переходите через `DispatchQueue.main.async` только для работы с UI.

[Полный ответ →](https://irbisa.com/ru/topic/gcd-threading/#q-71518)

### 3. Как выполнить код с задержкой через GCD и как отменить его до запуска?

`DispatchQueue.main.asyncAfter(deadline: .now() + 0.3) { … }` планирует замыкание, но обычное замыкание отменить нельзя. Оберните работу в `DispatchWorkItem`, запланируйте его и вызовите у него `cancel()`, когда он больше не нужен.

[Полный ответ →](https://irbisa.com/ru/topic/gcd-threading/#q-71519)

### 4. Что такое Thread Sanitizer и чем он отличается от Main Thread Checker?

Thread Sanitizer (TSan) представляет собой инструмент времени выполнения, который находит гонки данных (data race): два потока обращаются к одной памяти без синхронизации, и хотя бы один из них пишет. Main Thread Checker сообщает только о вызовах UIKit и AppKit не с главного потока.

[Полный ответ →](https://irbisa.com/ru/topic/gcd-threading/#q-71520)

### 5. Что выведет этот код, если `run()` вызван на главном потоке, и на каком потоке выполнится каждый блок?

```swift
// run() is called on the main thread, for example from viewDidLoad
func run() {
  let serial = DispatchQueue(label: "com.app.serial")
  DispatchQueue.main.async { print("A", Thread.isMainThread) }
  serial.sync { print("B", Thread.isMainThread) }
  DispatchQueue.global().sync { print("C", Thread.isMainThread) }
  serial.async { print("D", Thread.isMainThread) }
  serial.sync { print("E", Thread.isMainThread) }
  print("F")
}
```

Он выведет `B true`, `C true`, `D false`, `E true`, `F` и в самом конце `A true`. `sync` выполняет блок на вызывающем потоке, здесь на главном, даже если очередь другая; `async` отдаёт блок рабочему потоку; `main.async` ждёт, пока `run()` вернёт управление.

[Полный ответ →](https://irbisa.com/ru/topic/gcd-threading/#q-71533)

### 6. Как barrier-блок превращает concurrent-очередь в reader-writer lock и когда флаг barrier ничего не делает?

Блок, отправленный с `.barrier`, ждёт, пока завершатся все блоки, отправленные в очередь раньше, выполняется один и только потом пропускает следующие блоки. Чтения идут через обычный `sync` и выполняются параллельно, а записи идут через barrier и выполняются эксклюзивно.

[Полный ответ →](https://irbisa.com/ru/topic/gcd-threading/#q-71521)

### 7. Найдите баг: этот потокобезопасный кэш падает при первом же вызове `image(for:orInsert:)`.

```swift
final class ImageCache: @unchecked Sendable {
  private let queue = DispatchQueue(label: "com.app.image-cache")
  private var storage: [URL: UIImage] = [:]
  func image(for url: URL) -> UIImage? {
    queue.sync { storage[url] }
  }

  func image(for url: URL, orInsert make: () -> UIImage) -> UIImage {
    queue.sync {
      if let cached = image(for: url) { return cached }
      let image = make()
      storage[url] = image
      return image
    }
  }
}
```

`image(for:orInsert:)` выполняется внутри `queue.sync` и вызывает `image(for:)`, который делает `queue.sync` на той же serial-очереди. Очередь ждёт саму себя: deadlock, который libdispatch обнаруживает и превращает в крэш.

[Полный ответ →](https://irbisa.com/ru/topic/gcd-threading/#q-71522)

### 8. Найдите баг: `loadAvatars` иногда так и не вызывает completion.

```swift
// api.fetchAvatar(id:completion:) is declared with a @MainActor completion, called on the main queue
@MainActor
func loadAvatars(ids: [String], completion: @escaping @MainActor ([UIImage]) -> Void) {
  let group = DispatchGroup()
  var images: [UIImage] = []
  for id in ids {
    group.enter()
    api.fetchAvatar(id: id) { result in
      guard case .success(let image) = result else { return }
      images.append(image)
      group.leave()
    }
  }
  group.notify(queue: .main) { completion(images) }
}
```

На пути ошибки колбэк выходит через `guard` до `group.leave()`, поэтому счётчик группы никогда не доходит до нуля и `notify` не срабатывает. Достаточно одного упавшего запроса из двадцати, и экран ждёт вечно, без крэша и без логов.

[Полный ответ →](https://irbisa.com/ru/topic/gcd-threading/#q-71523)

### 9. Найдите баг: этот таймер повтора запускается из фоновой очереди и никогда не срабатывает.

```swift
final class UploadRetrier: @unchecked Sendable {
  private let queue = DispatchQueue(label: "com.app.upload")
  private var retryTimer: Timer?

  // Called by the upload code when a request fails
  func uploadDidFail() {
    queue.async {
      self.retryTimer?.invalidate()
      self.retryTimer = Timer.scheduledTimer(withTimeInterval: 5, repeats: false) { [weak self] _ in
        self?.retry()
      }
    }
  }

  private func retry() { /* start the upload again */ }
}
```

`Timer.scheduledTimer` добавляет таймер в run loop текущего потока, а здесь это рабочий поток GCD, чей run loop никто не запускает. Таймер создаётся, лежит в этом run loop и не срабатывает никогда: ни ошибки, ни лога.

[Полный ответ →](https://irbisa.com/ru/topic/gcd-threading/#q-71534)

### 10. Что делает `DispatchQueue.concurrentPerform` и чем он лучше `global().async` в цикле для тяжёлых вычислений?

`concurrentPerform(iterations:execute:)` выполняет замыкание N раз параллельно примерно на стольких потоках, сколько ядер, и возвращается после завершения всех итераций. Цикл с `global().async` отправляет N независимых блоков без лимита и без способа их дождаться.

[Полный ответ →](https://irbisa.com/ru/topic/gcd-threading/#q-71524)

### 11. Какие локи вы используете в Swift (`NSLock`, `NSRecursiveLock`, `OSAllocatedUnfairLock`, `Mutex`) и почему не голый `os_unfair_lock`?

На iOS 18+ берите `Mutex`, на iOS 16–17 используйте `OSAllocatedUnfairLock`: оба владеют защищаемым состоянием и очень быстры. `NSLock` остаётся классическим запасным вариантом, `NSRecursiveLock` нужен для кода, который повторно входит в тот же лок, а голый `os_unfair_lock` в свойстве Swift не гарантирует адрес.

[Полный ответ →](https://irbisa.com/ru/topic/gcd-threading/#q-71525)

### 12. Что из этого потокобезопасно в Swift: `static let`, `lazy var`, `Array`, в который добавляют элементы, счётчик `Int`?

Только `static let`: глобальные переменные и статические хранимые свойства инициализируются лениво ровно один раз, даже если к ним одновременно впервые обращаются много потоков. `lazy var`, добавление в общий `Array` и `count += 1` превращаются в гонки данных, если это делают два потока одновременно.

[Полный ответ →](https://irbisa.com/ru/topic/gcd-threading/#q-71526)

### 13. Как выстроить цепочку работы через зависимости `Operation` и что происходит с зависимой операцией, если её зависимость отменили?

`b.addDependency(a)` не даёт `b` стать готовой, пока не завершится `a`, даже если они в разных очередях. Отменённая операция всё равно завершается, поэтому зависимые от неё стартуют; каждая операция должна сама проверять, не были ли отменены или не упали ли её зависимости.

[Полный ответ →](https://irbisa.com/ru/topic/gcd-threading/#q-71527)

### 14. Когда в кодовой базе на Swift 6 и async/await GCD всё ещё остаётся правильным инструментом?

Когда API просит очередь, когда работа блокирует поток и её нельзя сделать асинхронной, а также для таймеров и источников событий. Общее состояние и обычную фоновую работу отдают акторам и задачам; GCD остаётся на границах.

[Полный ответ →](https://irbisa.com/ru/topic/gcd-threading/#q-71528)

### 15. Как написать асинхронный подкласс `Operation` и почему очередь застревает, если `isFinished` не поддерживает KVO?

Переопределите `start()` вместо `main()`, сами отдавайте `isExecuting` и `isFinished` и отправляйте KVO-уведомления для обоих при каждом изменении. Очередь узнаёт о завершении операции только из KVO-изменения `isFinished`; без него операция остаётся в очереди навсегда.

[Полный ответ →](https://irbisa.com/ru/topic/gcd-threading/#q-71529)

### 16. С каким QoS выполняется блок, отправленный через `async`, как GCD решает проблему инверсии приоритетов и когда не может её решить?

По умолчанию `async` захватывает QoS отправляющего потока, и на очереди без собственного QoS блок выполняется с ним; очередь со своим QoS использует свой. Инверсия приоритетов (работа с высоким QoS ждёт работу с низким) решается автоматически, только если система знает, кого именно ждут.

[Полный ответ →](https://irbisa.com/ru/topic/gcd-threading/#q-71530)

### 17. Instruments показывает 60+ потоков, потому что у каждого объекта своя serial-очередь. Почему так происходит и как это исправляют target-очереди?

Независимые serial-очереди могут работать одновременно, каждая на своём потоке, и получить новый поток, даже когда все ядра заняты. Если направить их в одну serial target-очередь на подсистему, вся группа будет выполнять не больше одного блока за раз и не плодить потоки.

[Полный ответ →](https://irbisa.com/ru/topic/gcd-threading/#q-71531)

### 18. Совпадает ли `DispatchQueue.main.async` с выполнением на `@MainActor`, и что меняется при переводе GCD-кода на Swift Concurrency?

Во время выполнения оба варианта идут на главном потоке: executor главного актора и есть главная dispatch-очередь. Разница в компиляторе и во времени: `@MainActor` проверяется статически и из главного актора выполняется сразу, а `main.async` всегда ставит работу в очередь.

[Полный ответ →](https://irbisa.com/ru/topic/gcd-threading/#q-71532)
