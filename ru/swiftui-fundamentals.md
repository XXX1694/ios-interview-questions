# Основы SwiftUI: вопросы на собеседовании

Протокол View, модификаторы, контейнеры, навигация, списки. Junior · 25 вопросов.

**[Тренировать тему на irbisa.com →](https://irbisa.com/ru/topic/swiftui-fundamentals/)** · [Все темы по iOS](../README.ru.md#все-темы)

### 1. Что такое SwiftUI и в чём его плюсы и минусы по сравнению с UIKit?

SwiftUI представляет собой декларативный UI-фреймворк Apple: вы описываете, как выглядит экран при текущем состоянии, а фреймворк сам решает, какие изменения применить. В императивном UIKit вы создаёте view и затем сами их изменяете.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-2801)

### 2. Что значит «декларативный UI» и почему view в SwiftUI сделаны структурами, а не классами?

Декларативный означает, что вы описываете результат (как выглядит экран при данном состоянии), а обновления выполняет фреймворк. View в SwiftUI устроена как дешёвая структура, которая описывает кусок интерфейса; это не тот объект, что рисуется на экране.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-62815)

### 3. Что такое протокол `View`, что такое `body` и что означает `some View`?

Протоколу `View` соответствует каждая view в SwiftUI, и его единственным требованием служит вычисляемое свойство `body`, возвращающее другие view. `some View` обозначает непрозрачный (opaque) тип возврата: `body` возвращает один конкретный тип, который знает компилятор, но писать его вам не нужно.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-62816)

### 4. Что делают `@main`, `App` и `Scene` в SwiftUI-приложении?

`@main` отмечает точку входа в приложение, протокол `App` описывает приложение целиком, а его `body` возвращает одну или несколько сцен, то есть контейнеров вроде `WindowGroup`, которые система превращает в окна.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-62817)

### 5. Как `VStack`, `HStack` и `ZStack` раскладывают дочерние view и когда вместо них нужен `LazyVStack`?

`VStack` ставит дочерние view сверху вниз, `HStack` ставит их от leading к trailing, а `ZStack` накладывает их слоями от заднего к переднему. `LazyVStack` и `LazyHStack` раскладывают так же, но создают дочерние view только при приближении к экрану, что важно для длинного контента в `ScrollView`.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-2802)

### 6. Как выглядит каждый из трёх бейджей и почему перестановка двух модификаторов меняет результат?

```swift
// Badge A
Text("Sale")
  .padding()
  .background(.red)

// Badge B
Text("Sale")
  .background(.red)
  .padding()

// Badge C
Text("Sale")
  .background(.red)
  .frame(width: 200)
```

Бейдж A получается красным прямоугольником с текстом внутри стандартного отступа, B выглядит как узкая красная полоса за текстом с невидимым отступом вокруг, а C красный только за текстом внутри более широкой пустой рамки. Каждый модификатор оборачивает view, на которой вызван, поэтому влияет только на то, что было до него в цепочке.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-62818)

### 7. Как сделать переиспользуемый стиль в SwiftUI: свой `ViewModifier` или `ButtonStyle`?

`ViewModifier` нужен, чтобы собрать набор модификаторов для любой view, например вид карточки, а `ButtonStyle` нужен, чтобы стилизовать кнопки, потому что он дополнительно получает состояние нажатия и роль кнопки.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-2803)

### 8. Вы растянули `Text` через `.frame(maxWidth: .infinity)`, а он остался по центру. Чем различаются выравнивания в SwiftUI?

`.frame` по умолчанию центрирует дочернюю view, поэтому нужен `.frame(maxWidth: .infinity, alignment: .leading)`. `alignment` у стека выравнивает детей относительно друг друга, `alignment` у frame размещает дочернюю view внутри рамки, а `multilineTextAlignment` выравнивает строки внутри одного `Text`.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-62808)

### 9. Картинка `Image` из бандла игнорирует `.frame`, а `.scaledToFill()` вылезает на соседей: в каком порядке ставить модификаторы и чего не делает за вас `AsyncImage`?

