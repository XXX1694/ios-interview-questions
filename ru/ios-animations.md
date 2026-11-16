# Анимации и Core Animation: вопросы на собеседовании

CALayer, неявные анимации, UIViewPropertyAnimator, переходы, PhaseAnimator, KeyframeAnimator. Senior · 20 вопросов.

**[Тренировать тему на irbisa.com →](https://irbisa.com/ru/topic/ios-animations/)** · [Все темы по iOS](../README.ru.md#все-темы)

### 1. Эта точка уезжает в x = 300 и остаётся там. Что выведет `print` через секунду, и почему перенос в 100 ничего не меняет на экране?

```swift
let dot = ballView.layer                 // ballView is centred at x = 40

let slide = CABasicAnimation(keyPath: "position.x")
slide.fromValue = 40
slide.toValue = 300
slide.duration = 0.5
slide.fillMode = .forwards
slide.isRemovedOnCompletion = false
dot.add(slide, forKey: "slide")

DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
  print(dot.position.x)
  dot.position.x = 100                   // nothing moves on screen
}
```

Выведет 40. Анимация изменила только presentation layer, а `fillMode = .forwards` вместе с `isRemovedOnCompletion = false` замораживает эту картинку поверх модели, которая никуда не двигалась, и пока замороженная анимация висит на слое, она перекрывает любое новое значение модели.

[Полный ответ →](https://irbisa.com/ru/topic/ios-animations/#q-67218)

### 2. Почему изменение свойства у отдельного `CALayer` анимируется само, а то же изменение у слоя, на котором стоит вью, просто прыгает?

Оба изменения проходят один и тот же поиск action и получают разные ответы: у отдельного слоя нет делегата, и поиск доходит до анимации по умолчанию на 0,25 с, а слой вью спрашивает свой `UIView`, который возвращает `NSNull` (без анимации), если изменение сделано не внутри блока анимации.

[Полный ответ →](https://irbisa.com/ru/topic/ios-animations/#q-67202)

### 3. Что такое `CATransaction`, и как запретить анимацию для одного изменения слоя?

`CATransaction` представляет собой пачку, в которой изменения слоёв уходят в render server: она задаёт им длительность анимации, тайминг-функцию и completion и коммитит их вместе. Чтобы одно изменение не анимировалось, оберните его в транзакцию с `setDisableActions(true)`.

[Полный ответ →](https://irbisa.com/ru/topic/ios-animations/#q-67203)

### 4. Где будет нарисована карточка с трансформом `a` и с трансформом `b`, и что каждый раз выведет `print`?

```swift
// card is 100 x 100, centred at (200, 200), with an identity transform
let a = CGAffineTransform(translationX: 100, y: 0).rotated(by: .pi / 2)
let b = CGAffineTransform(rotationAngle: .pi / 2).translatedBy(x: 100, y: 0)

card.transform = a
print(card.center)      // what prints, and where is the card drawn?

card.transform = b
print(card.center)      // and now?
```

С `a` карточка рисуется на 100 pt правее, с центром в (300, 200); с `b` рисуется на 100 pt ниже, в (200, 300); в обоих случаях она повёрнута на 90°. `print` оба раза выводит (200, 200), потому что `transform` меняет только то, как вью рисуется, а не её `center` или `bounds`.

[Полный ответ →](https://irbisa.com/ru/topic/ios-animations/#q-67219)

### 5. Какие подклассы `CALayer` вы используете, и почему анимированное кольцо прогресса лучше делать на `CAShapeLayer`, а не через `draw(_:)`?

Специализированные слои (`CAShapeLayer`, `CAGradientLayer`, `CATextLayer`, `CAReplicatorLayer`, `CAEmitterLayer`) описывают содержимое, которое render server рисует сам. `draw(_:)` же даёт вью битмап, который CPU перерисовывает и заново загружает при каждом изменении.

[Полный ответ →](https://irbisa.com/ru/topic/ios-animations/#q-67213)

### 6. Что такое `CADisplayLink`, когда он нужен вместо Core Animation, и как получить от него 120 Гц на iPhone с ProMotion?

`CADisplayLink` представляет собой таймер, синхронизированный с дисплеем: он вызывает вас раз в кадр, чтобы вы сами посчитали значение для этого кадра. Используйте его, только когда Core Animation не может интерполировать нужное за вас (физика, живая осциллограмма, свой рендерер).

[Полный ответ →](https://irbisa.com/ru/topic/ios-animations/#q-67212)

### 7. Что даёт `UIViewPropertyAnimator` по сравнению с `UIView.animate`, и как управлять им из pan-жеста?

`UIView.animate` работает по принципу «запустил и забыл»; `UIViewPropertyAnimator` представляет собой объект, который вы храните, поэтому его можно поставить на паузу, скрабить `fractionComplete`, развернуть, добавлять анимации и completion на ходу и завершить с новым таймингом, а это то, что нужно интерактивному и прерываемому UI.

[Полный ответ →](https://irbisa.com/ru/topic/ios-animations/#q-67205)

### 8. Найдите баг: по нажатию на Filters панель должна плавно выезжать, но она сразу прыгает на полную высоту без анимации.

```swift
final class FiltersViewController: UIViewController {
  let panel = UIView()
  private var panelHeight: NSLayoutConstraint!   // panel.heightAnchor, starts at 0
  private var isExpanded = false

  @objc func toggleFilters() {
    isExpanded.toggle()
    UIView.animate(withDuration: 0.3) {
      self.panelHeight.constant = self.isExpanded ? 320 : 0
    }
  }
}
```

Изменение `constant` само по себе ничего не двигает: оно только помечает layout как требующий обновления. Frame меняются при следующем проходе layout, уже после того как блок анимации вернулся, поэтому панель прыгает на новую высоту.

[Полный ответ →](https://irbisa.com/ru/topic/ios-animations/#q-67221)

### 9. Как реализовать кастомный переход между view controller, и кто добавляет и убирает вью в контейнере?

Верните объект, реализующий `UIViewControllerAnimatedTransitioning`, из делегата навигации или transitioning delegate. UIKit вызывает его `animateTransition(using:)`: вы добавляете входящую вью в `containerView`, анимируете и вызываете `completeTransition(_:)`; уходящую вью UIKit убирает сам.

[Полный ответ →](https://irbisa.com/ru/topic/ios-animations/#q-67207)

### 10. Когда в SwiftUI на самом деле срабатывает `.transition`, и почему вью иногда просто появляется без него?

Transition срабатывает, только когда вью вставляется в иерархию или удаляется из неё (меняется её identity), и это изменение анимировано. Без анимации вью просто появляется; без смены identity переходить не из чего.

[Полный ответ →](https://irbisa.com/ru/topic/ios-animations/#q-67208)

### 11. Как выполнить код, когда анимация в SwiftUI закончилась, и как запустить одну анимацию после другой?

С iOS 17 `withAnimation(_:completionCriteria:_:completion:)` вызывает замыкание, когда анимация завершена, а цепочку строят, запуская следующий `withAnimation` внутри него. До этого был только вариант с задержкой, равной длительности, который ломается на пружинах и прерываниях.

[Полный ответ →](https://irbisa.com/ru/topic/ios-animations/#q-67220)

### 12. Чем `phaseAnimator` отличается от `keyframeAnimator`, и когда достаточно анимации с `repeatForever`?

`phaseAnimator` переводит все свойства по списку состояний вместе; `keyframeAnimator` даёт каждому свойству свою шкалу кейфреймов, и шкалы идут параллельно. `repeatForever` даёт одну зацикленную анимацию между двумя значениями: годится для пульсации, но не для последовательности.

[Полный ответ →](https://irbisa.com/ru/topic/ios-animations/#q-67210)

### 13. Этот `Ring` перескакивает к новому прогрессу вместо анимации: почему, и при чём здесь `Animatable` и `animatableData`?

```swift
struct Ring: Shape {
  var progress: Double                    // 0...1

  func path(in rect: CGRect) -> Path {
    var path = Path()
    path.addArc(center: CGPoint(x: rect.midX, y: rect.midY), radius: rect.width / 2,
                startAngle: .degrees(-90), endAngle: .degrees(-90 + 360 * progress),
                clockwise: false)
    return path
  }
}

// Ring(progress: done).stroke(.teal, lineWidth: 8)
//   .animation(.easeOut(duration: 0.6), value: done)
```

SwiftUI интерполирует только то, что тип отдаёт через `Animatable.animatableData`. `Shape` реализует `Animatable`, но по умолчанию его `animatableData` имеет тип `EmptyAnimatableData`, поэтому `progress` никогда не интерполируется, и кольцо перерисовывается один раз, сразу с конечным значением.

[Полный ответ →](https://irbisa.com/ru/topic/ios-animations/#q-67211)

### 14. Чем неявные анимации Core Animation отличаются от явных, и что такое `CABasicAnimation`, `CAKeyframeAnimation` и `CAAnimationGroup`?

Неявную анимацию Core Animation создаёт сам, когда вы меняете анимируемое свойство слоя; явная представляет собой `CAAnimation`, который вы собираете и добавляете через `layer.add(_:forKey:)`. Явная анимация меняет только presentation layer, а выставить значение в модели остаётся вашей задачей.

[Полный ответ →](https://irbisa.com/ru/topic/ios-animations/#q-67217)

### 15. Чем model layer отличается от presentation layer, и почему по вью нельзя тапнуть, пока она анимируется?

Model layer хранит значения, которые вы установили (во время анимации это уже конечные), а `presentation()` возвращает копию с тем, что на экране в этом кадре. UIKit делает hit testing по модели и по умолчанию отключает взаимодействие с анимируемыми вью, поэтому тапы по движущимся пикселям не срабатывают.

[Полный ответ →](https://irbisa.com/ru/topic/ios-animations/#q-67201)

### 16. Как работает `matchedGeometryEffect`, и почему карточка, которая должна разворачиваться в экран деталей, вместо этого просто гаснет и прыгает?

`matchedGeometryEffect` не морфит содержимое: он берёт frame вью, помеченной как источник, и применяет его к другой вью с тем же id и namespace. Анимация получается, только когда обе вью существуют в одной анимированной транзакции, в одном namespace и источник в каждый момент ровно один.

[Полный ответ →](https://irbisa.com/ru/topic/ios-animations/#q-67209)

### 17. Как передать скорость жеста в момент отпускания в пружинную анимацию, и что означают `duration` и `bounce` у пружины?

Передайте скорость пальца как начальную скорость пружины, и тогда движение продолжится, а не начнётся заново из состояния покоя. `duration` задаёт воспринимаемое время успокоения, `bounce` определяет величину перелёта, и, в отличие от массы, жёсткости и затухания, их можно настраивать независимо.

[Полный ответ →](https://irbisa.com/ru/topic/ios-animations/#q-67214)

### 18. Как сделать кастомный переход интерактивным, и что должно произойти, если пользователь отменил его на полпути?

Верните из делегата interaction controller (обычно `UIPercentDrivenInteractiveTransition`) и управляйте им из жеста через `update(_:)`, а затем `finish()` или `cancel()`. При отмене аниматор должен откатить свои изменения и сообщить `completeTransition(false)`, иначе экран останется замороженным.

[Полный ответ →](https://irbisa.com/ru/topic/ios-animations/#q-67206)

### 19. Какие свойства слоя дёшево анимировать, и что заставляет render server переходить на offscreen rendering?

Анимировать `transform`, `opacity`, `position` или `bounds` уже отрисованного слоя дёшево: приложение коммитит один раз, а render server интерполирует каждый кадр. Дорого становится, когда каждому кадру нужен offscreen-проход на GPU или новые пиксели от CPU.

[Полный ответ →](https://irbisa.com/ru/topic/ios-animations/#q-67204)

### 20. Если анимация дёргается на реальном iPhone, как понять, кто опаздывает: commit вашего приложения или render server?

Профилируйте release-сборку на устройстве шаблоном Animation Hitches в Instruments: он относит каждый hitch либо к фазе commit в вашем процессе, либо к фазе render в render server. Опоздание на commit означает работу на главном потоке, а опоздание на render означает, что дерево слоёв дорого рисовать.

[Полный ответ →](https://irbisa.com/ru/topic/ios-animations/#q-67215)
