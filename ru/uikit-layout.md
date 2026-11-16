# Auto Layout и UIKit-вёрстка: вопросы на собеседовании

Constraint-ы, intrinsic content size, UIStackView, frame-ы, safe area. Junior · 25 вопросов.

**[Тренировать тему на irbisa.com →](https://irbisa.com/ru/topic/uikit-layout/)** · [Все темы по iOS](../README.ru.md#все-темы)

### 1. Что такое Auto Layout и чем он отличается от вёрстки на frame?

Auto Layout строит вёрстку на constraint-ах: вы описываете отношения между view («на 16 pt ниже заголовка, той же ширины, что карточка»), а решатель на каждом проходе layout превращает их во frame-ы. При вёрстке на frame каждый прямоугольник вы считаете и задаёте сами.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-3101)

### 2. Какое уравнение выражает constraint в Auto Layout и что такое layout anchors?

Любой constraint представляет собой линейное уравнение `item1.attribute = multiplier × item2.attribute + constant`, где вместо равенства может стоять `≤` или `≥`. Layout anchors (`topAnchor`, `widthAnchor`, …) дают типобезопасный способ записать это уравнение.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-63115)

### 3. Сколько constraint-ов нужно view, чтобы её layout был однозначным?

Столько, чтобы зафиксировать четыре значения (позицию x и ширину, позицию y и высоту); обычно это четыре constraint-а, по два на ось. View с intrinsic content size, например `UILabel`, сама задаёт свой размер, поэтому ей хватает двух constraint-ов на позицию.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-63116)

### 4. Вы задали constraint-ы для view, созданной в коде, но она не появляется там, где ожидалось, а в консоли список сломанных constraint-ов. Что здесь не так?

```swift
override func viewDidLoad() {
  super.viewDidLoad()
  let card = UIView()
  card.backgroundColor = .systemBlue
  view.addSubview(card)
  NSLayoutConstraint.activate([
    card.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 16),
    card.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
    card.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16),
    card.heightAnchor.constraint(equalToConstant: 120),
  ])
}
```

У `card` по-прежнему `translatesAutoresizingMaskIntoConstraints` равно `true`, поэтому UIKit превращает её frame (`.zero`) и autoresizing mask в constraint-ы, которые противоречат вашим. Нужно выставить `card.translatesAutoresizingMaskIntoConstraints = false` до активации constraint-ов.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-63101)

### 5. Этот код настройки падает с «Unable to activate constraint ... because they have no common ancestor». Что здесь сделано не в том порядке?

```swift
let badge = UILabel()
badge.text = "NEW"
badge.translatesAutoresizingMaskIntoConstraints = false

NSLayoutConstraint.activate([
  badge.topAnchor.constraint(equalTo: card.topAnchor, constant: 8),
  badge.trailingAnchor.constraint(equalTo: card.trailingAnchor, constant: -8),
])

card.addSubview(badge)
```

Constraint-ы активируются до того, как `badge` попала в иерархию view. Активация устанавливает constraint на ближайшего общего предка обеих view, а у view без superview его нет, поэтому UIKit бросает исключение. Сначала `card.addSubview(badge)`, потом активация.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-63108)

### 6. Что такое safe area и как в UIKit удержать контент внутри неё?

Под safe area понимают часть view, которую не перекрывают статус-бар, navigation и tab bar, home indicator или Dynamic Island. Если привязать контент к `view.safeAreaLayoutGuide`, он останется видимым на любом устройстве и в любой ориентации.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-3104)

### 7. Почему constraint-ы ставят к `leadingAnchor` и `trailingAnchor`, а не к `leftAnchor` и `rightAnchor`?

Leading и trailing следуют направлению чтения, поэтому в арабском и иврите layout отзеркаливается автоматически; left и right обозначают физические стороны и не переворачиваются никогда.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-63106)

### 8. Что такое `UIStackView` и какие у него плюсы и минусы?

