# Основы UIKit: вопросы на собеседовании

UIView, жизненный цикл UIViewController, иерархия view, обработка касаний. Junior · 25 вопросов.

**[Тренировать тему на irbisa.com →](https://irbisa.com/ru/topic/uikit-fundamentals/)** · [Все темы по iOS](../README.ru.md#все-темы)

### 1. Что такое `UIWindow`, с чего начинается иерархия вью и чем окно отличается от обычного `UIView`?

`UIWindow` служит корнем иерархии вью сцены: подкласс `UIView` без superview, который держит `rootViewController`, получает события от `UIApplication` и передаёт их вью внутри себя.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-3001)

### 2. Расскажите о жизненном цикле `UIViewController`: какие колбэки вызываются один раз, а какие при каждом появлении, и где начинать загрузку данных?

`loadView` и `viewDidLoad` вызываются один раз на экземпляр контроллера; `viewWillAppear`, `viewIsAppearing`, `viewDidAppear` и два колбэка исчезновения вызываются каждый раз, когда экран показывается или скрывается.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-3002)

### 3. Что такое `loadView`, когда его переопределяют и чем он отличается от `viewDidLoad`?

`loadView` создаёт корневую вью контроллера и присваивает её в `view`; `viewDidLoad` вызывается сразу после него и настраивает уже существующую вью. Переопределять `loadView` нужно только чтобы сделать корневой вью свой класс.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-63015)

### 4. Чем отличаются `frame` и `bounds` и когда `bounds.origin` не равен нулю?

`frame` задаёт положение и размер вью в системе координат superview; `bounds` описывает тот же прямоугольник в собственных координатах вью. `bounds.origin` не равен нулю, когда вью показывает смещённую часть своего содержимого: у scroll view `bounds.origin` совпадает с его `contentOffset`.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-3003)

### 5. Каковы плюсы и минусы сторибордов, XIB и вёрстки интерфейса в коде?

Все три способа дают одни и те же вью UIKit: сториборды и XIB предлагают визуальный редактор, а код даёт читаемые диффы и заметно меньше конфликтов при слиянии. Большинство команд сейчас делают новые экраны в коде или на SwiftUI.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-3006)

### 6. Чем отличаются `@IBOutlet` и `@IBAction` и как target-action связывает контрол с вашим кодом?

`@IBOutlet` помечает свойство, которое Interface Builder заполняет ссылкой на вью, чтобы код мог её читать и менять; `@IBAction` помечает метод, который Interface Builder привязывает к событию контрола, чтобы контрол мог вызвать ваш код. Второе и есть target-action.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-63016)

### 7. Как работает `UINavigationController` и чем push отличается от present?

`UINavigationController` управляет стеком view controller'ов: `push` кладёт контроллер наверх с кнопкой «Назад» и свайпом назад, `pop` снимает его. `present` показывает контроллер модально поверх текущего, вне всякого стека, пока его не закроют.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-63003)

### 8. Как работает `UITableView`: что делают data source и delegate и что такое переиспользование ячеек?

Таблица спрашивает у data source, сколько строк, и ячейку для каждой видимой строки, а delegate сообщает о выборе, размерах и прокрутке. Существуют только ячейки на экране; ушедшие за край попадают в очередь переиспользования и возвращаются из `dequeueReusableCell(withIdentifier:for:)`.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-3005)

### 9. Как устроен `UIScrollView` и за что в нём отвечают свойства `contentSize`, `contentOffset` и `contentInset`?

Scroll view представляет собой вью, у которой `bounds.origin` сдвигается, когда пользователь тянет пальцем: `contentSize` задаёт размер прокручиваемого содержимого, `contentOffset` показывает, какая его точка находится в левом верхнем углу, а `contentInset` добавляет дополнительное прокручиваемое пространство вокруг.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-63017)

### 10. Чем отличаются `UIImage` и `UIImageView`, а также `.scaleAspectFit` и `.scaleAspectFill`?

`UIImage` хранит данные изображения (пиксели и масштаб), которые не находятся на экране; `UIImageView` является вью, которая показывает их согласно `contentMode`. `.scaleAspectFit` показывает изображение целиком с пустыми полями, `.scaleAspectFill` заполняет вью и обрезает лишнее.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-63018)

### 11. Как скруглить углы вью и добавить тень и почему они конфликтуют?

Скругляйте через `layer.cornerRadius` и `clipsToBounds`, а тень задавайте через `layer.shadowOpacity`/`shadowRadius`/`shadowOffset`; обрезка отсекает тень, поэтому нужны две вью: внешняя с тенью и внутренняя, которая обрезает.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-63008)

### 12. Этот лейбл обновляется из колбэка `URLSession`, и приложение потом падает в случайных местах. Что не так и как это исправить?

```swift
func loadProfile() {
  URLSession.shared.dataTask(with: profileURL) { data, _, _ in
    guard let data else { return }
    let profile = try? JSONDecoder().decode(Profile.self, from: data)
    self.nameLabel.text = profile?.name
    self.view.setNeedsLayout()
  }.resume()
}
```

Completion handler у `dataTask` выполняется на фоновой очереди, а UIKit можно использовать только на главном потоке. Обновляйте UI на главном потоке: оберните его в `DispatchQueue.main.async` или используйте async/await из main-actor-контекста.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-63009)

### 13. Как поддержать тёмную тему в UIKit и почему цвет рамки слоя не переключается вместе с ней?

Используйте динамические цвета (системные семантические вроде `.label` и `.systemBackground` или цвета из asset catalog с тёмным вариантом), и UIKit сам пересчитывает их при каждой смене оформления. `CGColor`, заданный слою, вычисляется один раз, поэтому его нужно выставлять заново при смене трейта.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-63012)

