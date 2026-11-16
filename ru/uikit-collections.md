# UICollectionView и diffable data source: вопросы на собеседовании

Compositional layout, diffable data source, регистрация ячеек, content configuration, префетч. Middle · 23 вопроса.

**[Тренировать тему на irbisa.com →](https://irbisa.com/ru/topic/uikit-collections/)** · [Все темы по iOS](../README.ru.md#все-темы)

### 1. Когда выбирать `UICollectionView`, а не `UITableView`?

Collection view выбирают всегда, когда экран не сводится к одной вертикальной колонке строк на всю ширину: сетки, карусели, смешанные секции. А начиная с iOS 14 его часто выбирают и для обычных списков, потому что list configuration даёт внутри него строки в стиле таблицы.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-collections/#q-69317)

### 2. Товары без скидки после небольшого скролла начинают показывать бейдж SALE: что не так с этим кодом?

```swift
override func collectionView(_ collectionView: UICollectionView,
                             cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
  let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "ProductCell",
                                                for: indexPath) as! ProductCell
  let product = products[indexPath.item]
  cell.titleLabel.text = product.title
  if product.isOnSale {
    cell.saleBadge.isHidden = false
    cell.priceLabel.textColor = .systemRed
  }
  return cell
}
```

Ячейка переиспользуется, а код выставляет бейдж и красную цену только в ветке `true`: ячейка, которая последней показывала товар со скидкой, сохраняет и то и другое, когда её достают из очереди для обычного товара. С заголовком всё в порядке, потому что он выставляется на любом пути.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-collections/#q-69319)

### 3. Найдите баг: свайп для удаления заметки роняет приложение с `NSInternalInconsistencyException`.

```swift
override func tableView(_ tableView: UITableView,
                        commit editingStyle: UITableViewCell.EditingStyle,
                        forRowAt indexPath: IndexPath) {
  guard editingStyle == .delete else { return }
  tableView.deleteRows(at: [indexPath], with: .automatic)
  notes.remove(at: indexPath.row)
  storage.save(notes)
}
```

`deleteRows` вызывается до того, как модель уменьшилась: во время обновления таблица спрашивает у data source новое число строк, всё ещё получает старое и бросает `NSInternalInconsistencyException` с «Invalid update: invalid number of rows in section 0».

[Полный ответ →](https://irbisa.com/ru/topic/uikit-collections/#q-69326)

### 4. При быстром скролле сетки фотографий ячейки на мгновение показывают картинку предыдущего элемента: в чём причина и как это исправить?

Ячейка из очереди представляет собой переиспользованный объект, на котором ещё висит старая картинка, а запрос, запущенный для её предыдущего элемента, позже дописывает в неё результат. Очищайте картинку и отменяйте загрузку при переиспользовании, а перед установкой результата проверяйте, какой элемент ячейка показывает сейчас.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-collections/#q-69301)

### 5. Найдите баг: после прихода новых песен нажатие на звёздочку в строке иногда добавляет в избранное другую песню.

```swift
override func tableView(_ tableView: UITableView,
                        cellForRowAt indexPath: IndexPath) -> UITableViewCell {
  let cell = tableView.dequeueReusableCell(withIdentifier: "Song", for: indexPath) as! SongCell
  cell.titleLabel.text = songs[indexPath.row].title
  cell.starButton.tag = indexPath.row
  cell.starButton.addTarget(self, action: #selector(starTapped(_:)), for: .touchUpInside)
  return cell
}

@objc private func starTapped(_ sender: UIButton) {
  songs[sender.tag].isFavorite.toggle()
}

func songArrived(_ song: Song) {
  songs.insert(song, at: 0)
  tableView.insertRows(at: [IndexPath(row: 0, section: 0)], with: .automatic)
}
```

Индекс строки копируется в `starButton.tag` при конфигурации ячейки, а `insertRows` не переконфигурирует ячейки, которые уже на экране, поэтому после появления песни сверху каждая видимая звёздочка хранит старый индекс и переключает песню на строку выше.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-collections/#q-69325)

### 6. Как сделать сетку в три колонки на `UICollectionViewFlowLayout` и когда переходить на compositional layout?

Flow layout раскладывает элементы в строку и переносит на следующую, поэтому сетка задаётся просто шириной элемента, посчитанной из доступной ширины: вычесть отступы секции и промежутки, разделить на число колонок и округлить вниз.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-collections/#q-69320)

### 7. Зачем нужен diffable data source вместо `reloadData` или `performBatchUpdates`?

Потому что вы описываете всё состояние снапшотом, а вставки, удаления и перемещения UIKit вычисляет сам. `reloadData` не анимирует и сбрасывает все видимые ячейки; `performBatchUpdates` падает, как только ваша арифметика indexPath разойдётся с моделью.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-collections/#q-69321)

### 8. Что класть в diffable-снапшот (структуры модели или только их id) и что ломается, если класть структуры?

Только стабильные id. Дифф сопоставляет элементы по `Hashable`-идентичности, поэтому если идентификатором служит сама структура, правка любого поля создаёт новую идентичность: строка анимированно уезжает и возвращается, а две строки с одинаковыми значениями становятся дубликатами, на которых снапшот падает.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-collections/#q-69307)

### 9. Переключение звёздочки «избранное» через `reloadItems` заставляет обложку мигать и убирает клавиатуру из текстового поля в той же строке: что нужно было вызвать?

`snapshot.reconfigureItems([id])`: он заново запускает регистрацию ячейки на *той же* ячейке, что на экране, а `reloadItems` выбрасывает эту ячейку и достаёт из очереди новую.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-collections/#q-69309)

### 10. Что заменяют `CellRegistration` и `contentConfiguration` в современном collection view и что они дают?

Они заменяют строковые reuse identifier с кастом `as!` и ручную настройку лейблов и картинок ячейки и убирают ошибки, которые допускал этот подход: регистрация связывает класс ячейки с типом элемента на этапе компиляции, а content configuration делает внешний вид ячейки значением, которое вы присваиваете, а не набором свойств, которые вы меняете.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-collections/#q-69306)

### 11. Этот data source компилируется без предупреждений: что произойдёт при запуске и как это исправить?

```swift
dataSource = UICollectionViewDiffableDataSource<Section, Contact.ID>(collectionView: collectionView) {
  collectionView, indexPath, id in
  let registration = UICollectionView.CellRegistration<UICollectionViewListCell, Contact.ID> {
    cell, _, id in
    var content = cell.defaultContentConfiguration()
    content.text = ContactStore.shared.name(for: id)
    cell.contentConfiguration = content
  }
  return collectionView.dequeueConfiguredReusableCell(using: registration, for: indexPath, item: id)
}
```

На iOS 15 и новее на первом же dequeue вылетит исключение: UIKit обнаруживает регистрацию, созданную внутри cell provider. Каждый вызов создавал бы новую регистрацию со своим reuse identifier, и ни одна ячейка не могла бы переиспользоваться.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-collections/#q-69322)

### 12. Что дают item, group и section в compositional layout и относительно чего на самом деле считается `.fractionalWidth(1.0)`?

Item задаёт место под одну ячейку, group выстраивает items вдоль оси, section повторяет свою группу и добавляет отступы и прокрутку, а layout содержит секции. Дробный размер всегда означает долю от контейнера элемента: для item это его группа, для группы это секция.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-collections/#q-69302)

### 13. Лента самоподстраивающихся ячеек дёргается при скролле, а индикатор прокрутки всё время меняет размер: в чём причина?

Пока ячейка не измерена, layout учитывает в размере контента её *оценочную* высоту. Каждое реальное измерение заменяет оценку, общая высота меняется, и контент под пальцем сдвигается; чем дальше оценка от реальности, тем сильнее скачок.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-collections/#q-69311)

### 14. Дизайн хочет вид inset-grouped таблицы с разделителями и swipe-to-delete, а экран уже сделан на collection view: что использовать?

`UICollectionLayoutListConfiguration`: она превращает compositional layout или одну его секцию в строки в стиле таблицы с разделителями, хедерами, свайп-действиями и аксессуарами.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-collections/#q-69305)

### 15. Пользователь пролистывает сотню строк, а запросы картинок продолжают приходить для ячеек, давно ушедших с экрана: что пропустил код префетча?

`collectionView(_:cancelPrefetchingForItemsAt:)`: без него каждый быстрый скролл забивает очереди загрузки и декодирования работой для элементов, на которые никто не посмотрит, и видимые строки ждут за ней.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-collections/#q-69312)

### 16. Этот код бесконечной ленты иногда загружает одну страницу дважды, а потом падает с ошибкой дублирующихся идентификаторов: что не так и как сделать правильно?

```swift
func collectionView(_ collectionView: UICollectionView,
                    willDisplay cell: UICollectionViewCell,
                    forItemAt indexPath: IndexPath) {
  guard indexPath.item == posts.count - 1 else { return }
  Task {
    let page = try await api.fetchPosts(page: currentPage + 1)
    currentPage += 1
    posts += page
    var snapshot = dataSource.snapshot()
    snapshot.appendItems(page.map(\.id))
    await dataSource.apply(snapshot)
  }
}
```

Ничто не мешает второму запросу, пока первый ещё идёт. `willDisplay` срабатывает для последней ячейки повторно (отскок, повторная раскладка), обе задачи читают одинаковый `currentPage`, и второе добавление вставляет идентификаторы, которые уже есть в снапшоте: «supplied item identifiers are not unique».

[Полный ответ →](https://irbisa.com/ru/topic/uikit-collections/#q-69323)

### 17. Как вставить горизонтально прокручиваемый collection view в ячейку таблицы и что ломается при переиспользовании?

Дайте каждой ячейке свой collection view и свой data source, передавайте элементы строки при конфигурации и сохраняйте горизонтальное смещение каждой строки по её id, иначе из-за переиспользования одна строка окажется прокрученной туда, где оставили другую, а прокрученная строка при возврате сбросится.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-collections/#q-69324)

### 18. Перестановка перетаскиванием выглядит правильно, пока следующее обновление модели не вернёт строку на прежнее место: что упустил обработчик перестановки?

UIKit переместил ячейку, а data source обновил свой снапшот, но новый порядок никто не записал в вашу модель, и следующий снапшот, построенный из модели, возвращает строку назад.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-collections/#q-69313)

### 19. Нужна скруглённая тонированная подложка под всей секцией и маленький бейдж, свисающий с угла ячейки: какие объекты layout это дают?

Decoration item для фона секции и привязанный supplementary item для бейджа. Декорации создаёт и размещает только layout; supplementary-вью достаются через ваш data source, как ячейки.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-collections/#q-69304)

### 20. Горизонтальная карусель с постраничной прокруткой должна увеличивать центральную карточку во время свайпа: как это сделать в compositional layout?

Поставьте секции `orthogonalScrollingBehavior = .groupPagingCentered` и масштабируйте карточки в `visibleItemsInvalidationHandler`: он вызывается на каждом кадре прокрутки этой секции с видимыми элементами и текущим смещением.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-collections/#q-69303)

### 21. Вы написали свой `UICollectionViewLayout` для «ступенчатой» сетки, и скролл ползёт: какой метод делает слишком много?

`layoutAttributesForElements(in:)`: UIKit вызывает его снова и снова, пока пользователь скроллит, поэтому версия, которая считает фреймы или обходит все элементы, делает каждый шаг скролла O(число элементов).

[Полный ответ →](https://irbisa.com/ru/topic/uikit-collections/#q-69314)

### 22. Живая лента из десяти тысяч строк на мгновение замирает при каждом обновлении из сокета: на что тратит время `apply` и что менять?

Внутри `apply`: он хеширует и сравнивает каждый идентификатор, чтобы найти изменения, а потом анимирует каждое изменение, причём на главном потоке. С тяжёлыми структурами-идентификаторами и одним `apply` на каждое сообщение сокета полный дифф повторяется много раз в секунду.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-collections/#q-69315)

### 23. Вам достался `UITableViewDelegate` на 600 строк, и его нужно перевести на compositional list: что переносить первым и как доказать, что ничего не сломалось?

Сначала выделите модель строк (то, по чему делает `switch` метод `cellForRowAt`, становится идентификатором элемента в снапшоте) и покройте тестами эту чистую функцию «состояние → строки» до того, как поменяется хоть один тип UIKit.

[Полный ответ →](https://irbisa.com/ru/topic/uikit-collections/#q-69316)
