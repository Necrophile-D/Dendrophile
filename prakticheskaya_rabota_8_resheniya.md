# Практическая работа №8 — решения заданий (ООП в Dart)

> Каждый блок — отдельная программа со своим `main()`. Вставляй по одному в [DartPad](https://dartpad.dev), жми **Run**, вывод появится справа. Перед следующим блоком очищай поле (Ctrl+A → вставить новый код). Нужен Dart 3+ (в DartPad он по умолчанию).
>
> Примечания:
> - DartPad запускает код с включёнными `assert`, поэтому проверки в списках инициализации срабатывают. При запуске через `dart run` для этого нужен флаг `--enable-asserts`.
> - Там, где в задании упомянуты Flutter-классы (`ChangeNotifier`, виджеты, контроллеры анимации), они заменены простыми аналогами на чистом Dart, чтобы код запускался в обычном DartPad без Flutter.

---

## Тема 1. Классы, инкапсуляция, методы и геттеры/сеттеры

### №1. Инкапсулированный термостат

```dart
class Thermostat {
  double _temperatureCelsius = 20.0;

  double get temperatureCelsius => _temperatureCelsius;

  // Сеттер с ограничением диапазона от -50 до +100 °C
  set temperatureCelsius(double value) {
    if (value < -50 || value > 100) {
      throw RangeError('Температура должна быть от -50 до +100 °C, получено: $value');
    }
    _temperatureCelsius = value;
  }

  // Геттер: температура в градусах Фаренгейта
  double get temperatureFahrenheit => _temperatureCelsius * 9 / 5 + 32;
}

void main() {
  final t = Thermostat();
  t.temperatureCelsius = 25;
  print('25 °C = ${t.temperatureFahrenheit.toStringAsFixed(1)} °F'); // 77.0 °F

  try {
    t.temperatureCelsius = 150;
  } on RangeError catch (e) {
    print('Ошибка: ${e.message}');
  }
  print('Текущая температура: ${t.temperatureCelsius} °C'); // осталась 25.0
}
```

### №2. Счетчик шагов с защитой от скручивания

```dart
class Pedometer {
  int _steps = 0;

  int get steps => _steps;

  // Шаги можно только увеличивать
  void addSteps(int count) {
    if (count <= 0) {
      throw ArgumentError('Шаги можно только добавлять (count > 0), получено: $count');
    }
    _steps += count;
  }
}

void main() {
  final p = Pedometer();
  p.addSteps(1000);
  p.addSteps(250);
  print('Шагов: ${p.steps}'); // 1250

  try {
    p.addSteps(-500); // попытка скрутить счетчик
  } on ArgumentError catch (e) {
    print('Ошибка: ${e.message}');
  }
  print('Шагов после попытки: ${p.steps}'); // 1250
}
```

### №3. Модель книги с расчетом времени чтения

```dart
class Book {
  final String title;
  final int pages;

  Book(this.title, this.pages);

  // 1 страница = 2 минуты
  double get estimatedReadingHours => pages * 2 / 60;

  @override
  String toString() => '«$title», $pages стр.';
}

void main() {
  final book = Book('Война и мир', 1225);
  print(book);
  print('Время чтения: ${book.estimatedReadingHours.toStringAsFixed(1)} ч.'); // 40.8 ч.
}
```

### №4. Валидатор пароля в сеттере

```dart
class UserCredentials {
  final String login;
  String _passwordHash = '';

  UserCredentials(this.login);

  // Сеттер принимает сырую строку, проверяет длину и сохраняет псевдо-хэш
  set password(String raw) {
    if (raw.length < 8) {
      throw ArgumentError('Пароль должен содержать минимум 8 символов');
    }
    _passwordHash = _fakeHash(raw);
  }

  bool get hasPassword => _passwordHash.isNotEmpty;
  String get passwordHash => _passwordHash;

  bool checkPassword(String raw) => _fakeHash(raw) == _passwordHash;

  // Учебный псевдо-хэш (в реальных проектах используют bcrypt/argon2)
  static String _fakeHash(String raw) {
    var h = 7;
    for (final unit in raw.codeUnits) {
      h = (h * 31 + unit) & 0x7FFFFFFF;
    }
    return h.toRadixString(16).padLeft(8, '0');
  }
}

void main() {
  final user = UserCredentials('anna');

  try {
    user.password = 'short';
  } on ArgumentError catch (e) {
    print('Ошибка: ${e.message}');
  }

  user.password = 'StrongPass123';
  print('Хэш сохранён: ${user.passwordHash}');
  print('Верный пароль: ${user.checkPassword('StrongPass123')}'); // true
  print('Неверный пароль: ${user.checkPassword('wrongpass1')}');  // false
}
```

### №5. Статический счетчик экземпляров

```dart
class DeviceSession {
  // Общее для всего класса поле
  static int activeSessionsCount = 0;

  final String deviceName;
  bool _closed = false;

  DeviceSession(this.deviceName) {
    activeSessionsCount++;
  }

  void close() {
    if (!_closed) {
      _closed = true;
      activeSessionsCount--;
    }
  }
}

void main() {
  print('Старт: ${DeviceSession.activeSessionsCount}'); // 0
  final s1 = DeviceSession('Phone');
  final s2 = DeviceSession('Tablet');
  final s3 = DeviceSession('Laptop');
  print('Создано три сессии: ${DeviceSession.activeSessionsCount}'); // 3
  s2.close();
  print('После закрытия одной: ${DeviceSession.activeSessionsCount}'); // 2
  s1.close();
  s3.close();
  print('Все закрыты: ${DeviceSession.activeSessionsCount}'); // 0
}
```

---

## Тема 2. Все виды конструкторов

### №1. Именованные конструкторы цвета

```dart
class AppColor {
  final int r, g, b;
  final double opacity;

  // Именованный конструктор со списком инициализации
  AppColor.fromRGBO(this.r, this.g, this.b, this.opacity)
      : assert(r >= 0 && r <= 255 && g >= 0 && g <= 255 && b >= 0 && b <= 255),
        assert(opacity >= 0 && opacity <= 1);

  // Фабричный конструктор: разбор строки вида "#FF5722"
  factory AppColor.fromHex(String hex) {
    final clean = hex.trim().replaceFirst('#', '');
    if (!RegExp(r'^[0-9a-fA-F]{6}$').hasMatch(clean)) {
      throw FormatException('Ожидается HEX-цвет из 6 символов, получено: $hex');
    }
    return AppColor.fromRGBO(
      int.parse(clean.substring(0, 2), radix: 16),
      int.parse(clean.substring(2, 4), radix: 16),
      int.parse(clean.substring(4, 6), radix: 16),
      1.0,
    );
  }

  @override
  String toString() => 'rgba($r, $g, $b, $opacity)';
}

void main() {
  print(AppColor.fromHex('#FF5722'));              // rgba(255, 87, 34, 1.0)
  print(AppColor.fromRGBO(10, 20, 30, 0.5));       // rgba(10, 20, 30, 0.5)
}
```

### №2. Константная точка на плоскости

```dart
class Point2D {
  final double x, y;

  const Point2D(this.x, this.y);

  @override
  String toString() => 'Point2D($x, $y)';
}

void main() {
  const p1 = Point2D(1, 2);
  const p2 = Point2D(1, 2);
  print('identical(p1, p2): ${identical(p1, p2)}'); // true — один и тот же объект

  final p3 = Point2D(1, 2); // без const — новый объект во время выполнения
  print('identical(p1, p3): ${identical(p1, p3)}'); // false
}
```

### №3. Список инициализации с assert

```dart
class Rectangle {
  final double width;
  final double height;

  Rectangle(this.width, this.height)
      : assert(width > 0 && height > 0, 'Стороны должны быть положительными');

  double get area => width * height;

  @override
  String toString() => 'Rectangle(${width}x$height), площадь $area';
}

void main() {
  print(Rectangle(4, 5)); // Rectangle(4.0x5.0), площадь 20.0

  try {
    final bad = Rectangle(-1, 5);
    print('assert не сработал (asserts выключены): $bad');
  } on AssertionError catch (e) {
    print('Сработал assert: ${e.message}');
  }
}
```

### №4. Фабричный синглтон сетевого клиента

```dart
class ApiClient {
  // Единственный приватный экземпляр
  static final ApiClient _instance = ApiClient._internal();

  final String baseUrl;

  ApiClient._internal() : baseUrl = 'https://api.example.com' {
    print('ApiClient создан (один раз)');
  }

  // Фабричный конструктор всегда возвращает один и тот же объект
  factory ApiClient() => _instance;

  void get(String path) => print('GET $baseUrl$path');
}

void main() {
  final a = ApiClient();
  final b = ApiClient();
  print('Один и тот же объект: ${identical(a, b)}'); // true
  a.get('/users');
}
```

### №5. Редирект конструктора

```dart
import 'dart:math';

class Circle {
  final double radius;

  Circle(this.radius);

  // Redirecting constructor: перенаправляет вызов на основной
  Circle.unit() : this(1.0);

  double get area => pi * radius * radius;

  @override
  String toString() => 'Circle(r=$radius), площадь ${area.toStringAsFixed(2)}';
}

void main() {
  print(Circle(2.5));
  print(Circle.unit()); // Circle(r=1.0), площадь 3.14
}
```

---

## Тема 3. Наследование, полиморфизм и super

### №1. Иерархия банковских карт

```dart
class Card {
  final String number;
  double balance;

  Card(this.number, this.balance);

  bool canPay(double amount) => amount > 0 && balance >= amount;

  bool pay(double amount) {
    if (!canPay(amount)) {
      print('[$number] Отказ: платёж $amount ₽ невозможен (баланс $balance ₽)');
      return false;
    }
    balance -= amount;
    print('[$number] Оплачено $amount ₽, баланс $balance ₽');
    return true;
  }
}

// Дебетовая карта: без овердрафта (поведение базового класса)
class DebitCard extends Card {
  DebitCard(super.number, super.balance);
}

// Кредитная карта: можно уходить в минус до лимита овердрафта
class CreditCard extends Card {
  final double overdraftLimit;

  CreditCard(super.number, super.balance, {required this.overdraftLimit});

  @override
  bool canPay(double amount) => amount > 0 && balance + overdraftLimit >= amount;
}

void main() {
  final debit = DebitCard('4276-0001', 1000);
  final credit = CreditCard('5469-0002', 1000, overdraftLimit: 5000);

  debit.pay(1500);  // отказ
  credit.pay(1500); // ок, баланс -500
  credit.pay(5000); // отказ: превышен лимит
}
```

### №2. Полиморфный список виджетов

```dart
abstract class BaseScreen {
  final String name;
  BaseScreen(this.name);

  void render();
}

class HomeScreen extends BaseScreen {
  HomeScreen() : super('Home');

  @override
  void render() => print('[$name] Лента новостей');
}

class ProfileScreen extends BaseScreen {
  ProfileScreen() : super('Profile');

  @override
  void render() => print('[$name] Аватар и данные пользователя');
}

class SettingsScreen extends BaseScreen {
  SettingsScreen() : super('Settings');

  @override
  void render() => print('[$name] Переключатели настроек');
}

void main() {
  final screens = <BaseScreen>[HomeScreen(), ProfileScreen(), SettingsScreen()];

  // Полиморфизм: render() вызывается без проверки конкретного типа
  for (final screen in screens) {
    screen.render();
  }
}
```

### №3. Использование super-параметров (Dart 2.17+)

```dart
class BaseModel {
  final int id;
  final String? title;

  BaseModel({required this.id, this.title});
}

// Старый способ:
// class OldItem extends BaseModel {
//   final double price;
//   OldItem({required int id, String? title, required this.price})
//       : super(id: id, title: title);
// }

// Новый компактный синтаксис с super-параметрами:
class Item extends BaseModel {
  final double price;

  Item({required super.id, super.title, required this.price});

  @override
  String toString() => 'Item(id=$id, title=$title, price=$price)';
}

void main() {
  print(Item(id: 1, title: 'Кроссовки', price: 7990));
  print(Item(id: 2, price: 100)); // title не передан → null
}
```

### №4. Ковариантный параметр метода (`covariant`)

```dart
class Food {}

class Grass extends Food {}

class Meat extends Food {}

class Animal {
  void eat(Food food) => print('Животное ест еду');
}

class Cow extends Animal {
  // covariant позволяет сузить тип аргумента: Food → Grass
  @override
  void eat(covariant Grass food) => print('Корова ест траву');
}

void main() {
  Animal cow = Cow();
  cow.eat(Grass()); // Корова ест траву

  try {
    cow.eat(Meat()); // компилятор пропускает, проверка — во время выполнения
  } catch (e) {
    print('Ошибка типа во время выполнения: корова не ест мясо');
  }
}
```

### №5. Предотвращение переопределения через `@nonVirtual`

```dart
// В обычном проекте аннотация берётся из пакета meta:
//   import 'package:meta/meta.dart';
// Для запуска в DartPad без пакетов объявляем её сами:
class _NonVirtual {
  const _NonVirtual();
}

const nonVirtual = _NonVirtual();

class FeatureGate {
  // Критическая проверка лицензии: переопределять нельзя.
  // С настоящей аннотацией meta анализатор выдаст предупреждение
  // invalid_override_of_non_virtual_member при попытке переопределить.
  @nonVirtual
  bool verifyLicense(String key) => key.startsWith('LIC-') && key.length == 12;

  void openFeature(String key) {
    print(verifyLicense(key) ? 'Функция открыта' : 'Нет действующей лицензии');
  }
}

class PremiumFeature extends FeatureGate {
  // verifyLicense здесь переопределять не нужно и нельзя
  @override
  void openFeature(String key) {
    print('Премиум-раздел:');
    super.openFeature(key);
  }
}

void main() {
  final f = PremiumFeature();
  f.openFeature('LIC-12345678'); // 12 символов → открыта
  f.openFeature('HACK');         // нет лицензии
}
```

---

## Тема 4. Абстрактные классы, интерфейсы и Sealed-классы

### №1. Контракт локального кэша

```dart
abstract interface class StorageService {
  void save(String key, String val);
  String? read(String key);
  void delete(String key);
}

// implements: обязаны переопределить все методы
class MemoryStorageService implements StorageService {
  final Map<String, String> _data = {};

  @override
  void save(String key, String val) => _data[key] = val;

  @override
  String? read(String key) => _data[key];

  @override
  void delete(String key) => _data.remove(key);
}

void main() {
  final StorageService storage = MemoryStorageService();
  storage.save('token', 'abc123');
  print(storage.read('token')); // abc123
  storage.delete('token');
  print(storage.read('token')); // null
}
```

### №2. Множественная реализация интерфейсов

```dart
abstract interface class Displayable {
  void showImage(String name);
}

abstract interface class NetworkConnectable {
  void connect(String ssid);
}

// Один класс реализует сразу два несвязанных интерфейса
class SmartTv implements Displayable, NetworkConnectable {
  @override
  void showImage(String name) => print('Телевизор показывает: $name');

  @override
  void connect(String ssid) => print('Телевизор подключён к Wi-Fi "$ssid"');
}

void main() {
  final tv = SmartTv();
  tv.connect('Home_5G');
  tv.showImage('Заставка.png');

  Displayable d = tv; // можно использовать как любой из интерфейсов
  d.showImage('Фото.jpg');
}
```

### №3. Sealed-иерархия авторизации

```dart
class User {
  final String name;
  const User(this.name);
}

sealed class AuthState {
  const AuthState();
}

class Unauthenticated extends AuthState {
  const Unauthenticated();
}

class Authenticating extends AuthState {
  const Authenticating();
}

class Authenticated extends AuthState {
  final User user;
  const Authenticated(this.user);
}

class AuthFailed extends AuthState {
  final String reason;
  const AuthFailed(this.reason);
}

// switch без default: компилятор проверяет, что учтены ВСЕ подтипы
String describe(AuthState state) => switch (state) {
      Unauthenticated() => 'Войдите в аккаунт',
      Authenticating() => 'Выполняется вход...',
      Authenticated(:final user) => 'Привет, ${user.name}!',
      AuthFailed(:final reason) => 'Ошибка входа: $reason',
    };

void main() {
  final states = <AuthState>[
    const Unauthenticated(),
    const Authenticating(),
    const Authenticated(User('Анна')),
    const AuthFailed('неверный пароль'),
  ];
  for (final s in states) {
    print(describe(s));
  }
}
```

### №4. Модификатор `base class`

```dart
// base: класс можно только наследовать (extends), но нельзя реализовать как интерфейс
base class Engine {
  void start() => print('Двигатель запущен');
}

// Наследник base-класса тоже обязан иметь модификатор base/final/sealed
base class V8Engine extends Engine {
  @override
  void start() {
    print('V8: прогрев...');
    super.start();
  }
}

// В другой библиотеке (другом файле) следующая строка была бы ошибкой:
// class FakeEngine implements Engine { ... }  // нельзя: Engine — base
// (в одном файле DartPad ограничения для своей же библиотеки не действуют)

void main() {
  Engine e = V8Engine();
  e.start();
}
```

### №5. Модификатор `interface class`

```dart
// interface: вне своей библиотеки класс можно только реализовать (implements),
// но нельзя унаследовать (extends) — реализация не «протекает» к клиентам.
interface class PaymentTerminal {
  void pay(double amount) => print('Базовая оплата $amount ₽');
}

// Клиент библиотеки обязан реализовать контракт сам
class MyTerminal implements PaymentTerminal {
  @override
  void pay(double amount) => print('Своя оплата $amount ₽ через MyTerminal');
}

// В другой библиотеке следующая строка была бы ошибкой:
// class Bad extends PaymentTerminal {}  // нельзя: PaymentTerminal — interface

void main() {
  PaymentTerminal t = MyTerminal();
  t.pay(499);
}
```

---

## Тема 5. Миксины

### №1. Миксин сериализации в JSON

```dart
import 'dart:convert';

mixin JsonPrintable {
  // Класс, использующий миксин, обязан предоставить toJson()
  Map<String, dynamic> toJson();

  void printAsJson() => print(jsonEncode(toJson()));
}

class Product with JsonPrintable {
  final String name;
  final double price;

  Product(this.name, this.price);

  @override
  Map<String, dynamic> toJson() => {'name': name, 'price': price};
}

class Person with JsonPrintable {
  final String name;
  final int age;

  Person(this.name, this.age);

  @override
  Map<String, dynamic> toJson() => {'name': name, 'age': age};
}

void main() {
  Product('Чайник', 2490).printAsJson(); // {"name":"Чайник","price":2490.0}
  Person('Анна', 21).printAsJson();      // {"name":"Анна","age":21}
}
```

### №2. Ограничение миксина на ChangeNotifier

```dart
// Во Flutter ChangeNotifier уже есть (package:flutter/foundation.dart).
// Здесь — упрощённый аналог, чтобы код работал в обычном DartPad.
class ChangeNotifier {
  final List<void Function()> _listeners = [];

  void addListener(void Function() listener) => _listeners.add(listener);

  void notifyListeners() {
    for (final l in List.of(_listeners)) {
      l();
    }
  }

  void dispose() => _listeners.clear();
}

// on ChangeNotifier: миксин можно подмешать только в наследников ChangeNotifier
mixin AutoDisposer on ChangeNotifier {
  bool isDisposed = false;

  @override
  void notifyListeners() {
    if (isDisposed) {
      print('Уведомление проигнорировано: объект уже уничтожен');
      return;
    }
    super.notifyListeners();
  }

  @override
  void dispose() {
    isDisposed = true;
    super.dispose();
  }
}

class CounterModel extends ChangeNotifier with AutoDisposer {
  int value = 0;

  void increment() {
    value++;
    notifyListeners();
  }
}

void main() {
  final model = CounterModel();
  model.addListener(() => print('Слушатель: value = ${model.value}'));

  model.increment(); // Слушатель: value = 1
  model.dispose();
  model.increment(); // уведомление игнорируется
  print('isDisposed = ${model.isDisposed}'); // true
}
```

### №3. Аналитический трекер кликов

```dart
mixin AnalyticsTracker {
  final List<String> trackedEvents = [];

  void trackEvent(String eventName) {
    trackedEvents.add(eventName);
    print('[Analytics] $eventName');
  }
}

class CustomButton with AnalyticsTracker {
  final String label;

  CustomButton(this.label);

  void click() {
    trackEvent('button_click:$label');
    print('Нажата кнопка "$label"');
  }
}

void main() {
  final button = CustomButton('Купить');
  button.click();
  button.click();
  print('Всего событий: ${button.trackedEvents.length}'); // 2
}
```

### №4. Порядок линеаризации миксинов

```dart
mixin MixinA {
  String execute() => 'MixinA';
}

mixin MixinB {
  String execute() => 'MixinB';
}

// Правый миксин имеет приоритет: побеждает MixinB (указан последним)
class FirstCase with MixinA, MixinB {}

class SecondCase with MixinB, MixinA {}

// Цепочка вызовов через super показывает порядок линеаризации
class Base {
  String execute() => 'Base';
}

mixin LoggerA on Base {
  @override
  String execute() => 'A -> ${super.execute()}';
}

mixin LoggerB on Base {
  @override
  String execute() => 'B -> ${super.execute()}';
}

class Chain extends Base with LoggerA, LoggerB {}

void main() {
  print(FirstCase().execute());  // MixinB
  print(SecondCase().execute()); // MixinA
  print(Chain().execute());      // B -> A -> Base
}
```

### №5. Миксин кэширования вычислений

```dart
mixin MemoizationMixin {
  final Map<String, Object?> _memo = {};
  int cacheHits = 0;

  // Результат хранится в словаре по строковому ключу аргументов
  T memoize<T>(String key, T Function() compute) {
    if (_memo.containsKey(key)) {
      cacheHits++;
      return _memo[key] as T;
    }
    final value = compute();
    _memo[key] = value;
    return value;
  }

  int get cacheSize => _memo.length;
}

class MathService with MemoizationMixin {
  int fib(int n) {
    if (n < 2) return n;
    return memoize<int>('fib:$n', () => fib(n - 1) + fib(n - 2));
  }

  int slowSquareSum(int n) => memoize<int>('sqsum:$n', () {
        var sum = 0;
        for (var i = 1; i <= n; i++) {
          sum += i * i;
        }
        return sum;
      });
}

void main() {
  final math = MathService();
  print('fib(40) = ${math.fib(40)}');           // 102334155
  print('Размер кэша: ${math.cacheSize}');
  print('sqsum(1000) = ${math.slowSquareSum(1000)}'); // 333833500
  print('Повтор: ${math.slowSquareSum(1000)}');       // из кэша
  print('Попаданий в кэш: ${math.cacheHits}');
}
```

---

## Смешанные задания — Блок 1. Модели данных, DTO и конструкторы

### №1. Неизменяемая карточка товара (Immutable Product)

```dart
class Product {
  final String id;
  final String name;
  final double price;
  final double discountPercent;

  // const-конструктор + валидация в списке инициализации
  const Product({
    required this.id,
    required this.name,
    required this.price,
    this.discountPercent = 0,
  })  : assert(price >= 0, 'Стоимость не может быть отрицательной'),
        assert(discountPercent >= 0 && discountPercent <= 100,
            'Скидка должна быть от 0 до 100%');

  double get discountedPrice => price * (1 - discountPercent / 100);

  @override
  String toString() =>
      '$name: $price ₽ → ${discountedPrice.toStringAsFixed(2)} ₽ (скидка $discountPercent%)';
}

void main() {
  const phone = Product(id: 'p1', name: 'Смартфон', price: 30000, discountPercent: 15);
  print(phone); // Смартфон: 30000.0 ₽ → 25500.00 ₽ (скидка 15.0%)

  try {
    Product(id: 'p2', name: 'Ошибка', price: -10);
  } on AssertionError catch (e) {
    print('Ошибка: ${e.message}');
  }
}
```

### №2. Парсер профиля пользователя с безопасными дефолтами

```dart
class User {
  final int id;
  final String name;
  final int? age;
  final String email;

  const User({required this.id, required this.name, this.age, required this.email});

  factory User.fromJson(Map<String, dynamic> json) {
    final id = json['id'];
    final name = json['name'];
    final age = json['age'];
    final email = json['email'];

    return User(
      // Проверяем типы через is — никаких TypeError
      id: id is int ? id : int.tryParse('$id') ?? 0,
      name: (name is String && name.trim().isNotEmpty) ? name : 'Без имени',
      age: age is num ? age.toInt() : null,
      email: email is String ? email : 'не указан',
    );
  }

  @override
  String toString() => 'User(id=$id, name=$name, age=$age, email=$email)';
}

void main() {
  print(User.fromJson({'id': 7, 'name': 'Анна', 'age': 21, 'email': 'a@mail.ru'}));
  print(User.fromJson({'id': '15', 'name': '', 'age': 'двадцать'}));
  print(User.fromJson({}));
}
```

### №3. Multiton локалей приложения

```dart
class AppLocale {
  static final Map<String, AppLocale> _instances = {};

  final String code;

  AppLocale._(this.code);

  // Фабрика возвращает единственный экземпляр для каждого языка
  factory AppLocale(String code) {
    final key = code.toLowerCase();
    return _instances.putIfAbsent(key, () => AppLocale._(key));
  }

  @override
  String toString() => 'AppLocale($code)';
}

void main() {
  final ru1 = AppLocale('ru');
  final ru2 = AppLocale('RU');
  final en = AppLocale('en');

  print('ru1 и ru2 — один объект: ${identical(ru1, ru2)}'); // true
  print('ru и en — разные объекты: ${!identical(ru1, en)}'); // true
  print(AppLocale._instances.keys.toList()); // [ru, en]
}
```

### №4. Модель геоточки с парсингом строки

```dart
class GeoPoint {
  final double lat;
  final double lon;

  const GeoPoint(this.lat, this.lon);

  // Именованный (фабричный) конструктор из строки "55.751244, 37.618423"
  factory GeoPoint.fromString(String input) {
    final parts = input.split(',');
    if (parts.length != 2) {
      throw FormatException('Ожидается формат "широта, долгота": $input');
    }
    final lat = double.tryParse(parts[0].trim());
    final lon = double.tryParse(parts[1].trim());
    if (lat == null || lon == null) {
      throw FormatException('Координаты должны быть числами: $input');
    }
    if (lat < -90 || lat > 90 || lon < -180 || lon > 180) {
      throw RangeError('Координаты вне допустимого диапазона: $input');
    }
    return GeoPoint(lat, lon);
  }

  @override
  String toString() => 'GeoPoint($lat, $lon)';
}

void main() {
  print(GeoPoint.fromString('55.751244, 37.618423'));

  try {
    GeoPoint.fromString('abc, 10');
  } on FormatException catch (e) {
    print('Ошибка: ${e.message}');
  }
}
```

### №5. Конструктор визитки из сущности БД

```dart
class ContactCard {
  final int id;
  final String fullName;
  final String? phone;
  final String? email;

  const ContactCard({required this.id, required this.fullName, this.phone, this.email});

  // Строка БД — Map<String, Object?>: значения могут быть любого типа или null
  factory ContactCard.fromDatabaseRecord(Map<String, Object?> row) {
    final id = row['id'];
    final first = row['first_name'];
    final last = row['last_name'];
    final phone = row['phone'];
    final email = row['email'];

    final fullName = [
      if (first is String && first.isNotEmpty) first,
      if (last is String && last.isNotEmpty) last,
    ].join(' ');

    return ContactCard(
      id: id is int ? id : 0,
      fullName: fullName.isEmpty ? 'Без имени' : fullName,
      phone: phone is String ? phone : null,
      email: email is String ? email : null,
    );
  }

  @override
  String toString() => 'ContactCard($id, $fullName, phone=$phone, email=$email)';
}

void main() {
  final row = <String, Object?>{
    'id': 3,
    'first_name': 'Иван',
    'last_name': 'Петров',
    'phone': '+79991234567',
    'email': null,
  };
  print(ContactCard.fromDatabaseRecord(row));
  print(ContactCard.fromDatabaseRecord({'id': null}));
}
```

### №6. Клонирование с изменением полей (copyWith)

```dart
class UserModel {
  final int id;
  final String name;
  final String avatarUrl;

  const UserModel({required this.id, required this.name, required this.avatarUrl});

  // Возвращает новый объект: неизменяемая модель никогда не меняется «на месте»
  UserModel copyWith({String? name, String? avatarUrl}) => UserModel(
        id: id,
        name: name ?? this.name,
        avatarUrl: avatarUrl ?? this.avatarUrl,
      );

  @override
  String toString() => 'UserModel($id, $name, $avatarUrl)';
}

void main() {
  const u1 = UserModel(id: 1, name: 'Анна', avatarUrl: 'a.png');
  final u2 = u1.copyWith(name: 'Анна Смирнова');
  final u3 = u2.copyWith(avatarUrl: 'new.png');

  print(u1); // оригинал не изменился
  print(u2);
  print(u3);
}
```

### №7. Конструктор диапазона дат

```dart
class DateRange {
  final DateTime start;
  final DateTime end;

  DateRange(this.start, this.end)
      : assert(!end.isBefore(start), 'Дата окончания раньше даты начала');

  int get days => end.difference(start).inDays;

  @override
  String toString() => 'DateRange($start … $end, дней: $days)';
}

void main() {
  final ok = DateRange(DateTime(2026, 1, 1), DateTime(2026, 1, 10));
  print('Дней в диапазоне: ${ok.days}'); // 9

  try {
    DateRange(DateTime(2026, 5, 10), DateTime(2026, 5, 1));
  } on AssertionError catch (e) {
    print('AssertionError: ${e.message}');
  }
}
```

### №8. DTO банковской транзакции

```dart
enum TransactionType { deposit, withdrawal, transfer }

class TransactionDto {
  final TransactionType type;
  final double amount;
  final String? fromAccount;
  final String? toAccount;

  // Пополнение: есть только счёт получателя
  TransactionDto.deposit(this.amount, {required String to})
      : type = TransactionType.deposit,
        fromAccount = null,
        toAccount = to;

  // Списание: есть только счёт отправителя
  TransactionDto.withdrawal(this.amount, {required String from})
      : type = TransactionType.withdrawal,
        fromAccount = from,
        toAccount = null;

  // Перевод: оба счёта
  TransactionDto.transfer(this.amount, {required String from, required String to})
      : type = TransactionType.transfer,
        fromAccount = from,
        toAccount = to;

  @override
  String toString() => '${type.name}: $amount ₽ (из: $fromAccount, на: $toAccount)';
}

void main() {
  print(TransactionDto.deposit(5000, to: 'ACC-1'));
  print(TransactionDto.withdrawal(1200, from: 'ACC-1'));
  print(TransactionDto.transfer(300, from: 'ACC-1', to: 'ACC-2'));
}
```

### №9. Парсер конфигурации окружения (Environment)

```dart
class EnvConfig {
  final String apiUrl;
  final bool isDebug;
  final int timeoutSeconds;

  const EnvConfig._(this.apiUrl, this.isDebug, this.timeoutSeconds);

  // Значения передаются при запуске: dart run -DAPI_URL=https://prod.example.com (Flutter: --dart-define=API_URL=...)
  // Если флаг не передан, используется defaultValue
  factory EnvConfig.fromEnvironment() {
    const apiUrl = String.fromEnvironment('API_URL', defaultValue: 'https://dev.example.com');
    const isDebug = bool.fromEnvironment('DEBUG', defaultValue: true);
    const timeout = int.fromEnvironment('TIMEOUT', defaultValue: 30);
    return const EnvConfig._(apiUrl, isDebug, timeout);
  }

  @override
  String toString() => 'EnvConfig(apiUrl=$apiUrl, debug=$isDebug, timeout=${timeoutSeconds}s)';
}

void main() {
  final config = EnvConfig.fromEnvironment();
  print(config); // в DartPad выводятся значения по умолчанию
}
```

### №10. Кастомный оператор эквивалентности

```dart
class CartItem {
  final int productId;
  final String selectedSize;
  final int quantity; // в сравнении НЕ участвует

  const CartItem(this.productId, this.selectedSize, this.quantity);

  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      (other is CartItem &&
          other.productId == productId &&
          other.selectedSize == selectedSize);

  // Правило: равные объекты обязаны иметь одинаковый hashCode
  @override
  int get hashCode => Object.hash(productId, selectedSize);

  @override
  String toString() => 'CartItem($productId, $selectedSize, x$quantity)';
}

void main() {
  const a = CartItem(1, 'M', 1);
  const b = CartItem(1, 'M', 5);
  const c = CartItem(1, 'L', 1);

  print('a == b: ${a == b}'); // true
  print('a == c: ${a == c}'); // false
  print('hashCode совпадает: ${a.hashCode == b.hashCode}'); // true

  final unique = {a, b, c};
  print('Уникальных позиций в Set: ${unique.length}'); // 2
}
```

---

## Смешанные задания — Блок 2. Наследование и иерархия UI-компонентов

### №11. Иерархия системных уведомлений

```dart
abstract class InAppNotification {
  final String title;
  final Duration timeout;

  const InAppNotification(this.title, this.timeout);

  String get color;
  String get icon;

  void show() {
    final auto = timeout == Duration.zero ? 'не закрывается само' : 'закроется через ${timeout.inSeconds} с';
    print('$icon [$color] $title ($auto)');
  }
}

class SuccessNotification extends InAppNotification {
  const SuccessNotification(String title) : super(title, const Duration(seconds: 3));

  @override
  String get color => 'зелёный';

  @override
  String get icon => '✅';
}

class WarningNotification extends InAppNotification {
  const WarningNotification(String title) : super(title, const Duration(seconds: 6));

  @override
  String get color => 'жёлтый';

  @override
  String get icon => '⚠️';
}

class CriticalAlertNotification extends InAppNotification {
  // Duration.zero: требует ручного закрытия
  const CriticalAlertNotification(String title) : super(title, Duration.zero);

  @override
  String get color => 'красный';

  @override
  String get icon => '⛔';
}

void main() {
  const list = <InAppNotification>[
    SuccessNotification('Заказ оформлен'),
    WarningNotification('Слабый сигнал сети'),
    CriticalAlertNotification('Оплата отклонена'),
  ];
  for (final n in list) {
    n.show();
  }
}
```

### №12. Компоненты диалоговых окон

```dart
abstract class BaseDialog {
  String get title;

  // Шаблонный метод: общая шапка и подвал, уникальное содержимое у потомков
  void show() {
    print('┌──────────────────────────────┐');
    print('│ $title');
    print('├──────────────────────────────┤');
    buildBody();
    print('├──────────────────────────────┤');
    buildFooter();
    print('└──────────────────────────────┘');
  }

  void buildBody();

  void buildFooter() => print('│ [Закрыть]');
}

class ConfirmActionDialog extends BaseDialog {
  final String question;

  ConfirmActionDialog(this.question);

  @override
  String get title => 'Подтверждение';

  @override
  void buildBody() => print('│ $question');

  @override
  void buildFooter() => print('│ [Подтвердить]  [Отмена]');
}

void main() {
  ConfirmActionDialog('Удалить аккаунт без возможности восстановления?').show();
}
```

### №13. Иерархия медиаплееров

```dart
class MediaPlayer {
  bool _isPlaying = false;
  Duration _position = Duration.zero;

  bool get isPlaying => _isPlaying;
  Duration get position => _position;

  void play() {
    _isPlaying = true;
    print('▶ Воспроизведение с ${_position.inSeconds} с');
  }

  void pause() {
    _isPlaying = false;
    print('⏸ Пауза на ${_position.inSeconds} с');
  }

  void seek(Duration to) {
    _position = to;
    print('⏩ Перемотка на ${to.inSeconds} с');
  }
}

class AudioPlayer extends MediaPlayer {
  double volume = 1.0;

  @override
  void play() {
    print('Аудио: громкость $volume');
    super.play();
  }
}

class HlsVideoPlayer extends MediaPlayer {
  final String playlistUrl;
  int bitrateKbps = 2500;

  HlsVideoPlayer(this.playlistUrl);

  @override
  void play() {
    print('HLS: загрузка плейлиста $playlistUrl, битрейт $bitrateKbps кбит/с');
    super.play();
  }

  @override
  void seek(Duration to) {
    print('HLS: загрузка сегмента для позиции ${to.inSeconds} с');
    super.seek(to);
  }
}

void main() {
  final players = <MediaPlayer>[
    AudioPlayer(),
    HlsVideoPlayer('https://example.com/live.m3u8'),
  ];
  for (final p in players) {
    p.play();
    p.seek(const Duration(seconds: 90));
    p.pause();
    print('---');
  }
}
```

### №14. Модели сенсоров смартфона

```dart
abstract class HardwareSensor {
  final String name;

  HardwareSensor(this.name);

  String read();

  void printReading() => print('[$name] ${read()}');
}

class AccelerometerSensor extends HardwareSensor {
  final double x, y, z;

  AccelerometerSensor(this.x, this.y, this.z) : super('Акселерометр');

  @override
  String read() => 'X=$x, Y=$y, Z=$z м/с²';
}

class BarometerSensor extends HardwareSensor {
  final double pressureHpa;

  BarometerSensor(this.pressureHpa) : super('Барометр');

  @override
  String read() => 'давление $pressureHpa гПа';
}

void main() {
  final sensors = <HardwareSensor>[
    AccelerometerSensor(0.1, 9.8, 0.3),
    BarometerSensor(1013.25),
  ];
  for (final s in sensors) {
    s.printReading();
  }
}
```

### №15. Система тарифов подписки

```dart
abstract class SubscriptionPlan {
  final String name;
  final double basePrice; // цена одного месяца

  const SubscriptionPlan(this.name, this.basePrice);

  // Сколько платит пользователь за весь период
  double get totalPrice;

  void printInfo() => print('$name: ${totalPrice.toStringAsFixed(2)} ₽');
}

class MonthlyPlan extends SubscriptionPlan {
  const MonthlyPlan(double basePrice) : super('Месячный', basePrice);

  @override
  double get totalPrice => basePrice;
}

// 12 месяцев со скидкой 20%
class AnnualDiscountPlan extends SubscriptionPlan {
  const AnnualDiscountPlan(double basePrice) : super('Годовой (-20%)', basePrice);

  @override
  double get totalPrice => basePrice * 12 * 0.8;
}

// Множитель цены за участников: каждый следующий участник +50% к цене
class FamilyPlan extends SubscriptionPlan {
  final int members;

  const FamilyPlan(double basePrice, this.members) : super('Семейный', basePrice);

  double get multiplier => 1 + (members - 1) * 0.5;

  @override
  double get totalPrice => basePrice * multiplier;
}

void main() {
  const plans = <SubscriptionPlan>[
    MonthlyPlan(300),
    AnnualDiscountPlan(300),
    FamilyPlan(300, 4),
  ];
  for (final p in plans) {
    p.printInfo();
  }
}
```

### №16. Слои графического редактора

```dart
abstract class CanvasLayer {
  final String name;
  final double opacity;
  final bool visible;
  final int zIndex;

  CanvasLayer(this.name, {this.opacity = 1.0, this.visible = true, this.zIndex = 0})
      : assert(opacity >= 0 && opacity <= 1);

  void render();

  @override
  String toString() => '$name (z=$zIndex, opacity=$opacity)';
}

class VectorLayer extends CanvasLayer {
  VectorLayer(super.name, {super.opacity, super.visible, super.zIndex});

  @override
  void render() => print('Векторный слой "$name": рисуем кривые Безье');
}

class RasterBitmapLayer extends CanvasLayer {
  final int width, height;

  RasterBitmapLayer(super.name, this.width, this.height,
      {super.opacity, super.visible, super.zIndex});

  @override
  void render() => print('Растровый слой "$name": картинка ${width}x$height');
}

class TextLayer extends CanvasLayer {
  final String text;

  TextLayer(super.name, this.text, {super.opacity, super.visible, super.zIndex});

  @override
  void render() => print('Текстовый слой "$name": «$text»');
}

void main() {
  final layers = <CanvasLayer>[
    TextLayer('Заголовок', 'Привет, мир', zIndex: 3),
    RasterBitmapLayer('Фон', 1080, 1920, zIndex: 0),
    VectorLayer('Рамка', zIndex: 2, opacity: 0.5),
    VectorLayer('Скрытый', zIndex: 1, visible: false),
  ];

  // Сортировка по z-index и отрисовка только видимых слоёв
  layers.sort((a, b) => a.zIndex.compareTo(b.zIndex));
  for (final l in layers.where((l) => l.visible)) {
    l.render();
  }
}
```

### №17. Иерархия платежных шлюзов

```dart
abstract class PaymentGateway {
  final String name;

  PaymentGateway(this.name);

  // В Dart нет protected: префикс _ делает метод приватным для библиотеки (файла),
  // то есть доступным наследникам в этом же файле, но скрытым от внешнего кода
  void _log(String message) => print('[$name] $message');

  bool pay(double amount);
}

class SbpGateway extends PaymentGateway {
  final String phone;

  SbpGateway(this.phone) : super('СБП');

  @override
  bool pay(double amount) {
    _log('Перевод $amount ₽ по номеру $phone');
    return true;
  }
}

class BankCardGateway extends PaymentGateway {
  final String cardNumber;

  BankCardGateway(this.cardNumber) : super('Банковская карта');

  @override
  bool pay(double amount) {
    if (cardNumber.length != 16) {
      _log('Некорректный номер карты');
      return false;
    }
    _log('Списание $amount ₽ с карты **** ${cardNumber.substring(12)}');
    return true;
  }
}

void main() {
  final gateways = <PaymentGateway>[
    SbpGateway('+79991234567'),
    BankCardGateway('1234567812345678'),
    BankCardGateway('123'),
  ];
  for (final g in gateways) {
    print('Результат: ${g.pay(1500)}');
  }
}
```

### №18. Экраны онбординга

```dart
abstract class OnboardingStep {
  String get illustration;
  String get title;

  // Общая для всех шагов часть: кнопки «Далее» и «Пропустить»
  void show() {
    print('🖼  $illustration');
    print('   $title');
    print('   [Далее]  [Пропустить]');
  }
}

class WelcomeStep extends OnboardingStep {
  @override
  String get illustration => 'welcome.png';

  @override
  String get title => 'Добро пожаловать!';
}

class FeaturesStep extends OnboardingStep {
  @override
  String get illustration => 'features.png';

  @override
  String get title => 'Всё нужное в одном месте';
}

class PermissionsStep extends OnboardingStep {
  @override
  String get illustration => 'permissions.png';

  @override
  String get title => 'Разрешите отправку уведомлений';
}

void main() {
  final tour = <OnboardingStep>[WelcomeStep(), FeaturesStep(), PermissionsStep()];
  const skipOnStep = 2; // на втором шаге пользователь жмёт «Пропустить»

  for (var i = 0; i < tour.length; i++) {
    print('Шаг ${i + 1} из ${tour.length}');
    tour[i].show();
    if (i + 1 == skipOnStep) {
      print('Пользователь нажал «Пропустить» → тур закрыт');
      return;
    }
  }
  print('Тур завершён');
}
```

### №19. Кастомная кнопка с состояниями

```dart
class WidgetButton {
  final String label;
  final Future<void> Function() onPressed;

  WidgetButton(this.label, this.onPressed);

  Future<void> tap() async {
    print('Нажата кнопка "$label"');
    await onPressed();
  }
}

class AsyncLoadingButton extends WidgetButton {
  bool _isLoading = false;

  AsyncLoadingButton(super.label, super.onPressed);

  bool get isLoading => _isLoading;

  @override
  Future<void> tap() async {
    if (_isLoading) {
      print('Кнопка "$label" занята — нажатие проигнорировано');
      return;
    }
    _isLoading = true;
    print('⏳ Индикатор загрузки включён');
    try {
      await super.tap();
    } finally {
      _isLoading = false;
      print('✔ Индикатор загрузки выключен');
    }
  }
}

Future<void> main() async {
  final button = AsyncLoadingButton('Отправить', () async {
    await Future.delayed(const Duration(milliseconds: 300));
    print('Данные отправлены на сервер');
  });

  final first = button.tap(); // запускается
  await button.tap();         // второе нажатие во время загрузки игнорируется
  await first;
}
```

### №20. Иерархия логгеров приложения

```dart
abstract class AppLogger {
  void log(String level, String message);

  void info(String message) => log('INFO', message);
  void error(String message) => log('ERROR', message);
}

class ConsoleLogger extends AppLogger {
  @override
  void log(String level, String message) {
    final time = DateTime.now().toIso8601String().substring(11, 19);
    print('[$time] ${level.padRight(5)} | $message');
  }
}

// Запись «на диск» имитируется буфером. В приложении здесь был бы dart:io:
//   File('crash.log').writeAsStringSync(line, mode: FileMode.append);
class FileCrashLogger extends AppLogger {
  final StringBuffer _file = StringBuffer();

  @override
  void log(String level, String message) {
    if (level == 'ERROR') {
      _file.writeln('${DateTime.now().toIso8601String()} $level $message');
    }
  }

  String get fileContent => _file.toString();
}

void main() {
  final console = ConsoleLogger();
  final crash = FileCrashLogger();

  for (final logger in <AppLogger>[console, crash]) {
    logger.info('Приложение запущено');
    logger.error('NullPointer в CartScreen');
  }

  print('--- Содержимое crash.log ---');
  print(crash.fileContent); // только строка с ERROR
}
```

---

## Смешанные задания — Блок 3. Абстрактные контракты, интерфейсы и репозитории

### №21. Репозиторий каталога товаров (Repository Pattern)

```dart
class Product {
  final int id;
  final String name;
  final double price;

  const Product(this.id, this.name, this.price);

  @override
  String toString() => '#$id $name ($price ₽)';
}

abstract interface class ProductRepository {
  Future<List<Product>> getAll();
  Future<Product> getById(int id);
}

// «Сеть»: имитируем ответ сервера с задержкой
class NetworkProductRepository implements ProductRepository {
  final List<Map<String, Object>> _serverJson = [
    {'id': 1, 'name': 'Ноутбук', 'price': 65000.0},
    {'id': 2, 'name': 'Мышь', 'price': 1500.0},
  ];

  @override
  Future<List<Product>> getAll() async {
    await Future.delayed(const Duration(milliseconds: 300));
    return [
      for (final j in _serverJson)
        Product(j['id'] as int, j['name'] as String, j['price'] as double),
    ];
  }

  @override
  Future<Product> getById(int id) async {
    final all = await getAll();
    return all.firstWhere((p) => p.id == id,
        orElse: () => throw StateError('Товар $id не найден'));
  }
}

// Заглушка для тестов: мгновенные ответы
class MockProductRepository implements ProductRepository {
  static const _items = [Product(1, 'Тестовый товар', 1), Product(2, 'Другой товар', 2)];

  @override
  Future<List<Product>> getAll() async => _items;

  @override
  Future<Product> getById(int id) async =>
      _items.firstWhere((p) => p.id == id, orElse: () => throw StateError('Товар $id не найден'));
}

Future<void> demo(String title, ProductRepository repo) async {
  print('== $title ==');
  print('Все: ${await repo.getAll()}');
  print('По id=2: ${await repo.getById(2)}');
  try {
    await repo.getById(99);
  } on StateError catch (e) {
    print('Ошибка: ${e.message}');
  }
}

Future<void> main() async {
  await demo('Network', NetworkProductRepository());
  await demo('Mock', MockProductRepository());
}
```

### №22. Интерфейс биометрической аутентификации

```dart
abstract interface class BiometricAuthService {
  Future<bool> canAuthenticate();
  Future<bool> authenticate();
}

// Заглушка для тестов: поведение настраивается флагами
class FakeBiometricAuthService implements BiometricAuthService {
  final bool available;
  final bool succeeds;

  FakeBiometricAuthService({this.available = true, this.succeeds = true});

  @override
  Future<bool> canAuthenticate() async => available;

  @override
  Future<bool> authenticate() async {
    await Future.delayed(const Duration(milliseconds: 100));
    return succeeds;
  }
}

Future<void> login(BiometricAuthService auth) async {
  if (!await auth.canAuthenticate()) {
    print('Биометрия недоступна → вход по паролю');
    return;
  }
  print(await auth.authenticate() ? 'Вход выполнен по отпечатку' : 'Отпечаток не распознан');
}

Future<void> main() async {
  await login(FakeBiometricAuthService());
  await login(FakeBiometricAuthService(succeeds: false));
  await login(FakeBiometricAuthService(available: false));
}
```

### №23. Шифрование пользовательских данных

```dart
import 'dart:convert';

abstract interface class CryptoService {
  String encrypt(String plainText);
  String decrypt(String cipherText);
}

// УЧЕБНАЯ реализация: настоящий AES в чистом Dart требует пакетов (encrypt, pointycastle).
// Здесь — XOR с ключом + Base64, чтобы показать сам паттерн «интерфейс → реализация».
class AesCryptoService implements CryptoService {
  final List<int> _key;

  AesCryptoService(String secret) : _key = utf8.encode(secret);

  List<int> _xor(List<int> data) =>
      [for (var i = 0; i < data.length; i++) data[i] ^ _key[i % _key.length]];

  @override
  String encrypt(String plainText) => base64Encode(_xor(utf8.encode(plainText)));

  @override
  String decrypt(String cipherText) => utf8.decode(_xor(base64Decode(cipherText)));
}

void main() {
  final CryptoService crypto = AesCryptoService('super-secret-key');
  final encrypted = crypto.encrypt('Номер карты: 1234 5678');
  print('Зашифровано: $encrypted');
  print('Расшифровано: ${crypto.decrypt(encrypted)}');
}
```

### №24. Контракт аналитического провайдера

```dart
abstract interface class AnalyticsProvider {
  void logEvent(String name, Map<String, Object> params);
}

class ConsoleAnalytics implements AnalyticsProvider {
  @override
  void logEvent(String name, Map<String, Object> params) =>
      print('[Console] $name $params');
}

class MemoryAnalytics implements AnalyticsProvider {
  final List<String> events = [];

  @override
  void logEvent(String name, Map<String, Object> params) => events.add(name);
}

class BrokenAnalytics implements AnalyticsProvider {
  @override
  void logEvent(String name, Map<String, Object> params) =>
      throw Exception('нет соединения с сервером');
}

// Отправляет событие сразу во все провайдеры; сбой одного не ломает остальные
class CompositeAnalyticsProvider implements AnalyticsProvider {
  final List<AnalyticsProvider> providers;

  CompositeAnalyticsProvider(this.providers);

  @override
  void logEvent(String name, Map<String, Object> params) {
    for (final p in providers) {
      try {
        p.logEvent(name, params);
      } catch (e) {
        print('Провайдер ${p.runtimeType} не отправил событие: $e');
      }
    }
  }
}

void main() {
  final memory = MemoryAnalytics();
  final analytics = CompositeAnalyticsProvider([ConsoleAnalytics(), BrokenAnalytics(), memory]);

  analytics.logEvent('purchase', {'amount': 1990, 'currency': 'RUB'});
  analytics.logEvent('screen_view', {'name': 'Cart'});
  print('В памяти сохранено событий: ${memory.events.length}'); // 2
}
```

### №25. Интерфейс геолокатора

```dart
class GeoPoint {
  final double lat, lon;

  const GeoPoint(this.lat, this.lon);

  @override
  String toString() => '(${lat.toStringAsFixed(4)}, ${lon.toStringAsFixed(4)})';
}

abstract interface class GeoLocator {
  Future<GeoPoint> getCurrentPosition();
  Stream<GeoPoint> get positionStream;
}

// Имитация движения: каждые 200 мс координаты немного меняются
class FakeGeoLocator implements GeoLocator {
  @override
  Future<GeoPoint> getCurrentPosition() async => const GeoPoint(55.7512, 37.6184);

  @override
  Stream<GeoPoint> get positionStream => Stream.periodic(
        const Duration(milliseconds: 200),
        (i) => GeoPoint(55.7512 + i * 0.0005, 37.6184 + i * 0.0005),
      ).take(4);
}

Future<void> main() async {
  final GeoLocator locator = FakeGeoLocator();
  print('Сейчас: ${await locator.getCurrentPosition()}');

  await for (final p in locator.positionStream) {
    print('Обновление позиции: $p');
  }
  print('Трекинг завершён');
}
```

### №26. Служба локальных уведомлений

```dart
abstract interface class NotificationSender {
  Future<void> sendNow(String title, String body);
  Future<void> schedule(String title, String body, Duration delay);
}

class ConsoleNotificationSender implements NotificationSender {
  @override
  Future<void> sendNow(String title, String body) async {
    print('🔔 СЕЙЧАС: $title — $body');
  }

  @override
  Future<void> schedule(String title, String body, Duration delay) async {
    print('🕒 Запланировано через ${delay.inMilliseconds} мс: $title');
    await Future.delayed(delay);
    await sendNow(title, body);
  }
}

Future<void> main() async {
  final NotificationSender sender = ConsoleNotificationSender();
  await sender.sendNow('Привет', 'Это мгновенное уведомление');
  await sender.schedule('Напоминание', 'Пора пить воду', const Duration(milliseconds: 500));
}
```

### №27. Контракт синхронизатора баз данных

```dart
abstract class DataSyncEngine {
  Future<List<String>> fetchRemoteDelta();
  Future<void> applyLocalChanges(List<String> changes);

  // Шаблонный метод: порядок шагов задан в абстрактном классе
  Future<void> sync() async {
    print('Синхронизация: старт');
    final delta = await fetchRemoteDelta();
    print('Получено с сервера: ${delta.length} изменений');
    await applyLocalChanges(delta);
    print('Синхронизация: готово');
  }
}

class NotesSyncEngine extends DataSyncEngine {
  final List<String> localNotes = ['Заметка 1'];

  @override
  Future<List<String>> fetchRemoteDelta() async {
    await Future.delayed(const Duration(milliseconds: 200));
    return ['Заметка 2', 'Заметка 3'];
  }

  @override
  Future<void> applyLocalChanges(List<String> changes) async {
    localNotes.addAll(changes);
  }
}

Future<void> main() async {
  final engine = NotesSyncEngine();
  await engine.sync();
  print('Локальная база: ${engine.localNotes}');
}
```

### №28. Провайдер сетевого статуса

```dart
import 'dart:async';

abstract interface class ConnectivityService {
  bool get isOnline;
  Stream<void> get onConnectionLost;
}

class FakeConnectivityService implements ConnectivityService {
  bool _online = true;
  final StreamController<void> _lost = StreamController<void>.broadcast();

  @override
  bool get isOnline => _online;

  @override
  Stream<void> get onConnectionLost => _lost.stream;

  // Метод для имитации смены состояния сети
  void setOnline(bool value) {
    final wasOnline = _online;
    _online = value;
    if (wasOnline && !value) _lost.add(null);
  }

  void dispose() => _lost.close();
}

Future<void> main() async {
  final service = FakeConnectivityService();
  service.onConnectionLost.listen((_) => print('⚠ Потеряно соединение с интернетом!'));

  print('Онлайн: ${service.isOnline}'); // true
  service.setOnline(false);
  await Future.delayed(const Duration(milliseconds: 10)); // даём событию дойти до слушателя
  print('Онлайн: ${service.isOnline}'); // false
  service.setOnline(true);
  print('Онлайн: ${service.isOnline}'); // true
  service.dispose();
}
```

### №29. Хранилище ключ-значение (KeyValueStorage)

```dart
abstract interface class KeyValueStorage {
  T? read<T>(String key);
  Future<void> write<T>(String key, T value);
}

class MemoryKeyValueStorage implements KeyValueStorage {
  final Map<String, Object?> _data = {};

  // Если тип не совпал (запросили int, а лежит String) — возвращаем null
  @override
  T? read<T>(String key) {
    final value = _data[key];
    return value is T ? value : null;
  }

  @override
  Future<void> write<T>(String key, T value) async {
    _data[key] = value;
  }
}

Future<void> main() async {
  final KeyValueStorage storage = MemoryKeyValueStorage();
  await storage.write<int>('launches', 5);
  await storage.write<String>('theme', 'dark');
  await storage.write<bool>('onboarded', true);

  print(storage.read<int>('launches'));    // 5
  print(storage.read<String>('theme'));    // dark
  print(storage.read<String>('launches')); // null (тип не совпал)
  print(storage.read<int>('missing'));     // null
}
```

### №30. Интерфейс экспортера отчетов

```dart
import 'dart:convert';

class Report {
  final String title;
  final List<List<String>> rows;

  const Report(this.title, this.rows);
}

abstract interface class ReportExporter {
  String get extension;
  List<int> export(Report report);
}

// Имитация PDF: настоящий PDF формируют пакеты (pdf, printing)
class PdfReportExporter implements ReportExporter {
  @override
  String get extension => 'pdf';

  @override
  List<int> export(Report report) {
    final text = '%PDF-1.4\n${report.title}\n${report.rows.map((r) => r.join(' | ')).join('\n')}';
    return utf8.encode(text);
  }
}

// Имитация Excel: табличный текст с разделителем ';' (открывается в Excel как CSV)
class ExcelReportExporter implements ReportExporter {
  @override
  String get extension => 'csv';

  @override
  List<int> export(Report report) =>
      utf8.encode(report.rows.map((r) => r.join(';')).join('\n'));
}

void main() {
  const report = Report('Продажи за неделю', [
    ['Товар', 'Кол-во'],
    ['Чайник', '12'],
    ['Мышь', '30'],
  ]);

  for (final ReportExporter e in [PdfReportExporter(), ExcelReportExporter()]) {
    final bytes = e.export(report);
    print('report.${e.extension}: ${bytes.length} байт');
  }
}
```

---

## Смешанные задания — Блок 4. Миксины и повторное использование функционала

### №31. Миксин дебаунса событий (Anti-Spam Click)

```dart
mixin DebounceMixin {
  final Map<String, DateTime> _lastCalls = {};

  // Возвращает true, если действие выполнено, и false, если нажатие подавлено
  bool debounce(
    String key,
    void Function() action, {
    Duration interval = const Duration(milliseconds: 500),
  }) {
    final now = DateTime.now();
    final last = _lastCalls[key];
    if (last != null && now.difference(last) < interval) return false;
    _lastCalls[key] = now;
    action();
    return true;
  }
}

class SubmitButton with DebounceMixin {
  void onTap() {
    final executed = debounce('submit', () => print('Форма отправлена!'));
    if (!executed) print('Нажатие подавлено (слишком часто)');
  }
}

Future<void> main() async {
  final button = SubmitButton();
  button.onTap(); // выполнено
  button.onTap(); // подавлено
  button.onTap(); // подавлено

  await Future.delayed(const Duration(milliseconds: 600));
  button.onTap(); // снова выполнено
}
```

### №32. Миксин восстановления состояния экрана (RestorationMixin)

```dart
// Имитация хранилища, которое переживает выгрузку экрана из памяти
final Map<String, Map<String, String>> _savedStates = {};

mixin RestorationMixin {
  String get restorationId;
  Map<String, String> get formFields;

  // Вызывается при сворачивании приложения
  void onAppPaused() {
    _savedStates[restorationId] = Map.of(formFields);
    print('[$restorationId] Состояние сохранено: $formFields');
  }

  // Вызывается при возврате в приложение
  void onAppResumed() {
    final saved = _savedStates[restorationId];
    if (saved == null) {
      print('[$restorationId] Нечего восстанавливать');
      return;
    }
    formFields
      ..clear()
      ..addAll(saved);
    print('[$restorationId] Состояние восстановлено: $formFields');
  }
}

class RegistrationScreen with RestorationMixin {
  @override
  String get restorationId => 'registration';

  @override
  final Map<String, String> formFields = {};
}

void main() {
  final screen = RegistrationScreen();
  screen.formFields['name'] = 'Анна';
  screen.formFields['email'] = 'anna@mail.ru';
  screen.onAppPaused(); // пользователь свернул приложение

  // Система выгрузила экран из памяти — при возврате создаётся новый объект
  final recreated = RegistrationScreen();
  print('Поля после пересоздания: ${recreated.formFields}'); // пусто
  recreated.onAppResumed();
}
```

### №33. Миксин проверки сетевого соединения

```dart
mixin NetworkAwareMixin {
  bool get isOnline;

  // Любой запрос выполняется только при наличии интернета
  Future<T?> safeRequest<T>(Future<T> Function() request) async {
    if (!isOnline) {
      print('Нет интернета: запрос не выполнен');
      return null;
    }
    return request();
  }
}

class ProductsPresenter with NetworkAwareMixin {
  bool online = true;

  @override
  bool get isOnline => online;

  Future<void> loadProducts() async {
    final products = await safeRequest<List<String>>(() async {
      await Future.delayed(const Duration(milliseconds: 200));
      return ['Ноутбук', 'Мышь', 'Клавиатура'];
    });
    print(products ?? 'Показываем данные из кэша');
  }
}

Future<void> main() async {
  final presenter = ProductsPresenter();
  await presenter.loadProducts(); // онлайн

  presenter.online = false;
  await presenter.loadProducts(); // офлайн
}
```

### №34. Миксин форматирования цен и валют

```dart
mixin CurrencyFormatterMixin {
  // 15400900.5 → "15 400 900.50 ₽"
  String formatMoney(num value, {String symbol = '₽'}) {
    final parts = value.toStringAsFixed(2).split('.');
    final grouped = parts[0].replaceAllMapped(
      RegExp(r'\B(?=(\d{3})+(?!\d))'),
      (_) => ' ',
    );
    return '$grouped.${parts[1]} $symbol';
  }
}

class CheckoutPresenter with CurrencyFormatterMixin {
  void showTotal(num total) => print('К оплате: ${formatMoney(total)}');
}

void main() {
  final p = CheckoutPresenter();
  p.showTotal(15400900.5); // К оплате: 15 400 900.50 ₽
  p.showTotal(999);        // К оплате: 999.00 ₽
  print(p.formatMoney(1234567.891, symbol: r'$')); // 1 234 567.89 $
}
```

### №35. Миксин таймера автообновления

```dart
import 'dart:async';

mixin PeriodicRefreshMixin {
  Timer? _timer;
  bool isScreenActive = true;

  Future<void> refresh();

  // По умолчанию опрос каждые 30 секунд
  void startAutoRefresh({Duration interval = const Duration(seconds: 30)}) {
    stopAutoRefresh();
    _timer = Timer.periodic(interval, (_) {
      if (isScreenActive) {
        refresh();
      } else {
        print('Экран неактивен — опрос пропущен');
      }
    });
  }

  void stopAutoRefresh() {
    _timer?.cancel();
    _timer = null;
  }
}

class FeedScreen with PeriodicRefreshMixin {
  int polls = 0;

  @override
  Future<void> refresh() async {
    polls++;
    print('Опрос сервера #$polls');
  }
}

Future<void> main() async {
  final screen = FeedScreen();

  // Для демонстрации интервал сокращён до 300 мс (в приложении — 30 секунд)
  screen.startAutoRefresh(interval: const Duration(milliseconds: 300));
  await Future.delayed(const Duration(milliseconds: 1000)); // 3 опроса

  screen.isScreenActive = false; // экран ушёл в фон
  await Future.delayed(const Duration(milliseconds: 400));

  screen.stopAutoRefresh();
  print('Всего опросов: ${screen.polls}');
}
```

### №36. Миксин валидации паспортных данных

```dart
mixin PassportValidationMixin {
  // Серия: 4 цифры (допускается запись "45 12")
  bool isValidSeries(String series) =>
      RegExp(r'^\d{4}$').hasMatch(series.replaceAll(' ', ''));

  // Номер: 6 цифр
  bool isValidNumber(String number) => RegExp(r'^\d{6}$').hasMatch(number);

  // Код подразделения: формат 123-456
  bool isValidDepartmentCode(String code) =>
      RegExp(r'^\d{3}-\d{3}$').hasMatch(code);

  List<String> validatePassport({
    required String series,
    required String number,
    required String departmentCode,
  }) {
    final errors = <String>[];
    if (!isValidSeries(series)) errors.add('Серия должна состоять из 4 цифр');
    if (!isValidNumber(number)) errors.add('Номер должен состоять из 6 цифр');
    if (!isValidDepartmentCode(departmentCode)) {
      errors.add('Код подразделения должен иметь вид 123-456');
    }
    return errors;
  }
}

class PassportForm with PassportValidationMixin {}

void main() {
  final form = PassportForm();

  final ok = form.validatePassport(series: '45 12', number: '123456', departmentCode: '770-001');
  print(ok.isEmpty ? 'Паспорт валиден' : ok);

  final bad = form.validatePassport(series: '45A', number: '12345', departmentCode: '770001');
  print('Ошибки: $bad');
}
```

### №37. Миксин анимации затухания (FadeAnimationController)

```dart
// Во Flutter для этого используют AnimationController; здесь — упрощённый аналог на чистом Dart
mixin FadeAnimationMixin {
  double _opacity = 0.0;

  double get opacity => _opacity;

  Future<void> fadeTo(
    double target, {
    int steps = 5,
    Duration stepDuration = const Duration(milliseconds: 40),
  }) async {
    final end = target.clamp(0.0, 1.0).toDouble();
    final start = _opacity;
    for (var i = 1; i <= steps; i++) {
      _opacity = start + (end - start) * i / steps;
      print('  opacity = ${_opacity.toStringAsFixed(2)}');
      await Future.delayed(stepDuration);
    }
  }

  Future<void> fadeIn() => fadeTo(1.0);
  Future<void> fadeOut() => fadeTo(0.0);
}

class ImageBanner with FadeAnimationMixin {}

Future<void> main() async {
  final banner = ImageBanner();
  print('Появление:');
  await banner.fadeIn();
  print('Исчезновение:');
  await banner.fadeOut();
}
```

### №38. Миксин переключения темы

```dart
enum AppTheme { light, dark }

class AppPalette {
  final String background;
  final String text;
  final String accent;

  const AppPalette(this.background, this.text, this.accent);

  static const light = AppPalette('#FFFFFF', '#212121', '#6200EE');
  static const dark = AppPalette('#121212', '#EEEEEE', '#BB86FC');
}

// Глобальное состояние темы приложения
class ThemeController {
  static AppTheme current = AppTheme.light;

  static void toggle() {
    current = current == AppTheme.light ? AppTheme.dark : AppTheme.light;
  }
}

// Миксин даёт любому экрану быстрый доступ к текущей палитре
mixin ThemeAccessMixin {
  AppTheme get theme => ThemeController.current;

  bool get isDark => theme == AppTheme.dark;

  AppPalette get palette => isDark ? AppPalette.dark : AppPalette.light;
}

class ProfileScreen with ThemeAccessMixin {
  void build() {
    print('Тема: ${theme.name}, фон ${palette.background}, текст ${palette.text}, акцент ${palette.accent}');
  }
}

void main() {
  final screen = ProfileScreen();
  screen.build();
  ThemeController.toggle();
  screen.build();
}
```

### №39. Миксин глубоких ссылок (DeepLinkHandlerMixin)

```dart
mixin DeepLinkHandlerMixin {
  String get scheme;

  void openScreen(String route, Map<String, String> args);

  bool handleDeepLink(String link) {
    final uri = Uri.tryParse(link);
    if (uri == null || uri.scheme != scheme) {
      print('Ссылка не принадлежит приложению: $link');
      return false;
    }

    final segments = uri.pathSegments;
    switch (uri.host) {
      case 'product':
        if (segments.isEmpty) return false;
        openScreen('/product', {'id': segments.first, ...uri.queryParameters});
        return true;
      case 'profile':
        openScreen('/profile', {'user': segments.isEmpty ? 'me' : segments.first});
        return true;
      case 'cart':
        openScreen('/cart', uri.queryParameters);
        return true;
      default:
        print('Неизвестный маршрут: ${uri.host}');
        return false;
    }
  }
}

class AppRouter with DeepLinkHandlerMixin {
  @override
  String get scheme => 'myapp';

  @override
  void openScreen(String route, Map<String, String> args) =>
      print('Открываем экран $route с параметрами $args');
}

void main() {
  final router = AppRouter();
  router.handleDeepLink('myapp://product/42?ref=ad');
  router.handleDeepLink('myapp://profile/anna');
  router.handleDeepLink('myapp://cart?promo=SALE10');
  router.handleDeepLink('myapp://unknown/1');
  router.handleDeepLink('https://example.com/page');
}
```

### №40. Миксин аудита действий администратора

```dart
// Имитация выделенного сервера безопасности
class SecurityServer {
  final List<String> receivedLog = [];

  Future<void> send(String entry) async {
    await Future.delayed(const Duration(milliseconds: 100));
    receivedLog.add(entry);
    print('[SecurityServer] принято: $entry');
  }
}

final SecurityServer _securityServer = SecurityServer();

mixin AdminAuditMixin {
  String get adminId;

  // Отправляет критическое действие модератора на сервер безопасности
  Future<void> audit(String action, {Map<String, Object?> details = const {}}) {
    final time = DateTime.now().toIso8601String().substring(0, 19);
    return _securityServer.send('$time admin=$adminId action=$action details=$details');
  }
}

class Moderator with AdminAuditMixin {
  @override
  final String adminId;

  Moderator(this.adminId);

  Future<void> banUser(String userId, String reason) async {
    print('Модератор $adminId блокирует пользователя $userId');
    await audit('ban_user', details: {'userId': userId, 'reason': reason});
  }

  Future<void> deletePost(int postId) async {
    print('Модератор $adminId удаляет пост $postId');
    await audit('delete_post', details: {'postId': postId});
  }
}

Future<void> main() async {
  final moderator = Moderator('mod-17');
  await moderator.banUser('user-505', 'спам');
  await moderator.deletePost(9001);
  print('Записей на сервере безопасности: ${_securityServer.receivedLog.length}'); // 2
}
```

---

## Смешанные задания — Блок 5. Современные модификаторы классов и архитектурные стейты

### №41. Sealed-состояние корзины интернет-магазина

```dart
class Item {
  final String name;
  final double price;

  const Item(this.name, this.price);
}

sealed class CartState {
  const CartState();
}

class CartEmpty extends CartState {
  const CartEmpty();
}

class CartLoading extends CartState {
  const CartLoading();
}

class CartLoaded extends CartState {
  final List<Item> items;
  final double total;

  const CartLoaded(this.items, this.total);
}

class CartCheckoutError extends CartState {
  final String error;

  const CartCheckoutError(this.error);
}

// Исчерпывающий switch: если добавить новое состояние, код перестанет компилироваться
String renderCart(CartState state) => switch (state) {
      CartEmpty() => 'Корзина пуста',
      CartLoading() => 'Загрузка корзины...',
      CartLoaded(:final items, :final total) =>
        'В корзине ${items.length} шт., итого ${total.toStringAsFixed(2)} ₽',
      CartCheckoutError(:final error) => 'Ошибка оформления: $error',
    };

void main() {
  final states = <CartState>[
    const CartEmpty(),
    const CartLoading(),
    const CartLoaded([Item('Чайник', 2490), Item('Кружка', 390)], 2880),
    const CartCheckoutError('Недостаточно средств'),
  ];
  for (final s in states) {
    print(renderCart(s));
  }
}
```

### №42. Sealed-иерархия сетевого ответа (Result Pattern)

```dart
sealed class Result<T> {
  const Result();

  // Удобный метод: обрабатывает оба исхода
  R when<R>({
    required R Function(T data) success,
    required R Function(Exception exception) failure,
  }) =>
      switch (this) {
        Success<T>(:final data) => success(data),
        Failure<T>(:final exception) => failure(exception),
      };
}

final class Success<T> extends Result<T> {
  final T data;

  const Success(this.data);
}

final class Failure<T> extends Result<T> {
  final Exception exception;

  const Failure(this.exception);
}

Result<int> parseAge(String input) {
  final age = int.tryParse(input);
  if (age == null) return Failure(FormatException('"$input" — не число'));
  if (age < 0 || age > 150) return Failure(FormatException('Возраст вне диапазона: $age'));
  return Success(age);
}

void main() {
  for (final text in ['25', 'abc', '200']) {
    final result = parseAge(text);
    final message = result.when(
      success: (age) => 'Возраст принят: $age',
      failure: (e) => 'Ошибка: $e',
    );
    print(message);
  }

  // То же самое через switch
  final r = parseAge('42');
  print(switch (r) {
    Success(:final data) => 'switch: данные $data',
    Failure(:final exception) => 'switch: ошибка $exception',
  });
}
```

### №43. Архитектура UI-событий экрана логина

```dart
sealed class LoginEvent {
  const LoginEvent();
}

class LoginEmailChanged extends LoginEvent {
  final String email;

  const LoginEmailChanged(this.email);
}

class LoginPasswordChanged extends LoginEvent {
  final String pass;

  const LoginPasswordChanged(this.pass);
}

class LoginSubmitted extends LoginEvent {
  const LoginSubmitted();
}

class LoginState {
  final String email;
  final String password;
  final bool isSubmitting;

  const LoginState({this.email = '', this.password = '', this.isSubmitting = false});

  LoginState copyWith({String? email, String? password, bool? isSubmitting}) => LoginState(
        email: email ?? this.email,
        password: password ?? this.password,
        isSubmitting: isSubmitting ?? this.isSubmitting,
      );

  @override
  String toString() =>
      'LoginState(email=$email, password=${'*' * password.length}, submitting=$isSubmitting)';
}

// Редьюсер: исчерпывающая обработка всех событий
LoginState reduce(LoginState state, LoginEvent event) => switch (event) {
      LoginEmailChanged(:final email) => state.copyWith(email: email),
      LoginPasswordChanged(:final pass) => state.copyWith(password: pass),
      LoginSubmitted() =>
        state.copyWith(isSubmitting: state.email.contains('@') && state.password.length >= 6),
    };

void main() {
  var state = const LoginState();
  final events = <LoginEvent>[
    const LoginSubmitted(), // форма пустая → отправка не стартует
    const LoginEmailChanged('anna@mail.ru'),
    const LoginPasswordChanged('secret123'),
    const LoginSubmitted(),
  ];
  for (final e in events) {
    state = reduce(state, e);
    print('${e.runtimeType} → $state');
  }
}
```

### №44. Модификатор `final class` для безопасности криптоключа

```dart
import 'dart:math';

// final: класс нельзя ни расширить (extends), ни реализовать (implements) вне библиотеки
final class EncryptionKey {
  final List<int> _bytes;

  EncryptionKey._(this._bytes);

  factory EncryptionKey.generate({int length = 32}) {
    final random = Random.secure();
    return EncryptionKey._(List<int>.generate(length, (_) => random.nextInt(256)));
  }

  int get length => _bytes.length;

  // Показываем только короткий отпечаток, но не сам ключ
  String get fingerprint =>
      _bytes.take(4).map((b) => b.toRadixString(16).padLeft(2, '0')).join();

  @override
  String toString() => 'EncryptionKey(***, $length байт)';
}

// В другой библиотеке обе строки были бы ошибками компиляции:
// class FakeKey extends EncryptionKey {}
// class FakeKey2 implements EncryptionKey {}

void main() {
  final key = EncryptionKey.generate();
  print(key);                              // ключ не раскрывается
  print('Отпечаток: ${key.fingerprint}');
}
```

### №45. Модификатор `interface class` для публичного API SDK

```dart
// abstract interface class: клиенты SDK могут только реализовать контракт (implements);
// наследовать (extends) его вне библиотеки нельзя, поэтому внутренняя логика не «протекает»
abstract interface class PaymentTerminal {
  Future<bool> charge(double amount);
  Future<bool> refund(double amount);
}

// Реализация клиента SDK
class SberTerminal implements PaymentTerminal {
  @override
  Future<bool> charge(double amount) async {
    print('Sber: списание $amount ₽');
    return true;
  }

  @override
  Future<bool> refund(double amount) async {
    print('Sber: возврат $amount ₽');
    return true;
  }
}

// В другой библиотеке была бы ошибкой строка:
// class Bad extends PaymentTerminal {}

Future<void> main() async {
  final PaymentTerminal terminal = SberTerminal();
  await terminal.charge(1500);
  await terminal.refund(500);
}
```

### №46. Исчерпывающий роутинг экрана через Sealed Route

```dart
sealed class AppRoute {
  const AppRoute();
}

final class HomeRoute extends AppRoute {
  const HomeRoute();
}

final class ProductRoute extends AppRoute {
  final int productId;

  const ProductRoute(this.productId);
}

final class CartRoute extends AppRoute {
  const CartRoute();
}

final class SettingsRoute extends AppRoute {
  final String section;

  const SettingsRoute([this.section = 'general']);
}

// Маршрут → путь (switch без default: все типы обязаны быть учтены)
String toPath(AppRoute route) => switch (route) {
      HomeRoute() => '/',
      ProductRoute(:final productId) => '/product/$productId',
      CartRoute() => '/cart',
      SettingsRoute(:final section) => '/settings/$section',
    };

// Путь → маршрут (list patterns из Dart 3)
AppRoute? fromPath(String path) {
  final segments = path.split('/').where((s) => s.isNotEmpty).toList();
  return switch (segments) {
    [] => const HomeRoute(),
    ['product', final id] when int.tryParse(id) != null => ProductRoute(int.parse(id)),
    ['cart'] => const CartRoute(),
    ['settings'] => const SettingsRoute(),
    ['settings', final section] => SettingsRoute(section),
    _ => null,
  };
}

void main() {
  const routes = <AppRoute>[HomeRoute(), ProductRoute(42), CartRoute(), SettingsRoute('privacy')];
  for (final r in routes) {
    print('${r.runtimeType} → ${toPath(r)}');
  }

  for (final path in ['/product/7', '/settings', '/product/abc', '/nowhere']) {
    final route = fromPath(path);
    print('$path → ${route == null ? 'маршрут не найден' : route.runtimeType}');
  }
}
```

### №47. Паттерн «Команда» (Command Pattern)

```dart
import 'dart:math';

class TextEditor {
  String text = '';
}

abstract class Command {
  void execute();
  void undo();
}

class InsertTextCommand extends Command {
  final TextEditor editor;
  final String value;

  InsertTextCommand(this.editor, this.value);

  @override
  void execute() => editor.text += value;

  @override
  void undo() => editor.text = editor.text.substring(0, editor.text.length - value.length);
}

class DeleteLastCommand extends Command {
  final TextEditor editor;
  final int count;
  String _deleted = '';

  DeleteLastCommand(this.editor, this.count);

  @override
  void execute() {
    final n = min(count, editor.text.length);
    _deleted = editor.text.substring(editor.text.length - n);
    editor.text = editor.text.substring(0, editor.text.length - n);
  }

  @override
  void undo() => editor.text += _deleted;
}

class CommandHistory {
  final List<Command> _undoStack = [];
  final List<Command> _redoStack = [];

  void run(Command command) {
    command.execute();
    _undoStack.add(command);
    _redoStack.clear();
  }

  void undo() {
    if (_undoStack.isEmpty) return;
    final command = _undoStack.removeLast();
    command.undo();
    _redoStack.add(command);
  }

  void redo() {
    if (_redoStack.isEmpty) return;
    final command = _redoStack.removeLast();
    command.execute();
    _undoStack.add(command);
  }
}

void main() {
  final editor = TextEditor();
  final history = CommandHistory();

  history.run(InsertTextCommand(editor, 'Привет'));
  history.run(InsertTextCommand(editor, ', мир'));
  print('После ввода: "${editor.text}"');       // Привет, мир

  history.run(DeleteLastCommand(editor, 3));
  print('После удаления: "${editor.text}"');    // Привет,

  history.undo();
  print('Undo: "${editor.text}"');              // Привет, мир
  history.undo();
  print('Undo: "${editor.text}"');              // Привет
  history.redo();
  print('Redo: "${editor.text}"');              // Привет, мир
}
```

### №48. Паттерн «Строитель» (Builder)

```dart
class AppNotification {
  final String title;
  final String body;
  final String icon;

  const AppNotification._(this.title, this.body, this.icon);

  @override
  String toString() => '$icon $title — $body';
}

class NotificationBuilder {
  String? _title;
  String? _body;
  String _icon = '🔔';

  // Каждый set-метод возвращает this → цепочка вызовов
  NotificationBuilder setTitle(String title) {
    _title = title;
    return this;
  }

  NotificationBuilder setBody(String body) {
    _body = body;
    return this;
  }

  NotificationBuilder setIcon(String icon) {
    _icon = icon;
    return this;
  }

  AppNotification build() {
    final title = _title;
    final body = _body;
    if (title == null || title.isEmpty) throw StateError('Заголовок обязателен');
    if (body == null || body.isEmpty) throw StateError('Текст обязателен');
    return AppNotification._(title, body, _icon);
  }
}

void main() {
  final n = NotificationBuilder()
      .setTitle('Заказ доставлен')
      .setBody('Курьер оставил посылку у двери')
      .setIcon('📦')
      .build();
  print(n);

  try {
    NotificationBuilder().setBody('Без заголовка').build();
  } on StateError catch (e) {
    print('Ошибка: ${e.message}');
  }
}
```

### №49. Паттерн «Наблюдатель» (Observer / EventBus)

```dart
class EventBus {
  final Map<Type, List<void Function(Object)>> _handlers = {};

  // Подписка на события типа T; возвращает функцию отписки
  void Function() subscribe<T extends Object>(void Function(T event) handler) {
    final void Function(Object) wrapper = (Object e) => handler(e as T);
    _handlers.putIfAbsent(T, () => []).add(wrapper);
    return () {
      _handlers[T]?.remove(wrapper);
    };
  }

  void publish(Object event) {
    final handlers = _handlers[event.runtimeType];
    if (handlers == null) return;
    for (final h in List.of(handlers)) {
      h(event);
    }
  }
}

class UserLoggedIn {
  final String name;

  const UserLoggedIn(this.name);
}

class CartUpdated {
  final int itemsCount;

  const CartUpdated(this.itemsCount);
}

void main() {
  final bus = EventBus();

  final unsubscribeLogin = bus.subscribe<UserLoggedIn>(
    (e) => print('Приветствие: добро пожаловать, ${e.name}!'),
  );
  bus.subscribe<UserLoggedIn>((e) => print('Аналитика: вход пользователя ${e.name}'));
  bus.subscribe<CartUpdated>((e) => print('Бейдж корзины: ${e.itemsCount}'));

  bus.publish(const UserLoggedIn('Анна'));
  bus.publish(const CartUpdated(3));

  unsubscribeLogin(); // отписываем первого слушателя
  print('--- после отписки ---');
  bus.publish(const UserLoggedIn('Иван')); // остался только слушатель аналитики
}
```

### №50. Комплексный архитектурный модуль заказа такси (Production Case)

```dart
import 'dart:math';

class GeoPoint {
  final double lat;
  final double lon;

  const GeoPoint(this.lat, this.lon);
}

// ---------- Sealed-состояния заказа ----------
sealed class TaxiOrderState {
  const TaxiOrderState();
}

class SearchingState extends TaxiOrderState {
  const SearchingState();
}

class DriverAssignedState extends TaxiOrderState {
  final String driverName;
  final String carPlate;
  final int etaMinutes;

  const DriverAssignedState(this.driverName, this.carPlate, this.etaMinutes);
}

class InTripState extends TaxiOrderState {
  final DateTime startedAt;

  const InTripState(this.startedAt);
}

class CompletedState extends TaxiOrderState {
  final double fare;
  final double distanceKm;

  const CompletedState(this.fare, this.distanceKm);
}

class CancelledState extends TaxiOrderState {
  final String reason;

  const CancelledState(this.reason);
}

// ---------- Миксины ----------
double _haversineKm(GeoPoint a, GeoPoint b) {
  const earthRadiusKm = 6371.0;
  double rad(double deg) => deg * pi / 180;
  final dLat = rad(b.lat - a.lat);
  final dLon = rad(b.lon - a.lon);
  final h = sin(dLat / 2) * sin(dLat / 2) +
      cos(rad(a.lat)) * cos(rad(b.lat)) * sin(dLon / 2) * sin(dLon / 2);
  return 2 * earthRadiusKm * asin(sqrt(h));
}

mixin LocationTrackingMixin {
  final List<GeoPoint> _track = [];

  void updateLocation(double lat, double lon) => _track.add(GeoPoint(lat, lon));

  double get distanceKm {
    var total = 0.0;
    for (var i = 1; i < _track.length; i++) {
      total += _haversineKm(_track[i - 1], _track[i]);
    }
    return total;
  }
}

mixin FareCalculationMixin {
  double get tariffMultiplier;

  double calculateFare(double km, int minutes) {
    const base = 100.0;
    const perKm = 15.0;
    const perMinute = 5.0;
    return (base + km * perKm + minutes * perMinute) * tariffMultiplier;
  }
}

// ---------- Заказ: фабричные конструкторы и список инициализации ----------
class TaxiOrder with LocationTrackingMixin, FareCalculationMixin {
  final String id;
  final String from;
  final String to;
  final String tariffName;
  final double _multiplier;
  final DateTime createdAt;

  TaxiOrder._(this.id, this.from, this.to, this.tariffName, double multiplier)
      : assert(id.isNotEmpty, 'Номер заказа не может быть пустым'),
        assert(multiplier >= 1.0, 'Множитель тарифа не может быть меньше 1'),
        _multiplier = multiplier,
        createdAt = DateTime.now();

  factory TaxiOrder.economy(String id, String from, String to) =>
      TaxiOrder._(id, from, to, 'Эконом', 1.0);

  factory TaxiOrder.comfort(String id, String from, String to) =>
      TaxiOrder._(id, from, to, 'Комфорт', 1.5);

  factory TaxiOrder.business(String id, String from, String to) =>
      TaxiOrder._(id, from, to, 'Бизнес', 2.2);

  @override
  double get tariffMultiplier => _multiplier;
}

// ---------- Контроллер поездки: исчерпывающие switch ----------
class TripController {
  final TaxiOrder order;
  TaxiOrderState _state = const SearchingState();

  TripController(this.order);

  TaxiOrderState get state => _state;

  void assignDriver(String name, String plate, int etaMinutes) {
    _state = switch (_state) {
      SearchingState() => DriverAssignedState(name, plate, etaMinutes),
      DriverAssignedState() || InTripState() || CompletedState() || CancelledState() => _state,
    };
  }

  void startTrip() {
    _state = switch (_state) {
      DriverAssignedState() => InTripState(DateTime.now()),
      SearchingState() || InTripState() || CompletedState() || CancelledState() => _state,
    };
  }

  void finishTrip({required int durationMinutes}) {
    _state = switch (_state) {
      InTripState() => CompletedState(
          order.calculateFare(order.distanceKm, durationMinutes),
          order.distanceKm,
        ),
      SearchingState() || DriverAssignedState() || CompletedState() || CancelledState() => _state,
    };
  }

  // Отменить можно только до начала поездки
  void cancel(String reason) {
    _state = switch (_state) {
      SearchingState() || DriverAssignedState() => CancelledState(reason),
      InTripState() || CompletedState() || CancelledState() => _state,
    };
  }

  String describe() => switch (_state) {
        SearchingState() => '🔍 Ищем машину (${order.tariffName}): ${order.from} → ${order.to}',
        DriverAssignedState(:final driverName, :final carPlate, :final etaMinutes) =>
          '🚕 $driverName ($carPlate) приедет через $etaMinutes мин.',
        InTripState(:final startedAt) =>
          '🛣  Поездка идёт с ${startedAt.toIso8601String().substring(11, 19)}',
        CompletedState(:final fare, :final distanceKm) =>
          '✅ Завершено: ${distanceKm.toStringAsFixed(2)} км, ${fare.toStringAsFixed(2)} ₽',
        CancelledState(:final reason) => '❌ Заказ отменён: $reason',
      };
}

void main() {
  // Сценарий 1: полная поездка
  final trip = TripController(TaxiOrder.comfort('A-100', 'Красная площадь', 'ВДНХ'));
  print(trip.describe());

  trip.assignDriver('Сергей', 'А123ВС', 4);
  print(trip.describe());

  trip.startTrip();
  trip.order.updateLocation(55.7539, 37.6208);
  trip.order.updateLocation(55.7700, 37.6450);
  trip.order.updateLocation(55.8264, 37.6377);
  print(trip.describe());

  trip.cancel('передумал'); // во время поездки отменить нельзя — состояние не изменится
  trip.finishTrip(durationMinutes: 22);
  print(trip.describe());

  // Сценарий 2: отмена до начала поездки
  final second = TripController(TaxiOrder.economy('A-101', 'Дом', 'Работа'));
  second.assignDriver('Ольга', 'К777МР', 2);
  second.cancel('долго ждать');
  print(second.describe());
}
```
