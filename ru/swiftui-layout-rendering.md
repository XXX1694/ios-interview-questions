# SwiftUI: layout, идентичность и отрисовка: вопросы на собеседовании

Протокол Layout, GeometryReader, PreferenceKey, идентичность view, Canvas, ленивые контейнеры. Senior · 21 вопрос.

**[Тренировать тему на irbisa.com →](https://irbisa.com/ru/topic/swiftui-layout-rendering/)** · [Все темы по iOS](../README.ru.md#все-темы)

### 1. В этом горизонтальном `ScrollView` `Spacer` схлопывается, и карточки стоят вплотную: что предлагает `ScrollView` и как это исправить?

```swift
ScrollView(.horizontal) {
  HStack {
    CardView(card: first)
    Spacer()
    CardView(card: second)
  }
}
```

Горизонтальный `ScrollView` предлагает своему контенту ширину `nil`, то есть «назови свою идеальную ширину». `Spacer` в ответ на вопрос об идеальном размере возвращает свою минимальную длину, поэтому лишнего места, которое он мог бы занять, просто нет.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-layout-rendering/#q-69201)

### 2. Когда стоит использовать `GeometryReader` и что он делает с layout вокруг себя?

`GeometryReader` передаёт в замыкание предложенный ему размер, но забирает всё это место себе и прижимает контент к верхнему левому углу, поэтому как обычный ребёнок он расталкивает соседей. Используйте его внутри `background` или `overlay` либо возьмите более новый API.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-layout-rendering/#q-69203)

### 3. Что такое `PreferenceKey` и как ребёнок передаёт значение вверх своим предкам?

`PreferenceKey` представляет собой типизированный канал, по которому значения идут от потомков вверх по дереву вью: дети задают его через `.preference(key:value:)`, SwiftUI сливает значения всех детей функцией `reduce` ключа, а предок читает результат в `onPreferenceChange`.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-layout-rendering/#q-69219)

### 4. Как `ViewThatFits` выбирает, какого ребёнка показать, и почему это часто лучше проверки `horizontalSizeClass`?

`ViewThatFits` по очереди спрашивает у каждого ребёнка его идеальный размер по указанным осям и показывает первого, кто помещается в предложенное ему место. Если не помещается никто, показывается последний ребёнок.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-layout-rendering/#q-69204)

### 5. Как обернуть UIKit-вью в `UIViewRepresentable`, встроить SwiftUI в UIKit через `UIHostingController` и как каждая сторона узнаёт размер другой?

`UIViewRepresentable` оборачивает UIKit-вью для SwiftUI, а `UIHostingController` оборачивает SwiftUI-вью в вью-контроллер. Размер по умолчанию переходит границу через `intrinsicContentSize`, в SwiftUI через `sizeThatFits(_:uiView:context:)`, а в Auto Layout через `sizingOptions`.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-layout-rendering/#q-69222)

### 6. Как работают alignment guides и как с помощью собственного выравнивания выстроить в линию вью из разных строк?

Стек спрашивает у каждого ребёнка одно значение, направляющую выравнивания (alignment guide), и сдвигает детей так, чтобы эти значения совпали. `alignmentGuide(_:computeValue:)` переопределяет его для одного ребёнка, а собственный `AlignmentID` ставит на одну линию вью из разных строк.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-layout-rendering/#q-69205)

### 7. Что такое identity вью в SwiftUI (структурная и явная) и что происходит, когда identity вью меняется?

По identity SwiftUI отличает «ту же вью, но обновлённую» от «другой вью»: структурная identity берётся из типа вью и её позиции в `body`, а явная из `.id(_:)` и id в `ForEach`. Когда она меняется, старая вью уничтожается, а новая начинает с чистого состояния.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-layout-rendering/#q-69207)

### 8. Пользователь выбирает другой элемент в сайдбаре, а этот редактор по-прежнему показывает заголовок предыдущего: почему и как это исправить?

```swift
struct DetailEditor: View {
  let item: Item
  @State private var draft: String

  init(item: Item) {
    self.item = item
    _draft = State(initialValue: item.title)
  }

  var body: some View {
    TextField("Title", text: $draft)
  }
}
```

SwiftUI хранит `@State` по identity вью и использует начальное значение только один раз. При выборе другого элемента структура `DetailEditor` пересоздаётся на том же месте, identity не меняется, `State(initialValue:)` игнорируется, и в `draft` остаётся старый заголовок.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-layout-rendering/#q-69208)

### 9. SwiftUI предупреждает, что ID встречается в `ForEach` несколько раз, и строки начинают путать состояние: что сломалось и где это чинить?

Два элемента отдают `ForEach` один и тот же id, и SwiftUI не может сопоставить старые строки с новыми: состояние, фокус и анимации привязываются не к той строке, а один из дубликатов может вообще не отрисоваться. Чинить нужно в данных: id должны быть уникальными и стабильными.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-layout-rendering/#q-69209)

### 10. Каждый раз, когда `isHighlighted` меняется, счётчик в этой строке сбрасывается в ноль: что не так с этим хелпером условного модификатора?

```swift
extension View {
  @ViewBuilder
  func `if`<T: View>(_ condition: Bool, transform: (Self) -> T) -> some View {
    if condition { transform(self) } else { self }
  }
}

struct Row: View {
  let isHighlighted: Bool

  var body: some View {
    Counter()   // holds @State private var count = 0
      .if(isHighlighted) { $0.foregroundStyle(.red) }
  }
}
```

`if`/`else` внутри хелпера даёт `_ConditionalContent<T, Self>`, поэтому подсвеченный и обычный `Counter` живут в двух разных ветках: это две разные структурные identity. Смена флага уничтожает один `Counter` вместе с его `@State` и создаёт другой с нуля.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-layout-rendering/#q-69218)

### 11. Найдите баг: все подписи в форме обрезаны до ширины «Phone», и «Email address» усечено, хотя ширину должна задавать самая широкая подпись.

```swift
struct LabelWidthKey: PreferenceKey {
  static let defaultValue: CGFloat = 0
  static func reduce(value: inout CGFloat, nextValue: () -> CGFloat) {
    value = nextValue()
  }
}
struct SettingsForm: View {
  @State private var labelWidth: CGFloat = 0
  let titles = ["Name", "Email address", "Phone"]
  var body: some View {
    VStack(alignment: .leading) {
      ForEach(titles, id: \.self) { title in
        Text(title)
          .background(GeometryReader { Color.clear.preference(key: LabelWidthKey.self, value: $0.size.width) })
          .frame(width: labelWidth == 0 ? nil : labelWidth, alignment: .leading)
      }
    }
    .onPreferenceChange(LabelWidthKey.self) { labelWidth = $0 }
  }
}
```

`reduce` вызывается по разу для каждого ребёнка, который задаёт preference, а `value = nextValue()` перезаписывает значение: выживает только ширина последней строки, поэтому `labelWidth` становится шириной «Phone», и все подписи сжимаются до неё.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-layout-rendering/#q-69220)

### 12. Почему `onAppear` в `LazyVStack` срабатывает раньше, чем строка появилась на экране, и как сделать пагинацию, которая это переживёт?

Ленивый стек строит строки чуть раньше, чем они видны (точная дистанция не документирована), а `onAppear` срабатывает при создании, то есть означает «построена», а не «видна». Сделайте триггер загрузки идемпотентным или используйте настоящий API видимости.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-layout-rendering/#q-69210)

### 13. Почему `proxy.scrollTo(id)`, вызванный сразу после добавления сообщения, часто промахивается, и что можно использовать вместо `ScrollViewReader` на iOS 17–18?

`scrollTo` ищет id в текущем дереве вью, а только что добавленная строка появится лишь после того, как SwiftUI применит изменение состояния, поэтому вызов разрешается по старому контенту. Альтернатива на состоянии: сделать позицию скролла состоянием через `.scrollPosition` и `scrollTargetLayout()`.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-layout-rendering/#q-69211)

### 14. Что ломается, если повесить `.ignoresSafeArea()` на корень экрана, и как дать дойти до краёв только фону?

`ignoresSafeArea` предлагает поддереву, к которому он прикреплён, большую область, поэтому на корне он уводит весь контент под статус-бар и индикатор Home. Без аргументов он ещё и игнорирует область клавиатуры, а это отключает подъём контента над клавиатурой.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-layout-rendering/#q-69214)

### 15. Как построить через протокол `Layout` раскладку тегов с переносом на новые строки и что обязаны делать `sizeThatFits` и `placeSubviews`?

`sizeThatFits` отвечает на предложение размером, который нужен контейнеру, измеряя детей через `LayoutSubview`; `placeSubviews` получает итоговые границы и должен разместить каждого ребёнка ровно один раз. Оба метода обязаны считать одинаково, иначе дети нарисуются за пределами заявленного размера.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-layout-rendering/#q-69206)

### 16. Найдите баг: этот layout тегов рисует теги в левом верхнем углу экрана, а не под заголовком в `VStack`.

```swift
struct TagFlow: Layout {
  var spacing: CGFloat = 8
  func sizeThatFits(proposal: ProposedViewSize, subviews: Subviews, cache: inout ()) -> CGSize {
    measure(subviews, maxWidth: proposal.width ?? .infinity) // correct: sums the rows, no origin involved
  }
  func placeSubviews(in bounds: CGRect, proposal: ProposedViewSize, subviews: Subviews, cache: inout ()) {
    var x: CGFloat = 0, y: CGFloat = 0, rowHeight: CGFloat = 0
    for subview in subviews {
      let size = subview.sizeThatFits(.unspecified)
      if x + size.width > bounds.width, x > 0 { x = 0; y += rowHeight + spacing; rowHeight = 0 }
      subview.place(at: CGPoint(x: x, y: y), proposal: ProposedViewSize(size))
      x += size.width + spacing
      rowHeight = max(rowHeight, size.height)
    }
  }
}
```

`placeSubviews` расставляет теги начиная с `(0, 0)`, но `bounds` описывает область, которую родитель выделил layout, в координатах родителя, и её начало почти никогда не равно нулю. `place(at:)` работает в тех же координатах, поэтому теги игнорируют заголовок и отступы над ними.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-layout-rendering/#q-69221)

### 17. Что делают `compositingGroup()` и `drawingGroup()` и когда каждый из них помогает, а когда вредит?

Тень, прозрачность и режим смешивания, применённые к контейнеру, ложатся на каждого ребёнка отдельно; `compositingGroup()` собирает поддерево в один слой до этих эффектов. `drawingGroup()` вдобавок растеризует его вне экрана через Metal в одну картинку ради скорости.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-layout-rendering/#q-69213)

### 18. Волна из 2000 вью `Rectangle` теряет кадры: когда переходить на `Canvas` и чем за это придётся заплатить?

Переходите, когда элементы являются просто пикселями: пользователь не нажимает на них, не фокусируется и не анимирует их по одному. Каждый `Rectangle` представляет собой вью с identity, layout и hit testing, а `Canvas` остаётся одной вью, которая рисует всё в `GraphicsContext` за один проход.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-layout-rendering/#q-69212)

### 19. Контейнер `Card` должен ставить разделитель между детьми, но переданный `ForEach` считается одной вью: как добраться до настоящих детей?

Разверните контент на отдельные subview: `Group(subviews:)` и `ForEach(subviewOf:)` (iOS 18) превращают значение `@ViewBuilder` в коллекцию отдельных детей, причём любой `ForEach` внутри уже развёрнут.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-layout-rendering/#q-69215)

### 20. `body` дешёвые, но инструмент SwiftUI показывает время в layout, и экран всё равно подтормаживает: что делает layout дорогим?

Дорогой layout означает, что одно и то же поддерево измеряется много раз за кадр: вложенные гибкие стеки, запросы идеального размера у `fixedSize` и `ViewThatFits`, текст, заново разбиваемый на строки для каждой предложенной ширины, и измерения, возвращаемые обратно в состояние.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-layout-rendering/#q-69216)

### 21. Что такое AttributeGraph и как SwiftUI решает, какой `body` выполнить заново после изменения состояния?

AttributeGraph представляет собой приватный граф зависимостей SwiftUI: состояние, значения environment, входные данные вью, результаты `body` и layout служат узлами, а рёбра фиксируют, кто что читал. Изменение помечает зависимые узлы как «грязные», и следующее обновление пересчитывает только их.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-layout-rendering/#q-69217)
