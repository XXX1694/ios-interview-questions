# Безопасность, приватность и Keychain: вопросы на собеседовании

Классы защиты данных, ATS и TLS, pinning, биометрия, jailbreak, секреты в сборке, разрешения, ATT и геолокация. Middle · 20 вопросов.

**[Тренировать тему на irbisa.com →](https://irbisa.com/ru/topic/ios-security-keychain/)** · [Все темы по iOS](../README.ru.md#все-темы)

### 1. Что такое Keychain в iOS, как он защищён и какие элементы в нём можно хранить?

Keychain представляет собой зашифрованную базу небольших секретов, которую ведёт система: приложение обращается к ней только через API `SecItem`, а каждый элемент зашифрован ключами, которые защищены кодом-паролем устройства и Secure Enclave.

[Полный ответ →](https://irbisa.com/ru/topic/ios-security-keychain/#q-71601)

### 2. Можно ли спрятать API-ключ в `Info.plist` или в `.xcconfig`? Где на самом деле должен жить секрет, от которого зависит приложение?

Нет. Всё, что есть в `.ipa`, получает каждый пользователь: `Info.plist` лежит в бандле обычным читаемым файлом, а строковые константы лежат в бинарнике, так что ключ достают за минуты. Ключ, который даёт реальные возможности, остаётся на вашем сервере, а приложение ходит в ваш бэкенд.

[Полный ответ →](https://irbisa.com/ru/topic/ios-security-keychain/#q-71603)

### 3. Как добавить в приложение Face ID / Touch ID через `LocalAuthentication`?

Создайте `LAContext`, вызовите `canEvaluatePolicy`, чтобы проверить, что биометрия доступна и настроена, затем `evaluatePolicy` со строкой-причиной. Для Face ID ещё нужен `NSFaceIDUsageDescription` в `Info.plist`.

[Полный ответ →](https://irbisa.com/ru/topic/ios-security-keychain/#q-71604)

### 4. Как устроены runtime-разрешения в iOS: что будет, если нет `NSCameraUsageDescription`, и что приложение может сделать после «Не разрешать»?

Приложение объявляет строку с назначением в `Info.plist`, а система спрашивает пользователя один раз, при первом запросе. Без `NSCameraUsageDescription` приложение убивается при этом первом обращении; после «Не разрешать» запрос больше не появится, и вернуть доступ может только сам пользователь в Настройках.

[Полный ответ →](https://irbisa.com/ru/topic/ios-security-keychain/#q-71617)

### 5. Как HTTPS защищает запрос: о чём договаривается TLS-рукопожатие и что приложение проверяет в сертификате сервера?

TLS даёт запросу шифрование, целостность и подтверждение того, кто сервер. В рукопожатии клиент и сервер договариваются о версии, шифре и общем ключе через ECDHE, а сервер доказывает, что владеет закрытым ключом сертификата; клиент проверяет этот сертификат, прежде чем отправить запрос.

[Полный ответ →](https://irbisa.com/ru/topic/ios-security-keychain/#q-71618)

### 6. Какие есть классы Data Protection для файлов, какой файл получает по умолчанию и когда его стоит менять?

Каждый файл шифруется ключом класса, который решает, когда его можно прочитать: `complete` разрешает это только при разблокированном устройстве, при `completeUnlessOpen` уже открытые файлы работают и после блокировки, `completeUntilFirstUserAuthentication` (по умолчанию) разрешает после первой разблокировки с момента загрузки, а `none` разрешает всегда.

[Полный ответ →](https://irbisa.com/ru/topic/ios-security-keychain/#q-71605)

### 7. Найдите баг: пользователей случайно разлогинивает, обычно когда приложение синхронизируется в фоне, а телефон заблокирован.

```swift
func saveToken(_ token: Data) {
  SecItemAdd([kSecClass: kSecClassGenericPassword, kSecAttrService: "auth", kSecValueData: token,
              kSecAttrAccessible: kSecAttrAccessibleWhenUnlocked] as CFDictionary, nil)
}
func readToken() -> String? {
  var result: CFTypeRef?
  let query = [kSecClass: kSecClassGenericPassword, kSecAttrService: "auth", kSecReturnData: true] as CFDictionary
  guard SecItemCopyMatching(query, &result) == errSecSuccess, let data = result as? Data else { return nil }
  return String(decoding: data, as: UTF8.self)
}
// Runs from a silent push and a BGAppRefreshTask, usually with the phone locked:
func backgroundSync() async {
  guard let token = readToken() else { return session.logOut() }
  await api.sync(token: token)
}
```

Токен сохранён как `kSecAttrAccessibleWhenUnlocked`, поэтому фоновая синхронизация на заблокированном телефоне получает `errSecInteractionNotAllowed`. `readToken()` превращает эту ошибку в `nil`, и приложение разлогинивает пользователя с совершенно рабочим токеном.

[Полный ответ →](https://irbisa.com/ru/topic/ios-security-keychain/#q-71606)

### 8. Запрос по `https` падает с `-1200` и `-9802` в логе, хотя тот же URL открывается в браузере. Чего требует ATS и чего не хватает серверу, и как это проверить?

ATS отклоняет TLS-рукопожатие, потому что сервер не дотягивает до минимума Apple: TLS 1.2 или новее, шифр с forward secrecy (ECDHE) и сертификат с подписью SHA-256 на ключе RSA от 2048 бит или ECC от 256 бит. Браузеры мягче, поэтому подключаются.

[Полный ответ →](https://irbisa.com/ru/topic/ios-security-keychain/#q-71607)

### 9. Найдите баг: пиннинг в тесте работает, но прокси с установленным пользователем CA всё равно читает трафик приложения.

```swift
final class PinningDelegate: NSObject, URLSessionDelegate {
  let pinnedKeyHash: Data   // SHA-256 of the server's public key
  init(pinnedKeyHash: Data) { self.pinnedKeyHash = pinnedKeyHash }

  func urlSession(_ session: URLSession, didReceive challenge: URLAuthenticationChallenge,
                  completionHandler: @escaping @Sendable (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {
    guard let trust = challenge.protectionSpace.serverTrust,
          SecTrustEvaluateWithError(trust, nil),
          let leaf = (SecTrustCopyCertificateChain(trust) as? [SecCertificate])?.first,
          let key = SecCertificateCopyKey(leaf),
          let raw = SecKeyCopyExternalRepresentation(key, nil) as Data?,
          Data(SHA256.hash(data: raw)) == pinnedKeyHash
    else {
      completionHandler(.performDefaultHandling, nil)   // pin mismatch: use system trust
      return
    }
    completionHandler(.useCredential, URLCredential(trust: trust))
  }
}
```

При несовпадении пина делегат откатывается на `.performDefaultHandling`, поэтому принимается любой сертификат, подписанный доверенным CA, и пиннинг ничего не защищает. Проверка пина лишь выбирает, какая ветка выполнится.

[Полный ответ →](https://irbisa.com/ru/topic/ios-security-keychain/#q-71602)

### 10. Как хешировать, шифровать и аутентифицировать данные с CryptoKit и где хранить ключ шифрования?

`SHA256.hash` даёт отпечаток, `AES.GCM.seal` (или `ChaChaPoly`) шифрует и одновременно обнаруживает подмену, а `HMAC` доказывает, кто создал сообщение. `SymmetricKey` хранится в Keychain, но никогда в бандле или рядом с зашифрованным файлом.

[Полный ответ →](https://irbisa.com/ru/topic/ios-security-keychain/#q-71608)

### 11. Как устроена приватность в `Logger` и `os_log` и что никогда не должно попадать в лог устройства?

Unified log скрывает интерполированные значения, которые вы не пометили `.public`: строки и объекты выводятся как `<private>`, а числа и булевы значения по умолчанию публичны. Токены, пароли и одноразовые коды не логируют вообще, ни скрытыми, ни открытыми.

[Полный ответ →](https://irbisa.com/ru/topic/ios-security-keychain/#q-71609)

### 12. Как не допустить попадания чувствительных экранов в снапшот переключателя приложений и в запись экрана на iOS?

В iOS нет флага, который запрещает скриншоты, поэтому контент закрывают сами: показывают защитный оверлей, как только сцена становится неактивной (до того, как система сделает снапшот для переключателя приложений), и прячут чувствительные view, пока `sceneCaptureState` равен `.active`.

[Полный ответ →](https://irbisa.com/ru/topic/ios-security-keychain/#q-71610)

### 13. Что такое App Tracking Transparency, когда можно показывать её запрос и что происходит с IDFA и атрибуцией рекламы, если пользователь отказал?

ATT управляет разрешением, без которого приложение не может отслеживать пользователя в приложениях и на сайтах других компаний, в том числе читать IDFA. Запрос появляется только пока приложение активно и только один раз; после отказа IDFA состоит из нулей, а атрибуция переходит на SKAdNetwork или AdAttributionKit.

[Полный ответ →](https://irbisa.com/ru/topic/ios-security-keychain/#q-71619)

### 14. Чем доступ к геолокации When In Use отличается от Always и что меняется для вашего кода, если выключена «Точная геопозиция»?

When In Use отдаёт координаты, пока приложением пользуются; Always вдобавок позволяет системе перезапускать приложение ради геозон, посещений и значительных перемещений. С выключенной «Точной геопозицией» место приблизительное: круг радиусом около 5 км, который пересчитывается примерно четыре раза в час.

[Полный ответ →](https://irbisa.com/ru/topic/ios-security-keychain/#q-71620)

### 15. Эта проверка на джейлбрейк стоит в банковском приложении. Что она ловит, как её обходят и что делать вместо `exit(0)`?

```swift
@MainActor func isJailbroken() -> Bool {
  let paths = ["/Applications/Cydia.app", "/bin/bash", "/usr/sbin/sshd", "/etc/apt"]
  if paths.contains(where: { FileManager.default.fileExists(atPath: $0) }) { return true }
  if UIApplication.shared.canOpenURL(URL(string: "cydia://")!) { return true }
  do {
    try "x".write(toFile: "/private/jb.txt", atomically: true, encoding: .utf8)
    return true   // the sandbox should have refused this write
  } catch {
    return false
  }
}

// application(_:didFinishLaunchingWithOptions:)
// if isJailbroken() { exit(0) }
```

Она ловит только старый rootful-джейлбрейк с Cydia и немодифицированную копию приложения. Современные rootless-джейлбрейки держат файлы в `/var/jb`, а твики для скрытия перехватывают `fileExists` и `canOpenURL`, так что каждая проверка возвращает `false`.

[Полный ответ →](https://irbisa.com/ru/topic/ios-security-keychain/#q-71611)

### 16. Что такое Secure Enclave и как создать и использовать в нём ключ через CryptoKit?

Secure Enclave представляет собой отдельный сопроцессор, который генерирует и использует ключи, никогда не показывая их iOS. В CryptoKit вы создаёте ключ `SecureEnclave.P256`, сохраняете его зашифрованный дескриптор и просите анклав подписать данные или выполнить key agreement.

[Полный ответ →](https://irbisa.com/ru/topic/ios-security-keychain/#q-71612)

### 17. Что меняют `.userPresence`, `.biometryAny` и `.biometryCurrentSet` для элемента Keychain и что с ним происходит, когда пользователь регистрирует новое лицо?

Они задают, что система потребует перед расшифровкой элемента: `.userPresence` принимает биометрию или код-пароль, `.biometryAny` принимает любое зарегистрированное лицо или палец, `.biometryCurrentSet` принимает только набор, зарегистрированный при сохранении элемента; после новой регистрации такой элемент больше не прочитать.

[Полный ответ →](https://irbisa.com/ru/topic/ios-security-keychain/#q-71613)

### 18. Вы пинните публичный ключ сервера. Как сменить этот ключ, не отрезав пользователей на старых версиях приложения?

Выпускайте пин следующего ключа заранее: каждый релиз пиннит текущий ключ и заранее сгенерированный резервный, так что сервер может переключиться на резервный, а старые сборки продолжат подключаться. Ротацию планируют по самой старой версии приложения в использовании, а не по дате истечения сертификата.

[Полный ответ →](https://irbisa.com/ru/topic/ios-security-keychain/#q-71614)

### 19. Что такое DeviceCheck и App Attest и что каждый из них позволяет проверить вашему серверу?

Оба позволяют серверу спросить Apple о том, кто к нему обращается. DeviceCheck хранит для вашей команды два бита на устройство, которые переживают переустановку; App Attest доказывает, что запрос подписан настоящим экземпляром вашего приложения на реальном устройстве Apple.

[Полный ответ →](https://irbisa.com/ru/topic/ios-security-keychain/#q-71615)

### 20. Как бы вы спроектировали блокировку банковского приложения с 6-значным PIN, Face ID и автоблокировкой через минуту в фоне?

Держите настоящие учётные данные за привязанными к устройству ключами Secure Enclave, а судьёй сделайте сервер: Face ID открывает ключ через `SecAccessControl`, PIN проверяет сервер с лимитом попыток, а сессией служит короткоживущий токен, который хранится только в памяти.

[Полный ответ →](https://irbisa.com/ru/topic/ios-security-keychain/#q-71616)