`UIStackView` выстраивает свои subview в строку или столбец и сам создаёт constraint-ы между ними: вы задаёте `axis`, `distribution`, `alignment` и `spacing`, а constraint-ы ставите только самому стеку.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-3103)

### 9. В горизонтальном `UIStackView` все элементы получились одной ширины, а иконка растянута на всю высоту строки. Какие два свойства заданы неверно?

`distribution` задаёт размеры вдоль оси, а `alignment` отвечает за расположение поперёк неё: одинаковую ширину даёт `distribution = .fillEqually`, растянутую иконку даёт `alignment = .fill` по умолчанию.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-63107)

### 10. Что такое intrinsic content size и как content hugging и compression resistance решают, какая view уступит?

Intrinsic content size означает размер, который view хочет иметь по своему контенту: текст лейбла, заголовок кнопки с отступами. Content hugging сопротивляется росту сверх него, compression resistance сопротивляется сжатию меньше него, и первой уступает view с более низким приоритетом.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-3102)

### 11. Что делают `@IBDesignable` и `@IBInspectable` и чем команды заменяют live rendering сегодня?

`@IBInspectable` выводит свойство кастомной view в Attributes inspector в Interface Builder, а `@IBDesignable` заставляет Interface Builder собрать view и отрисовать её на канвасе. Сегодня UIKit-view обычно смотрят через макрос `#Preview` (Xcode 15+).

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-63119)

### 12. Что такое приоритеты constraint-ов и почему для constraint-а, который «обязан» выполняться, ставят 999, а не 1000?

Приоритет от 1 до 1000 говорит, насколько сильно решатель старается выполнить constraint: 1000 (`.required`) обязан выполниться, всё ниже считается необязательным: решатель выполняет его настолько близко, насколько может, и без записи в лог уступает при конфликте с более сильным. 999 означает «как можно сильнее, но может уступить».

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-63117)

### 13. Этот бейдж после изменения `count` рисует новое число, но сохраняет старую ширину. Чего не хватает?

```swift
final class BadgeView: UIView {
  var count = 0 {
    didSet { setNeedsDisplay() }
  }

  override var intrinsicContentSize: CGSize {
    let text = "\(count)" as NSString
    let font = UIFont.preferredFont(forTextStyle: .caption1)
    let size = text.size(withAttributes: [.font: font])
    return CGSize(width: ceil(size.width) + 16, height: ceil(size.height) + 8)
  }

  override func draw(_ rect: CGRect) {
    // draws the pill and the number
  }
}
```

В `didSet` нет вызова `invalidateIntrinsicContentSize()`. Auto Layout кэширует intrinsic size и перечитывает его только после этого вызова; `setNeedsDisplay()` лишь перерисовывает тот же прямоугольник.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-63109)

### 14. Чем отличается привязка к краю view, к её `layoutMarginsGuide` и к её `readableContentGuide`?

Край совпадает с буквальными bounds view, `layoutMarginsGuide` даёт bounds минус margins самой view, а `readableContentGuide` дополнительно ограничивает ширину, чтобы строки текста на широких экранах оставались удобной длины.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-63105)

### 15. Чем отличаются `setNeedsLayout()`, `layoutIfNeeded()` и `layoutSubviews()`?

`setNeedsLayout()` помечает view как требующую layout в следующем цикле обновления, `layoutIfNeeded()` выполняет отложенный layout прямо сейчас, а `layoutSubviews()` является методом, который UIKit вызывает во время этого layout: первые два вызываете вы, третий вы переопределяете.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-3106)

### 16. Layout должен меняться при переключении состояния. Вы меняете `constant`, переключаете активные constraint-ы или меняете приоритет?

Меняйте `constant`, когда меняется только число (это чисто анимируется), переключайте наборы активных constraint-ов, когда меняются сами отношения, и меняйте приоритеты, когда два конкурирующих constraint-а должны побеждать по очереди.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-63104)