`Image` сохраняет свой натуральный размер, пока вы не вызовете `.resizable()`, а `.scaledToFill()` рисует за пределами рамки, если её не обрезать. У `AsyncImage` нет кэша декодированных картинок в памяти, поэтому строка, вернувшаяся в список при скролле, снова запрашивает картинку и снова её декодирует.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-62809)

### 10. Как собрать `List` с `ForEach` и зачем каждой строке стабильный `id`?

`List` служит прокручиваемым контейнером в стиле платформы, а `ForEach` создаёт по строке на каждый элемент коллекции. SwiftUI сопоставляет строки между обновлениями по `id`, поэтому id должен браться из данных и никогда не меняться, иначе строки теряют состояние и анимируются неправильно.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-2805)

### 11. Как собрать форму в SwiftUI с `TextField`, `SecureField`, `Toggle` и `Picker`?

Оберните контролы в `Form`, который даёт им сгруппированный вид как в настройках, и привяжите каждый контрол к состоянию через `$`, например `TextField("Email", text: $email)`. Контрол сам читает и пишет это состояние, поэтому не нужно подключать делегат или target-action.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-62819)

### 12. Как показать sheet или alert в SwiftUI и когда использовать форму `item:` вместо `isPresented:`?

Показ экрана задаётся модификатором, привязанным к состоянию: установите `Bool` или опциональное значение, и SwiftUI покажет sheet или alert; сбросьте, и экран закроется. Используйте `.sheet(item:)` всякий раз, когда sheet показывает конкретное значение, например нажатую строку.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-62803)

### 13. Когда использовать `LazyVGrid`, а когда `Grid`, и как сделать, чтобы число колонок зависело от ширины экрана?

`LazyVGrid` раскладывает коллекцию по колонкам и строит ячейки лениво, что подходит для галереи; `Grid`, неленивая таблица, строки которой делят ширину колонок, подходит для формы или таблицы сравнения. `.adaptive(minimum:)` вмещает столько колонок, сколько позволяет ширина.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-62811)

### 14. Как писать превью SwiftUI для view с несколькими состояниями или с зависимостью от сервиса?

Используйте макрос `#Preview` (по одному именованному превью на каждое состояние, способное сломать вёрстку) и подавайте в него тестовые данные и фейковые сервисы, чтобы превью никогда не ходило в сеть или в настоящую базу.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-2806)

### 15. Когда использовать `List`, а когда `ScrollView` с `LazyVStack`?

`List` подходит для длинных однотипных данных: он переиспользует строки и бесплатно даёт разделители, swipe-действия, выделение и режим редактирования. `ScrollView` с `LazyVStack` берут, когда дизайну нужна своя вёрстка, с которой стиль списка будет бороться.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-62805)

### 16. Как SwiftUI определяет размер view: за кем последнее слово, за родителем или за ребёнком?

Layout устроен как переговоры: родитель предлагает размер, дочерняя view выбирает свой размер, а затем родитель её размещает. Последнее слово за ребёнком, и это объясняет большинство сюрпризов вёрстки.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-62801)

### 17. Как работает `NavigationStack` и как переходить между экранами из кода или по диплинку?

`NavigationStack` хранит свои экраны как путь (path) из значений: `NavigationLink(value:)` добавляет значение, `.navigationDestination(for:)` превращает значение этого типа в экран, а кнопка «Назад» удаляет последнее. Привяжите path к состоянию, и навигация сводится к изменению массива.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-2804)

### 18. Тап по строке этого списка ничего не делает, а Xcode пишет предупреждение о навигации: в чём ошибка?

```swift
struct ShopView: View {
  let products: [Product]

  var body: some View {
    NavigationStack {
      List(products) { product in
        NavigationLink(product.name, value: product)
          .navigationDestination(for: Product.self) { product in
            ProductDetail(product: product)
          }
      }
      .navigationTitle("Shop")
    }
  }
}
```

`.navigationDestination` стоит внутри строк `List`. `List` строит строки лениво, поэтому стек не может рассчитывать, что увидит destination в момент срабатывания ссылки, и ссылка может ничего не сделать; перенесите модификатор на сам `List`.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-62820)