### 14. Как анимировать вью в UIKit и как анимировать изменение констрейнта Auto Layout?

Поместите изменения свойств внутрь `UIView.animate(withDuration:animations:)`. Для Auto Layout сначала измените `constant` констрейнта, затем внутри блока анимации вызовите `layoutIfNeeded()` у superview: анимируется именно проход layout.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-63005)

### 15. Что такое `viewIsAppearing(_:)` и почему для настройки, зависящей от геометрии, он лучше, чем `viewWillAppear` или `viewDidLayoutSubviews`?

`viewIsAppearing` вызывается один раз за появление: после того как вью оказалась в иерархии с окончательным размером и трейтами, но до того как она на экране. `viewWillAppear` слишком ранний для геометрии, а `viewDidLayoutSubviews` срабатывает на каждом проходе layout.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-63011)

### 16. Найдите баг: после нескольких заходов на экран корзины одно изменение корзины попадает в аналитику несколько раз.

```swift
final class CartViewController: UIViewController {
  override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    NotificationCenter.default.addObserver(
      self, selector: #selector(cartDidChange),
      name: .cartDidChange, object: nil)
    reloadBadge()
  }

  @objc private func cartDidChange() {
    reloadBadge()
    analytics.log("cart_changed")
  }
}
```

Наблюдатель добавляется во `viewWillAppear`, который вызывается при каждом появлении экрана, и нигде не удаляется. `NotificationCenter` не отсеивает дубликаты регистраций, поэтому после трёх заходов одно `.cartDidChange` вызывает `cartDidChange()` три раза: три события аналитики, три перезагрузки.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-63024)

### 17. Как передать данные вперёд, в следующий view controller, и назад, в предыдущий?

Вперёд: отдайте следующему контроллеру всё нужное до его показа, лучше всего через `init`. Назад: дочерний контроллер сообщает через колбэк, который задал родитель (замыкание или weak-делегат), но никогда не через прямую ссылку на тип родителя.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-63019)

### 18. Что такое containment view controller'ов и какие вызовы нужны, чтобы добавить и убрать дочерний контроллер?

Containment позволяет одному view controller'у встроить вью другого в свою, при этом дочерний остаётся полноценным контроллером со своим жизненным циклом. Добавляют через `addChild`, затем добавляют его вью, затем вызывают `didMove(toParent:)`; убирают в зеркальном порядке.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-63020)

### 19. Что такое responder chain и как `hitTest(_:with:)` определяет, какая вью получит касание?

UIKit сначала выполняет hit-testing от окна вниз, чтобы найти самую глубокую вью под пальцем, и доставляет касание ей; всё, что эта вью не обработала, поднимается по responder chain: superview, view controller, окно, приложение.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-3004)

### 20. Как работают gesture recognizers и как сделать, чтобы одиночный тап ждал двойной, а pan работал вместе со scroll view?

Gesture recognizer представляет собой машину состояний, прикреплённую к вью: он следит за касаниями и вызывает свой action, когда распознаёт жест. По умолчанию побеждает только один рекогнайзер; `require(toFail:)` задаёт порядок, а метод делегата `shouldRecognizeSimultaneouslyWith` позволяет двум работать одновременно.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-63010)

### 21. Чем отличаются `UIView` и `CALayer` и когда работают со слоем напрямую?

Каждый `UIView` опирается на `CALayer`: вью отвечает за касания, layout, responder chain и accessibility, а слой хранит содержимое, которое рисует и анимирует Core Animation. Слои легче, но не получают касаний.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-63002)

### 22. Чем отличаются `setNeedsLayout` и `setNeedsDisplay` и когда вызывается `draw(_:)`?

`setNeedsLayout` помечает вью для нового прохода layout, и вызывается `layoutSubviews`; `setNeedsDisplay` помечает её содержимое устаревшим, и вызывается `draw(_:)` для перерисовки. Ни один не действует сразу: оба откладываются до конца текущей итерации run loop.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-63021)

### 23. Как работает главный run loop и почему `Timer` перестаёт срабатывать, пока пользователь скроллит?

Главный run loop служит циклом событий главного потока: он обрабатывает касания, таймеры и блоки из очереди, затем коммитит layout и отрисовку. Запланированный `Timer` срабатывает только в режиме `.default`, а при прокрутке цикл работает в режиме `.tracking`, поэтому таймер ждёт.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-63022)

### 24. Этот контроллер закрывают через 3 секунды после открытия. Что напечатается, когда и есть ли здесь утечка памяти?

```swift
final class DetailViewController: UIViewController {
  private let statusLabel = UILabel()

  override func viewDidLoad() {
    super.viewDidLoad()
    DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
      self.statusLabel.text = "Loaded"
      print("closure ran")
    }
  }

  deinit { print("deinit") }
}
```

Через 3 секунды ничего не напечатается. Через 5 секунд замыкание выполнится и напечатает «closure ran», затем отпустит `self`, и напечатается «deinit». Замыкание держит закрытый контроллер на 2 секунды дольше, но цикла сильных ссылок нет, так что это не утечка.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-63023)

### 25. Найдите баг: при быстром скролле в строках на мгновение появляется аватар другого пользователя, а иногда чужой так и остаётся.

```swift
override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
  let cell = tableView.dequeueReusableCell(withIdentifier: "Avatar", for: indexPath) as! AvatarCell
  let user = users[indexPath.row]
  cell.nameLabel.text = user.name
  Task {
    let image = await imageLoader.image(for: user.avatarURL)
    cell.avatarView.image = image
  }
  return cell
}
```

`Task` захватывает ячейку, а не строку. Ячейки переиспользуются, поэтому, когда медленная загрузка завершается, ячейка может уже показывать другого пользователя и получает чужой аватар; к тому же старое изображение видно, пока не придёт новое.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-fundamentals/#q-63025)