### 17. Найдите баг: панель должна плавно выезжать снизу, но прыгает на новое место без анимации.

```swift
final class PlayerViewController: UIViewController {
  private let panel = UIView()
  // activated in viewDidLoad with leading, trailing and height 300
  private lazy var panelBottom = panel.bottomAnchor.constraint(
    equalTo: view.bottomAnchor, constant: 300)  // starts off-screen

  func showPanel() {
    panelBottom.constant = 0
    UIView.animate(withDuration: 0.3) {
      self.panel.layoutIfNeeded()
    }
  }
}
```

`layoutIfNeeded()` вызывается у `panel`, но изменённый constraint задаёт положение `panel` внутри `view`, поэтому блок ничего не двигает. Панель прыгает на следующем проходе layout, уже вне анимации, вместо того чтобы плавно выехать.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-63121)

### 18. В консоли появляется «Unable to simultaneously satisfy constraints». Как найти виновника?

Прочитайте лог: в нём перечислены constraint-ы, которые не могут выполниться вместе, и назван сломанный. Задайте им идентификаторы, затем найдите виновника через breakpoint на `UIViewAlertForUnsatisfiableConstraints` и View Debugger.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-3105)

### 19. Что должно выполняться, чтобы ячейка таблицы сама подстраивала высоту под контент?

Нужна непрерывная цепочка вертикальных constraint-ов от верха до низа `contentView`, чтобы Auto Layout мог вычислить высоту; таблица при этом использует `UITableView.automaticDimension`, и это уже значение по умолчанию.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-63102)

### 20. Как разложить контент внутри `UIScrollView` на Auto Layout так, чтобы он действительно прокручивался?

Scroll view вычисляет размер контента по его constraint-ам, поэтому контент нужно привязать ко всем четырём краям `contentLayoutGuide`, а ширину взять от `frameLayoutGuide`.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-63103)

### 21. Текстовое поле внизу формы оказывается под клавиатурой. Как его правильно поднять в современном UIKit и в чём ошибался старый код на нотификациях?

Привяжите низ поля к `view.keyboardLayoutGuide.topAnchor`. С iOS 15 этот guide отслеживает клавиатуру в координатах вашей view и анимируется вместе с ней, так что один constraint заменяет весь код на нотификациях.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-63111)

### 22. Что такое size classes и как переключаться между двухколоночной вёрсткой на iPad и одноколоночной на iPhone?

Size classes дают грубые трейты, `.compact` или `.regular`, для ширины и высоты, которые реально достались вашей view. Ориентируйтесь на `traitCollection.horizontalSizeClass`, а не на устройство: приложение на iPad в Split View или в узком окне получает compact-ширину.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-63113)

### 23. Как в UIKit-вёрстке обрабатывать поворот устройства?

В основном никак: constraint-ы относительно родителя и safe area сами пересчитываются под новый размер. Код нужен, только когда меняется структура, и размещают его в `viewWillTransition(to:with:)` или в обработчике смены size class, а не в нотификациях об ориентации устройства.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-63118)

### 24. Один экран загружает CPU на 100% и зависает, а `layoutSubviews()` вызывается снова и снова. Что вызывает петлю обратной связи в Auto Layout?

Что-то внутри прохода layout снова инвалидирует layout: constraint-ы меняются на основе геометрии, которую только что вычислил тот же проход, и каждый проход планирует следующий.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-63112)

### 25. Прокручиваемый список роняет кадры, а Instruments показывает время в Auto Layout. Что вы проверите и что поменяете?

Выясните, сколько работы layout делает каждая ячейка на шаге прокрутки, и сократите её: меньше view и constraint-ов в ячейке, никаких изменений constraint-ов при переиспользовании, а для немногих «горячих» ячеек, которые всё ещё слишком дороги, используйте frame-ы.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-layout/#q-63120)
