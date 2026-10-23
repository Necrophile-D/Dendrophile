# Практическая работа №7 — решения заданий (Dart)

> Каждый блок — отдельная программа со своим `main()`. Вставляй по одному в [DartPad](https://dartpad.dev), жми **Run**, вывод появится справа. Нужен Dart 3+ (в DartPad он по умолчанию).


---

## 1. Тема 1. Числовые типы

### №1. Конвертер валют

```dart
String convertRublesToDollars(int rubles, double rate) {
  if (rate <= 0) throw ArgumentError('Курс должен быть положительным');
  return (rubles / rate).toStringAsFixed(2);
}

void main() => print(convertRublesToDollars(1000, 92.5));
```

### №2. Безопасный калькулятор возраста

```dart
int parseAge(String input) => int.tryParse(input.trim()) ?? -1;

void main() {
  print(parseAge('18'));     // 18
  print(parseAge('восемь')); // -1
}
```

### №3. Расчет плотности пикселей (DP)

```dart
double pixelsToDp(double px, double density) {
  if (density <= 0) throw ArgumentError('Плотность должна быть больше нуля');
  return px / density;
}

void main() => print(pixelsToDp(300, 3)); // 100.0
```

### №4. Побитовые флаги сенсоров

```dart
const int accelerometer = 1 << 0;
const int gyroscope = 1 << 1;
const int proximity = 1 << 2;

void main() {
  int enabledSensors = accelerometer | gyroscope;
  print((enabledSensors & accelerometer) != 0); // true
  enabledSensors |= proximity;
  print(enabledSensors); // 7
}
```

### №5. Проверка четности побитовым оператором

```dart
bool isEven(int number) => (number & 1) == 0;

void main() {
  print(isEven(12)); // true
  print(isEven(7));  // false
}
```


---

## 2. Тема 2. Строки

### №1. Маскирование email

```dart
String maskEmail(String email) {
  final at = email.indexOf('@');
  if (at <= 0 || at == email.length - 1) return email;
  final name = email.substring(0, at);
  final domain = email.substring(at);
  if (name.length <= 2) return '${name[0]}${List.filled(name.length - 1, '*').join()}$domain';
  return '${name[0]}${List.filled(name.length - 2, '*').join()}${name[name.length - 1]}$domain';
}

void main() => print(maskEmail('ivanov.flutter@gmail.com'));
```

### №2. Валидатор безопасного префикса ссылки

```dart
bool isSafeUrl(String value) => value.trim().toLowerCase().startsWith('https://');

void main() {
  print(isSafeUrl('https://example.com')); // true
  print(isSafeUrl('http://example.com'));  // false
}
```

### №3. Генератор инициалов

```dart
String initials(String fullName) {
  final parts = fullName.trim().split(RegExp(r'\s+'));
  if (parts.length < 2) return parts.isEmpty ? '' : parts.first[0].toUpperCase();
  return '${parts[0][0]}${parts[1][0]}'.toUpperCase();
}

void main() => print(initials('Александр Пушкин')); // АП
```

### №4. Разбор CSV-строки корзины

```dart
void main() {
  final fields = 'Яблоко,3,150.5'.split(',');
  if (fields.length != 3) throw FormatException('Ожидалось три поля');
  final name = fields[0];
  final quantity = int.parse(fields[1]);
  final price = double.parse(fields[2]);
  print('$name: $quantity шт. по $price руб.');
}
```

### №5. Подсчет кодовых точек в строке с эмодзи

```dart
int countCodePoints(String text) => text.runes.length;

void main() {
  print(countCodePoints('Привет 🚀')); // Количество Unicode code points
  // Примечание: .runes считает кодовые точки, но не все визуальные составные эмодзи как один символ.
}
```


---

## 3. Тема 3. Логический тип

### №1. Проверка права доступа к экрану

```dart
bool canOpenScreen({
  required bool isAuthorized,
  required bool emailConfirmed,
  required bool isBlocked,
}) => isAuthorized && emailConfirmed && !isBlocked;

void main() => print(canOpenScreen(
  isAuthorized: true, emailConfirmed: true, isBlocked: false,
));
```

### №2. Определитель пустоты строки

```dart
bool isBlank(String? input) => input == null || input.trim().isEmpty;

void main() {
  print(isBlank(null));    // true
  print(isBlank('   '));   // true
  print(isBlank(' Dart '));// false
}
```

### №3. Тоггл состояния темной темы

```dart
bool toggleDarkMode(bool isDarkMode) => !isDarkMode;

void main() {
  var isDarkMode = false;
  isDarkMode = toggleDarkMode(isDarkMode);
  print(isDarkMode); // true
}
```

### №4. Валидатор возраста для аренды авто

```dart
bool canRentCar(int age, int drivingYears) => age >= 21 && drivingYears >= 2;

void main() {
  print(canRentCar(25, 4)); // true
  print(canRentCar(20, 3)); // false
}
```

### №5. Строгая проверка положительного баланса

```dart
bool hasPositiveBalance(num balance) => balance > 0;

void main() {
  print(hasPositiveBalance(0));    // false
  print(hasPositiveBalance(0.01)); // true
}
```


---

## 4. Тема 4. Списки

### №1. Удаление дубликатов с сохранением порядка

```dart
List<int> uniqueInOrder(List<int> values) => values.toSet().toList();

void main() => print(uniqueInOrder([3, 1, 3, 2, 1])); // [3, 1, 2]
```

### №2. Пагинация списка

```dart
List<T> getPage<T>(List<T> list, int page, int pageSize) {
  if (page < 0 || pageSize <= 0) throw ArgumentError('Некорректная страница');
  final start = page * pageSize;
  if (start >= list.length) return [];
  final end = (start + pageSize < list.length) ? start + pageSize : list.length;
  return list.sublist(start, end);
}

void main() => print(getPage([1, 2, 3, 4, 5], 1, 2)); // [3, 4]
```

### №3. Сортировка товаров по цене по убыванию

```dart
void main() {
  final prices = <double>[99.9, 20, 150.5, 75];
  prices.sort((a, b) => b.compareTo(a));
  print(prices); // [150.5, 99.9, 75.0, 20.0]
}
```

### №4. Поиск первого слова длиннее 8 символов

```dart
void main() {
  final words = ['dart', 'mobile', 'application', 'flutter'];
  final result = words.firstWhere(
    (word) => word.length > 8,
    orElse: () => 'Подходящее слово не найдено',
  );
  print(result);
}
```

### №5. Генератор строк через List.generate

```dart
void main() {
  final slots = List<String>.generate(10, (index) => 'Слот ${index + 1}');
  print(slots);
}
```


---

## 5. Тема 5. Set и Map

### №1. Подсчет частоты слов

```dart
Map<String, int> wordFrequency(String text) {
  final result = <String, int>{};
  for (final word in text.toLowerCase().split(RegExp(r'[^\p{L}\p{N}]+', unicode: true))) {
    if (word.isEmpty) continue;
    result[word] = (result[word] ?? 0) + 1;
  }
  return result;
}

void main() => print(wordFrequency('Dart Flutter Dart'));
```

### №2. Симметрическая разность тегов

```dart
Set<T> symmetricDifference<T>(Set<T> a, Set<T> b) =>
    (a.difference(b)).union(b.difference(a));

void main() => print(symmetricDifference(
  {'mobile', 'dart'}, {'dart', 'flutter'},
)); // {mobile, flutter}
```

### №3. Безопасное извлечение вложенной настройки

```dart
String readStringSetting(
  Map<String, dynamic> settings,
  String key, {
  String fallback = 'По умолчанию',
}) {
  final value = settings[key];
  return value is String ? value : fallback;
}

void main() => print(readStringSetting({'theme': 'dark'}, 'theme'));
```

### №4. Конвертация цен из рублей в доллары

```dart
Map<String, double> convertPrices(
  Map<String, double> rubPrices,
  double rubPerDollar,
) {
  if (rubPerDollar <= 0) throw ArgumentError('Некорректный курс');
  return rubPrices.map((name, rubles) =>
      MapEntry(name, rubles / rubPerDollar));
}

void main() => print(convertPrices({'Телефон': 92500}, 92.5));
```

### №5. Группировка пользователей по городу

```dart
Map<String, List<Map<String, dynamic>>> groupByCity(
  List<Map<String, dynamic>> users,
) {
  final grouped = <String, List<Map<String, dynamic>>>{};
  for (final user in users) {
    final city = user['city'];
    if (city is! String || city.trim().isEmpty) continue;
    grouped.putIfAbsent(city, () => []).add(user);
  }
  return grouped;
}

void main() => print(groupByCity([
  {'name': 'Анна', 'city': 'Тула'},
  {'name': 'Иван', 'city': 'Тула'},
  {'name': 'Олег', 'city': 'Москва'},
]));
```


---

## 6. Тема 6. Null Safety, Records и Pattern Matching

### №1. Координаты центра экрана через Record

```dart
({double x, double y}) screenCenter(double width, double height) =>
    (x: width / 2, y: height / 2);

void main() {
  final center = screenCenter(1080, 2400);
  print('x=${center.x}, y=${center.y}');
}
```

### №2. Безопасная распаковка профиля

```dart
String displayBio(String? bio) => bio ?? 'Информация отсутствует';

void main() {
  print(displayBio(null));          // Информация отсутствует
  print(displayBio('Люблю Dart'));  // Люблю Dart
}
```

### №3. Pattern Matching над JSON

```dart
String describePacket(Object? json) => switch (json) {
  {'id': int id} => 'Валидный пакет, id=$id',
  {'error': String message} => 'Ошибка: $message',
  _ => 'Неизвестный формат пакета',
};

void main() {
  print(describePacket({'id': 42, 'data': 'ok'}));
  print(describePacket({'error': 'Нет доступа'}));
}
```

### №4. Отложенное вычисление с late final

```dart
late final double expensiveCalculation = calculate();

double calculate() {
  print('Вычисление выполняется один раз');
  return List<double>.generate(1000, (i) => i / 3)
      .fold(0.0, (sum, value) => sum + value);
}

void main() {
  print('До первого чтения');
  print(expensiveCalculation); // вычисление запускается здесь
  print(expensiveCalculation); // повторного вычисления нет
}
```

### №5. Безопасный поиск первого числа

```dart
int firstOrMinusOne(List<int>? numbers) =>
    (numbers != null && numbers.isNotEmpty) ? numbers[0] : -1;

void main() {
  List<int>? numbers = [10, 20, 30];
  print(numbers?[0] ?? -1);                 // 10 (оператор ?[0] и ??)
  List<int>? nothing;
  print(nothing?[0] ?? -1);                 // -1
  print(firstOrMinusOne([]));               // -1 (безопасно и для пустого списка)
}
```


---

## Смешанные задания — Блок 1. Числа и финансы

### №1. Калькулятор скидки с промокодом

```dart
double finalPrice(double price) {
  final discount = (price * 0.15).clamp(0.0, 500.0);
  return price - discount;
}

void main() => print(finalPrice(5000)); // 4500.0
```

### №2. Форматирование цены по стандарту валют

```dart
String formatRubles(num value) {
  final fixed = value.toStringAsFixed(2).split('.');
  final integer = fixed[0];
  final grouped = integer.replaceAllMapped(
    RegExp(r'\B(?=(\d{3})+(?!\d))'), (_) => ' ',
  );
  return '$grouped.${fixed[1]} ₽';
}

void main() => print(formatRubles(15400900.5)); // 15 400 900.50 ₽
```

### №3. Расчет кешбэка баллами

```dart
int cashbackPoints(int receiptRubles) {
  if (receiptRubles < 0) throw ArgumentError('Чек не может быть отрицательным');
  return receiptRubles ~/ 100;
}

void main() => print(cashbackPoints(1299)); // 12
```

### №4. Ограничитель диапазона громкости

```dart
double clampVolume(double value) => value.clamp(0.0, 1.0).toDouble();

void main() {
  print(clampVolume(-0.2)); // 0.0
  print(clampVolume(1.4));  // 1.0
}
```

### №5. Конвертер байтов в мегабайты

```dart
String bytesToMegabytes(int bytes) {
  final mb = bytes / (1024 * 1024);
  return '${mb.toStringAsFixed(1)} МБ';
}

void main() => print(bytesToMegabytes(2500000));
```

### №6. Вычисление процентного прогресса

```dart
double progress(int current, int total) {
  if (total <= 0) return 0.0;
  return (current / total).clamp(0.0, 1.0).toDouble();
}

void main() => print(progress(3, 4)); // 0.75
```

### №7. Округление вверх до десятка

```dart
int ceilToTen(int value) => ((value + 9) ~/ 10) * 10;

void main() {
  print(ceilToTen(43)); // 50
  print(ceilToTen(50)); // 50
}
```

### №8. Калькулятор стоимости доставки

```dart
double deliveryCost(double distanceKm) {
  if (distanceKm < 0) throw ArgumentError('Расстояние не может быть отрицательным');
  return 200 + (distanceKm > 5 ? (distanceKm - 5) * 30 : 0);
}

void main() => print(deliveryCost(8)); // 290.0
```

### №9. Проверка делимости на 1024

```dart
bool isWholeBlockSize(int bytes) => bytes >= 0 && bytes % 1024 == 0;

void main() => print(isWholeBlockSize(4096)); // true
```

### №10. Парсер диапазона цен

```dart
({double min, double max}) parsePriceRange(String input) {
  final match = RegExp(r'(\d+(?:\.\d+)?)\s*-\s*(\d+(?:\.\d+)?)').firstMatch(input);
  if (match == null) throw FormatException('Диапазон цен не распознан');
  return (min: double.parse(match[1]!), max: double.parse(match[2]!));
}

void main() => print(parsePriceRange('1500 - 4500 руб'));
```


---

## Смешанные задания — Блок 2. Строки и валидация

### №11. Валидатор пароля

```dart
bool isStrongPassword(String password) =>
    password.length >= 8 &&
    RegExp(r'\d').hasMatch(password) &&
    RegExp(r'[A-Z]').hasMatch(password);

void main() => print(isStrongPassword('Flutter123')); // true
```

### №12. Очистка поискового ввода

```dart
String cleanSearch(String input) => input
    .replaceAll(RegExp(r'[#$%]'), '')
    .replaceAll(RegExp(r'\s+'), ' ')
    .trim()
    .toLowerCase();

void main() => print(cleanSearch('  Dart  ## FLUTTER % '));
```

### №13. Сокращенное имя автора

```dart
String shortAuthorName(String fullName) {
  final parts = fullName.trim().split(RegExp(r'\s+'));
  if (parts.isEmpty || parts.first.isEmpty) return '';
  final surname = parts.first;
  final initials = parts.skip(1).where((p) => p.isNotEmpty)
      .map((p) => '${p[0].toUpperCase()}.').join(' ');
  return initials.isEmpty ? surname : '$surname $initials';
}

void main() => print(shortAuthorName('Иванов Иван Иванович'));
```

### №14. Парсинг HEX-цвета для Flutter

```dart
int parseHexColor(String input) {
  var hex = input.trim().replaceFirst('#', '');
  if (!RegExp(r'^[0-9a-fA-F]{6}$').hasMatch(hex)) {
    throw FormatException('Ожидается HEX-цвет из 6 символов');
  }
  return int.parse('FF$hex', radix: 16);
}

void main() {
  final color = parseHexColor('#FF5722');
  print(color);                                    // 4294923042
  print('0x${color.toRadixString(16).toUpperCase()}'); // 0xFFFF5722
}
```

### №15. Форматирование таймера звонка

```dart
String formatDuration(int seconds) {
  if (seconds < 0) throw ArgumentError('Секунды не могут быть отрицательными');
  final minutes = seconds ~/ 60;
  final remainder = seconds % 60;
  return '${minutes.toString().padLeft(2, '0')}:${remainder.toString().padLeft(2, '0')}';
}

void main() => print(formatDuration(125)); // 02:05
```

### №16. Скрытие номера телефона

```dart
String maskPhone(String phone) {
  final digits = phone.replaceAll(RegExp(r'\D'), '');
  if (digits.length != 11 || !digits.startsWith('7')) return phone;
  return '+7 (${digits.substring(1, 4)}) ***-**-${digits.substring(9)}';
}

void main() => print(maskPhone('+79991234567'));
```

### №17. Определитель кириллицы

```dart
bool containsCyrillic(String text) =>
    text.runes.any((rune) =>
      (rune >= 0x0400 && rune <= 0x04FF) ||
      (rune >= 0x0500 && rune <= 0x052F));

void main() => print(containsCyrillic('Flutter и Dart')); // true
```

### №18. Генератор слага (Slugify)

```dart
String slugifyRussian(String input) {
  const transliteration = {
    'а':'a','б':'b','в':'v','г':'g','д':'d','е':'e','ё':'yo','ж':'zh',
    'з':'z','и':'i','й':'i','к':'k','л':'l','м':'m','н':'n','о':'o',
    'п':'p','р':'r','с':'s','т':'t','у':'u','ф':'f','х':'kh','ц':'ts',
    'ч':'ch','ш':'sh','щ':'shch','ъ':'','ы':'y','ь':'','э':'e','ю':'yu','я':'ya'
  };
  final out = StringBuffer();
  for (final rune in input.toLowerCase().runes) {
    final ch = String.fromCharCode(rune);
    if (transliteration.containsKey(ch)) {
      out.write(transliteration[ch]);
    } else if (RegExp(r'[a-z0-9]').hasMatch(ch)) {
      out.write(ch);
    } else if (ch.trim().isEmpty || RegExp(r'[^a-z0-9]').hasMatch(ch)) {
      out.write('-');
    }
  }
  return out.toString().replaceAll(RegExp(r'-+'), '-').replaceAll(RegExp(r'^-|-$'), '');
}

void main() => print(slugifyRussian('Быстрый старт во Flutter!'));
```

### №19. Подсчет слов в заметке

```dart
int countWords(String note) {
  final trimmed = note.trim();
  if (trimmed.isEmpty) return 0;
  return trimmed.split(RegExp(r'\s+')).length;
}

void main() => print(countWords('Одна строка\nи вторая строка'));
```

### №20. Выделение хештегов из текста

```dart
List<String> extractHashtags(String text) =>
    RegExp(r'#[\p{L}\p{N}_]+', unicode: true)
        .allMatches(text).map((m) => m.group(0)!).toList();

void main() => print(extractHashtags('Пишу про #Flutter и #Dart_3'));
```


---

## Смешанные задания — Блок 3. Списки и фильтрация

### №21. Фильтр каталога по диапазону цен

```dart
List<Map<String, dynamic>> filterByPrice(
  List<Map<String, dynamic>> products, double min, double max,
) => products.where((p) {
  final price = p['price'];
  return price is num && price >= min && price <= max;
}).toList();

void main() => print(filterByPrice([
  {'name':'Телефон','price':50000}, {'name':'Чехол','price':500}
], 1000, 60000));
```

### №22. Удаление завершенных задач

```dart
List<Map<String, dynamic>> unfinishedTasks(
  List<Map<String, dynamic>> tasks,
) => tasks.where((task) => task['isCompleted'] == false).toList();

void main() => print(unfinishedTasks([
  {'title':'Учиться','isCompleted':false},
  {'title':'Отдых','isCompleted':true},
]));
```

### №23. Инверсия порядка чатов

```dart
Iterable<T> reversedMessages<T>(List<T> messages) => messages.reversed;

void main() => print(reversedMessages(['Первое', 'Второе', 'Третье']));
```

### №24. Склейка двух списков баннеров

```dart
List<String> mergeBanners(
  List<String> systemPromos, List<String> personalOffers,
) => [...systemPromos, ...personalOffers];

void main() => print(mergeBanners(['Скидка'], ['Для вас']));
```

### №25. Поиск товара с максимальным рейтингом

```dart
Map<String, dynamic> topRatedProduct(List<Map<String, dynamic>> products) {
  if (products.isEmpty) throw ArgumentError('Список товаров пуст');
  return products.reduce((a, b) =>
      (a['rating'] as num) >= (b['rating'] as num) ? a : b);
}

void main() => print(topRatedProduct([
  {'name':'A','rating':4.2}, {'name':'B','rating':4.8}
]));
```

### №26. Вставка рекламного баннера после каждого 5-го элемента

```dart
List<T> insertAdBanners<T>(List<T> feed, T banner) {
  final result = <T>[];
  for (var i = 0; i < feed.length; i++) {
    result.add(feed[i]);
    if ((i + 1) % 5 == 0 && i != feed.length - 1) result.add(banner);
  }
  return result;
}

void main() => print(insertAdBanners<int>(List.generate(12, (i) => i + 1), 0));
```

### №27. Проверка идентичности двух списков

```dart
bool sameElementsAtSamePositions<T>(List<T> a, List<T> b) {
  if (a.length != b.length) return false;
  for (var i = 0; i < a.length; i++) {
    if (a[i] != b[i]) return false;
  }
  return true;
}

void main() => print(sameElementsAtSamePositions([1, 2], [1, 2]));
```

### №28. Извлечение уникальных категорий

```dart
List<String> uniqueCategories(List<Map<String, dynamic>> products) =>
    products.map((p) => p['category'])
        .whereType<String>().toSet().toList();

void main() => print(uniqueCategories([
  {'category':'Электроника'}, {'category':'Книги'}, {'category':'Книги'}
]));
```

### №29. Сортировка по доступности и цене

```dart
void sortProducts(List<Map<String, dynamic>> products) {
  products.sort((a, b) {
    final availableA = a['inStock'] == true;
    final availableB = b['inStock'] == true;
    if (availableA != availableB) return availableA ? -1 : 1;
    return (a['price'] as num).compareTo(b['price'] as num);
  });
}

void main() {
  final products = <Map<String, dynamic>>[
    {'name':'A','inStock':false,'price':100},
    {'name':'B','inStock':true,'price':500},
    {'name':'C','inStock':true,'price':300},
  ];
  sortProducts(products);
  print(products);
}
```

### №30. Постраничная группировка по 10 элементов

```dart
List<List<T>> chunk<T>(List<T> items, int size) {
  if (size <= 0) throw ArgumentError('Размер пачки должен быть положительным');
  return [
    for (var start = 0; start < items.length; start += size)
      items.sublist(start, (start + size < items.length) ? start + size : items.length)
  ];
}

void main() => print(chunk(List.generate(100, (i) => i + 1), 10).length); // 10
```


---

## Смешанные задания — Блок 4. Map, JSON и кэш

### №31. Десериализация профиля

```dart
({int id, String name, bool isPremium}) parseProfile(Map<String, dynamic> json) {
  return (
    id: json['id'] is int ? json['id'] as int : 0,
    name: json['name'] is String ? json['name'] as String : 'Без имени',
    isPremium: json['isPremium'] is bool ? json['isPremium'] as bool : false,
  );
}

void main() => print(parseProfile({'id': 7, 'name': 'Аня'}));
```

### №32. Преобразование List<User> в Map по id

```dart
class User {
  final int id;
  final String name;
  User(this.id, this.name);
}

Map<int, User> usersById(List<User> users) =>
    {for (final user in users) user.id: user};

void main() {
  final map = usersById([User(1, 'Аня'), User(2, 'Иван')]);
  print(map[1]?.name);
}
```

### №33. Слияние локальных и серверных настроек

```dart
Map<String, dynamic> mergeSettings(
  Map<String, dynamic> local, Map<String, dynamic> server,
) => {...local, ...server};

void main() => print(mergeSettings(
  {'theme':'light','sound':true}, {'theme':'dark'}
));
```

### №34. Удаление ключей с null-значениями

```dart
Map<String, dynamic> removeNullValues(Map<String, dynamic> input) =>
    Map.fromEntries(input.entries.where((entry) => entry.value != null));

void main() => print(removeNullValues({'name':'Ира','phone':null}));
```

### №35. Простейший LRU-кэш последних 5 страниц

```dart
class PageCache {
  final Map<String, Object?> _pages = {};

  void open(String page, Object? data) {
    _pages.remove(page); // повторное открытие делает страницу самой свежей
    _pages[page] = data;
    if (_pages.length > 5) _pages.remove(_pages.keys.first);
  }

  List<String> get pagesOldestFirst => _pages.keys.toList();
}

void main() {
  final cache = PageCache();
  for (var i = 1; i <= 6; i++) { cache.open('page$i', i); }
  print(cache.pagesOldestFirst); // page2 ... page6
}
```

### №36. Инверсия карты

```dart
Map<String, String> invertMap(Map<String, String> source) {
  final result = <String, String>{};
  for (final entry in source.entries) {
    if (result.containsKey(entry.value)) {
      throw ArgumentError('Значения должны быть уникальны: ${entry.value}');
    }
    result[entry.value] = entry.key;
  }
  return result;
}

void main() => print(invertMap({'en':'Hello','ru':'Привет'}));
```

### №37. Сумма покупок по категориям

```dart
Map<String, double> totalsByCategory(
  List<Map<String, dynamic>> purchases,
) {
  final totals = <String, double>{};
  for (final item in purchases) {
    final category = item['category'];
    final price = item['price'];
    if (category is String && price is num) {
      totals[category] = (totals[category] ?? 0) + price.toDouble();
    }
  }
  return totals;
}

void main() => print(totalsByCategory([
  {'category':'Еда','price':200}, {'category':'Еда','price':150}
]));
```

### №38. Проверка обязательных JSON-полей

```dart
bool hasRequiredFields(Map<String, dynamic> response) =>
    response.containsKey('status') &&
    response.containsKey('data') &&
    response.containsKey('timestamp');

void main() => print(hasRequiredFields({
  'status':'ok','data':[], 'timestamp':DateTime.now().toIso8601String()
}));
```

### №39. Построение дерева категорий

```dart
Map<int, List<Map<String, dynamic>>> buildCategoryTree(
  List<Map<String, dynamic>> categories,
) {
  final tree = <int, List<Map<String, dynamic>>>{};
  for (final category in categories) {
    final parentId = category['parentId'];
    if (parentId is int) tree.putIfAbsent(parentId, () => []).add(category);
  }
  return tree; // ключ 0 или специальный id может обозначать корневые категории
}

void main() => print(buildCategoryTree([
  {'id':1,'name':'Техника','parentId':0},
  {'id':2,'name':'Телефоны','parentId':1}
]));
```

### №40. Сериализация параметров фильтра в URL

```dart
String queryString(Map<String, Object?> params) =>
    params.entries.map((e) =>
      '${Uri.encodeQueryComponent(e.key)}=${Uri.encodeQueryComponent('${e.value}')}')
      .join('&');

void main() => print(queryString({'brand':'Apple','minPrice':50000}));
```


---

## Смешанные задания — Блок 5. Null Safety, Records и пайплайны

### №41. Парсер геолокации через Record

```dart
({double lat, double lon})? parseCoordinates(String input) {
  final parts = input.split(',');
  if (parts.length != 2) return null;
  final lat = double.tryParse(parts[0].trim());
  final lon = double.tryParse(parts[1].trim());
  if (lat == null || lon == null || lat < -90 || lat > 90 || lon < -180 || lon > 180) {
    return null;
  }
  return (lat: lat, lon: lon);
}

void main() => print(parseCoordinates('54.01, 38.29'));
```

### №42. Резолвер состояний загрузки через Pattern Matching

```dart
String loadingMessage(({bool isLoading, bool hasError}) state) =>
    switch (state) {
      (isLoading: true, hasError: false) => 'Загрузка...',
      (isLoading: false, hasError: true) => 'Произошла ошибка',
      (isLoading: false, hasError: false) => 'Данные загружены',
      _ => 'Состояние не определено',
    };

void main() {
  print(loadingMessage((isLoading: true, hasError: false)));
  print(loadingMessage((isLoading: false, hasError: true)));
}
```

### №43. Безопасная цепочка вызовов вложенного объекта

```dart
String readStreet(Map<String, dynamic>? user) =>
    user?['company']?['address']?['street'] ?? 'Адрес не указан';

void main() {
  print(readStreet({
    'company': {'address': {'street': 'Ленина'}}
  })); // Ленина
  print(readStreet(null));                  // Адрес не указан
  print(readStreet({'company': {}}));       // Адрес не указан
}
```

### №44. Сопоставление типа ответа API

```dart
String handleApiResponse(Object? response) => switch (response) {
  int code => 'HTTP-код: $code',
  String error => 'Ошибка: $error',
  Map data => 'Получены данные: ${data.length} полей',
  _ => 'Неизвестный тип ответа',
};

void main() {
  print(handleApiResponse(200));
  print(handleApiResponse('Сервер недоступен'));
  print(handleApiResponse({'ok': true}));
}
```

### №45. Деструктуризация координат в цикле

```dart
void main() {
  final points = <(int x, int y)>[(0, 0), (10, 20), (-3, 7)];
  for (final (x, y) in points) {
    print('x=$x, y=$y');
  }
}
```

### №46. Обмен значений через Record

```dart
void main() {
  var a = 10;
  var b = 20;
  (a, b) = (b, a);
  print('a=$a, b=$b'); // a=20, b=10
}
```

### №47. Валидатор формы регистрации через Record

```dart
({bool isValid, List<String> errors}) validateRegistration({
  required String email,
  required String password,
  required String confirmation,
}) {
  final errors = <String>[];
  if (!RegExp(r'^[^@\s]+@[^@\s]+\.[^@\s]+$').hasMatch(email)) {
    errors.add('Укажите корректный email');
  }
  if (password.length < 8) errors.add('Пароль должен содержать минимум 8 символов');
  if (password != confirmation) errors.add('Пароли не совпадают');
  return (isValid: errors.isEmpty, errors: errors);
}

void main() => print(validateRegistration(
  email: 'user@example.com', password: '12345678', confirmation: '12345678',
));
```

### №48. Проверка диапазона громкости через Switch Guard

```dart
String volumeMessage(Object? value) => switch (value) {
  int v when v > 80 => 'Слишком громко!',
  int v when v >= 0 && v <= 80 => 'Громкость допустима',
  int _ => 'Громкость вне допустимого диапазона',
  _ => 'Уровень громкости должен быть целым числом',
};

void main() {
  print(volumeMessage(90));
  print(volumeMessage(50));
}
```

### №49. Очистка телефонной книги

```dart
Set<String> cleanPhoneBook(List<String?> entries) {
  final result = <String>{};
  for (final entry in entries) {
    if (entry == null) continue;
    final normalized = entry.replaceAll(RegExp(r'[^0-9+]'), '');
    if (normalized.isNotEmpty) result.add(normalized);
  }
  return result;
}

void main() => print(cleanPhoneBook([
  '+7 (999) 123-45-67', null, ' ', '+7 999 123-45-67'
]));
```

### №50. Итоговый аналитический модуль корзины

```dart
({double total, double discountAmount, bool isFreeDelivery}) analyzeCart(
  List<Map<String, dynamic>> cart,
) {
  double gross = 0;
  double total = 0;
  for (final item in cart) {
    final price = item['price'];
    final quantity = item['quantity'];
    final discount = item['discountPercent'] ?? 0;
    if (price is! num || quantity is! int || discount is! num) continue;
    if (price < 0 || quantity < 0) continue;
    final lineGross = price.toDouble() * quantity;
    final safeDiscount = discount.clamp(0, 100).toDouble();
    gross += lineGross;
    total += lineGross * (1 - safeDiscount / 100);
  }
  final discountAmount = gross - total;
  return (
    total: total,
    discountAmount: discountAmount,
    isFreeDelivery: total >= 2000,
  );
}

void main() {
  final result = analyzeCart([
    {'name':'Телефон','price':1800,'quantity':1,'discountPercent':10},
    {'name':'Чехол','price':300,'quantity':1,'discountPercent':0},
  ]);
  print(result);
}
```