### 19. Что выведет этот код при появлении экрана и при каждом тапе и что это говорит об `init`, `body` и `onAppear`?

```swift
struct Child: View {
  init() { print("init") }
  var body: some View {
    let _ = print("body")
    Text("Child").onAppear { print("appear") }
  }
}

struct Parent: View {
  @State private var count = 0
  var body: some View {
    VStack { Text("Taps: \(count)"); Child(); Button("Tap") { count += 1 } }
  }
}
```

В обычном запуске при первом появлении выводится `init`, `body`, `appear`; после каждого тапа выводится только `init`. `body` родителя перевычисляется и заново создаёт структуру `Child`, но новое значение равно старому, поэтому SwiftUI пропускает `Child.body`, а дочерняя view не уходила с экрана, поэтому `onAppear` повторно не срабатывает.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-62810)

### 20. Найдите баг: при каждом возвращении на этот экран секундомер считает всё быстрее и продолжает тикать, пока поверх открыт другой экран.

```swift
struct StopwatchView: View {
  @State private var seconds = 0

  var body: some View {
    Text("\(seconds) s")
      .font(.largeTitle.monospacedDigit())
      .onAppear {
        Task {
          do {
            while true {
              try await Task.sleep(for: .seconds(1))
              seconds += 1
            }
          } catch {}
        }
      }
  }
}
```

`Task`, запущенный в `.onAppear`, не привязан к view, и его никто не отменяет: каждое возвращение на экран запускает ещё один бесконечный цикл, и все они продолжают увеличивать `seconds`, даже пока этот экран закрыт другим.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-62821)

### 21. Что `@ViewBuilder` делает под капотом и почему возвращать `AnyView` считается плохой привычкой?

`@ViewBuilder` работает как result builder, который сворачивает список выражений в один вложенный generic-тип view; поэтому в `body` можно написать несколько view и `if`, не выписывая этот тип вручную. `AnyView` стирает этот тип, и SwiftUI больше не может сравнивать старое и новое дерево структурно.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-62802)

### 22. Чем `withAnimation` отличается от модификатора `.animation` и почему одноаргументный `.animation(_:)` объявили устаревшим?

`withAnimation` анимирует все изменения, вызванные мутацией состояния внутри его замыкания, где бы ни находились затронутые view; модификатор `.animation(_:value:)` анимирует реакцию одной view на одно конкретное значение.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-62804)

### 23. В строке списка есть кнопка «звезда» и кнопка удаления, и тап по любой запускает обе плюс навигацию строки: что происходит?

Внутри строки `List` кнопки со стилем по умолчанию используют всю строку как область нажатия, поэтому один тап срабатывает у всех кнопок строки. Задайте каждой кнопке `.buttonStyle(.borderless)` или `.plain`, чтобы она отвечала только за область своего лейбла.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-62813)

### 24. После сохранения на четвёртом экране стека нужно вернуться в корень, а тот же флоу работает и внутри sheet: как это построить?

Возврат в корень сводится к одному изменению данных (очистке массива, управляющего `NavigationStack`), поэтому глубокому экрану нужен доступ к этому path, ведь `@Environment(\.dismiss)` откатывает только один уровень. В sheet у флоу свой `NavigationStack` и свой роутер.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-62814)

### 25. Найдите баг: при выборе другой статьи заголовок обновляется, а этот web view продолжает показывать первую страницу.

```swift
struct WebView: UIViewRepresentable {
  let url: URL

  func makeUIView(context: Context) -> WKWebView {
    let webView = WKWebView()
    webView.load(URLRequest(url: url))
    return webView
  }

  func updateUIView(_ webView: WKWebView, context: Context) {}
}

// In the parent: WebView(url: selectedArticle.url)
```

`updateUIView` пустой, поэтому новый `url` так и не доходит до `WKWebView`. `makeUIView` вызывается один раз на идентичность view; когда родитель передаёт другой URL, SwiftUI оставляет тот же UIKit-view и вызывает только `updateUIView`, который ничего не делает.

[Полный ответ →](https://irbisa.com/ru/topic/swiftui-fundamentals/#q-62822)
