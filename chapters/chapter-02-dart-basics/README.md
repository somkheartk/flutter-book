# บทที่ 2: พื้นฐาน Dart Programming

## 2.1 ตัวแปรและชนิดข้อมูล

Dart เป็นภาษาที่มี strong typing แต่รองรับ type inference

### การประกาศตัวแปร

```dart
// แบบระบุ type
String name = 'สมชาย';
int age = 25;
double height = 175.5;
bool isStudent = true;

// แบบให้ Dart inference type เอง
var city = 'กรุงเทพ';  // String
var score = 95;         // int
var price = 99.99;      // double
var isActive = false;   // bool

// ค่าคงที่
final String country = 'Thailand';  // กำหนดค่าได้ครั้งเดียว runtime
const double pi = 3.14159;          // กำหนดค่าได้ครั้งเดียว compile time

// ตัวแปรที่เป็น null ได้
String? nickname;  // nullable
int? count;        // nullable
```

### ชนิดข้อมูลพื้นฐาน

#### Numbers
```dart
// Integers
int quantity = 10;
int hexValue = 0xFF;
int large = 9999999999999;

// Doubles
double temperature = 36.5;
double scientific = 1.42e5;  // 142000.0

// Operations
int sum = 10 + 5;        // 15
int difference = 10 - 5;  // 5
int product = 10 * 5;     // 50
double quotient = 10 / 5; // 2.0
int intDiv = 10 ~/ 3;     // 3 (integer division)
int remainder = 10 % 3;   // 1

// Conversion
int numInt = 42;
double numDouble = numInt.toDouble();  // 42.0
String numString = numInt.toString();  // '42'
```

#### Strings
```dart
// String literals
String single = 'Hello';
String double = "World";
String multiline = '''
  This is a
  multi-line
  string
''';

// String interpolation
String name = 'อรุณ';
int age = 30;
String message = 'สวัสดี $name อายุ $age ปี';
String calculation = 'ผลลัพธ์: ${2 + 2}';  // ผลลัพธ์: 4

// String operations
String text = 'Flutter';
print(text.length);           // 7
print(text.toUpperCase());    // FLUTTER
print(text.toLowerCase());    // flutter
print(text.contains('utter')); // true
print(text.substring(0, 4));  // Flut
print(text.split('t'));       // [Flu, , er]

// Raw strings
String path = r'C:\Users\Documents';  // ไม่ escape characters
```

#### Booleans
```dart
bool isTrue = true;
bool isFalse = false;

// Logical operations
bool and = true && false;   // false
bool or = true || false;    // true
bool not = !true;           // false

// Comparison
bool equal = 5 == 5;        // true
bool notEqual = 5 != 3;     // true
bool greater = 10 > 5;      // true
bool less = 5 < 10;         // true
```

#### Lists (Arrays)
```dart
// List literal
List<String> fruits = ['แอปเปิ้ล', 'กล้วย', 'ส้ม'];
var numbers = [1, 2, 3, 4, 5];  // List<int>

// List operations
fruits.add('มะม่วง');           // เพิ่มท้ายสุด
fruits.insert(0, 'ทุเรียน');    // เพิ่มที่ตำแหน่ง 0
fruits.remove('กล้วย');         // ลบตามค่า
fruits.removeAt(0);             // ลบตามตำแหน่ง
print(fruits.length);           // จำนวนสมาชิก
print(fruits[0]);               // เข้าถึงตำแหน่ง 0
print(fruits.contains('ส้ม'));  // ตรวจสอบว่ามีหรือไม่

// List methods
numbers.forEach((n) => print(n));
var doubled = numbers.map((n) => n * 2).toList();
var evens = numbers.where((n) => n % 2 == 0).toList();

// Spread operator
var moreNumbers = [0, ...numbers, 6, 7];  // [0, 1, 2, 3, 4, 5, 6, 7]
```

#### Sets
```dart
// Set literal - ไม่มีค่าซ้ำ
Set<String> colors = {'แดง', 'เขียว', 'น้ำเงิน'};
var uniqueNumbers = {1, 2, 3, 2, 1};  // {1, 2, 3}

// Set operations
colors.add('เหลือง');
colors.remove('แดง');
print(colors.contains('เขียว'));  // true
```

#### Maps (Dictionaries)
```dart
// Map literal
Map<String, int> scores = {
  'สมชาย': 85,
  'สมหญิง': 90,
  'สมศักดิ์': 78,
};

// Dynamic map
var person = {
  'name': 'อรุณ',
  'age': 25,
  'city': 'กรุงเทพ',
};

// Map operations
scores['สมชาย'] = 88;           // อัพเดทค่า
scores['สมหมาย'] = 92;          // เพิ่มคู่ key-value ใหม่
scores.remove('สมศักดิ์');      // ลบ
print(scores.containsKey('สมชาย'));  // true
print(scores.length);           // จำนวนคู่

// Iteration
scores.forEach((name, score) {
  print('$name ได้ $score คะแนน');
});
```

## 2.2 การควบคุมโฟลว์

### If-Else Statements

```dart
int score = 85;

if (score >= 80) {
  print('เกรด A');
} else if (score >= 70) {
  print('เกรด B');
} else if (score >= 60) {
  print('เกรด C');
} else if (score >= 50) {
  print('เกรด D');
} else {
  print('เกรด F');
}

// Ternary operator
String result = score >= 50 ? 'ผ่าน' : 'ไม่ผ่าน';

// Null-aware operators
String? name;
String displayName = name ?? 'ไม่ระบุชื่อ';  // ถ้า name เป็น null ใช้ 'ไม่ระบุชื่อ'
```

### Switch-Case Statements

```dart
String grade = 'A';

switch (grade) {
  case 'A':
    print('ดีมาก');
    break;
  case 'B':
    print('ดี');
    break;
  case 'C':
    print('ปานกลาง');
    break;
  case 'D':
    print('พอใช้');
    break;
  default:
    print('ไม่ผ่าน');
}

// Switch expression (Dart 3.0+)
String description = switch (grade) {
  'A' => 'ดีมาก',
  'B' => 'ดี',
  'C' => 'ปานกลาง',
  'D' => 'พอใช้',
  _ => 'ไม่ผ่าน',
};
```

### Loops

#### For Loop
```dart
// Traditional for loop
for (int i = 0; i < 5; i++) {
  print('รอบที่ $i');
}

// For-in loop
List<String> fruits = ['แอปเปิ้ล', 'กล้วย', 'ส้ม'];
for (var fruit in fruits) {
  print(fruit);
}

// forEach method
fruits.forEach((fruit) {
  print(fruit);
});
```

#### While Loop
```dart
int count = 0;
while (count < 5) {
  print('Count: $count');
  count++;
}

// Do-while loop
int num = 0;
do {
  print('Number: $num');
  num++;
} while (num < 5);
```

#### Break และ Continue
```dart
// Break - ออกจาก loop
for (int i = 0; i < 10; i++) {
  if (i == 5) break;
  print(i);  // พิมพ์ 0-4
}

// Continue - ข้ามรอบปัจจุบัน
for (int i = 0; i < 10; i++) {
  if (i % 2 == 0) continue;
  print(i);  // พิมพ์เฉพาะเลขคี่
}
```

## 2.3 Functions และ Methods

### การประกาศ Function

```dart
// Function พื้นฐาน
void greet() {
  print('สวัสดี!');
}

// Function ที่มี parameter
void greetPerson(String name) {
  print('สวัสดี $name!');
}

// Function ที่ return ค่า
int add(int a, int b) {
  return a + b;
}

// Arrow function (สำหรับ function บรรทัดเดียว)
int multiply(int a, int b) => a * b;

// การเรียกใช้
greet();
greetPerson('สมชาย');
int sum = add(5, 3);
print(multiply(4, 5));
```

### Optional Parameters

```dart
// Named parameters
void createUser({String name = '', int age = 0, String? email}) {
  print('Name: $name, Age: $age, Email: ${email ?? "N/A"}');
}

// Required named parameters
void login({required String username, required String password}) {
  print('Logging in: $username');
}

// การเรียกใช้
createUser(name: 'สมชาย', age: 25);
createUser(name: 'สมหญิง', age: 30, email: 'som@example.com');
login(username: 'user1', password: 'pass123');

// Positional optional parameters
String formatName(String firstName, [String? lastName]) {
  if (lastName != null) {
    return '$firstName $lastName';
  }
  return firstName;
}

print(formatName('สมชาย'));            // สมชาย
print(formatName('สมชาย', 'ใจดี'));    // สมชาย ใจดี
```

### Higher-Order Functions

```dart
// Function ที่รับ function เป็น parameter
void executeOperation(int a, int b, Function operation) {
  print(operation(a, b));
}

executeOperation(10, 5, (a, b) => a + b);  // 15
executeOperation(10, 5, (a, b) => a * b);  // 50

// Function ที่ return function
Function makeMultiplier(int factor) {
  return (int value) => value * factor;
}

var triple = makeMultiplier(3);
print(triple(5));  // 15
```

## 2.4 Classes และ Objects

### การสร้าง Class

```dart
// Class พื้นฐาน
class Person {
  // Properties
  String name;
  int age;
  
  // Constructor
  Person(this.name, this.age);
  
  // Method
  void introduce() {
    print('สวัสดี ผม/ดิฉัน$name อายุ $age ปี');
  }
  
  // Getter
  bool get isAdult => age >= 18;
  
  // Setter
  set updateAge(int newAge) {
    if (newAge > 0) {
      age = newAge;
    }
  }
}

// การใช้งาน
var person = Person('สมชาย', 25);
person.introduce();
print(person.isAdult);  // true
person.updateAge = 30;
```

### Named Constructor

```dart
class User {
  String username;
  String email;
  int age;
  
  // Default constructor
  User(this.username, this.email, this.age);
  
  // Named constructor
  User.guest()
      : username = 'guest',
        email = 'guest@example.com',
        age = 0;
  
  User.fromJson(Map<String, dynamic> json)
      : username = json['username'],
        email = json['email'],
        age = json['age'];
}

// การใช้งาน
var user1 = User('john', 'john@example.com', 25);
var user2 = User.guest();
var user3 = User.fromJson({
  'username': 'jane',
  'email': 'jane@example.com',
  'age': 30,
});
```

### Inheritance (การสืบทอด)

```dart
// Parent class
class Animal {
  String name;
  int age;
  
  Animal(this.name, this.age);
  
  void makeSound() {
    print('$name makes a sound');
  }
}

// Child class
class Dog extends Animal {
  String breed;
  
  Dog(String name, int age, this.breed) : super(name, age);
  
  @override
  void makeSound() {
    print('$name barks: Woof! Woof!');
  }
  
  void fetch() {
    print('$name fetches the ball');
  }
}

// การใช้งาน
var dog = Dog('บัดดี้', 3, 'Golden Retriever');
dog.makeSound();  // บัดดี้ barks: Woof! Woof!
dog.fetch();      // บัดดี้ fetches the ball
```

### Abstract Class

```dart
// Abstract class - ไม่สามารถสร้าง instance ได้โดยตรง
abstract class Shape {
  // Abstract method - ต้อง implement ใน subclass
  double getArea();
  double getPerimeter();
  
  // Concrete method
  void describe() {
    print('This is a shape with area ${getArea()}');
  }
}

class Rectangle extends Shape {
  double width;
  double height;
  
  Rectangle(this.width, this.height);
  
  @override
  double getArea() => width * height;
  
  @override
  double getPerimeter() => 2 * (width + height);
}

class Circle extends Shape {
  double radius;
  
  Circle(this.radius);
  
  @override
  double getArea() => 3.14159 * radius * radius;
  
  @override
  double getPerimeter() => 2 * 3.14159 * radius;
}

// การใช้งาน
var rect = Rectangle(5, 10);
print('พื้นที่: ${rect.getArea()}');        // 50.0
print('เส้นรอบรูป: ${rect.getPerimeter()}'); // 30.0

var circle = Circle(5);
print('พื้นที่: ${circle.getArea()}');       // 78.53975
```

### Mixins

```dart
// Mixin - เพิ่มความสามารถให้กับ class
mixin Swimmer {
  void swim() {
    print('Swimming...');
  }
}

mixin Flyer {
  void fly() {
    print('Flying...');
  }
}

class Bird with Flyer {
  String name;
  Bird(this.name);
}

class Duck extends Bird with Swimmer {
  Duck(String name) : super(name);
}

// การใช้งาน
var duck = Duck('โดนัล');
duck.fly();   // Flying...
duck.swim();  // Swimming...
```

## 2.5 Null Safety

Dart มี Null Safety เพื่อป้องกัน null reference errors

### Nullable vs Non-nullable

```dart
// Non-nullable (ต้องมีค่า)
String name = 'สมชาย';
int age = 25;
// name = null;  // Error! ไม่สามารถใส่ null ได้

// Nullable (สามารถเป็น null ได้)
String? nickname;  // null โดย default
int? score;        // null โดย default
nickname = 'ชาย';
nickname = null;   // OK
```

### Null-aware Operators

```dart
String? name;

// ?? operator - ใช้ค่าด้านขวาถ้าด้านซ้ายเป็น null
String displayName = name ?? 'ไม่ระบุชื่อ';

// ??= operator - กำหนดค่าถ้ายังเป็น null
name ??= 'ค่าเริ่มต้น';

// ?. operator - เรียก method ถ้าไม่เป็น null
String? email;
int? length = email?.length;  // null ถ้า email เป็น null

// ! operator - บอก compiler ว่าแน่ใจว่าไม่ใช่ null
String definitelyNotNull = name!;  // ถ้า name เป็น null จะ error ตอน runtime
```

### Late Variables

```dart
// late - ประกาศตัวแปรที่จะกำหนดค่าทีหลัง
class User {
  late String name;
  
  void initialize() {
    name = 'John';
  }
  
  void printName() {
    print(name);  // ถ้ายังไม่ initialize จะ error
  }
}

// late final - กำหนดค่าได้ครั้งเดียวแต่ทีหลัง
late final String config;

void loadConfig() {
  config = 'loaded config';
}
```

## แบบฝึกหัด

1. **ตัวแปร**: สร้างตัวแปรเก็บข้อมูลส่วนตัว (ชื่อ, อายุ, ส่วนสูง, น้ำหนัก)
2. **List**: สร้าง List เก็บชื่อเพื่อน 5 คน แล้วพิมพ์ชื่อทั้งหมด
3. **Function**: เขียน function คำนวณ BMI (BMI = น้ำหนัก / ส่วนสูง²)
4. **Class**: สร้าง class Student ที่มี properties: ชื่อ, อายุ, เกรด และ method แสดงข้อมูล
5. **Null Safety**: เขียน function รับ nullable String แล้ว return ความยาวหรือ 0 ถ้าเป็น null

## เฉลยแบบฝึกหัด

```dart
// 1. ตัวแปร
String myName = 'สมชาย';
int myAge = 25;
double myHeight = 175.0;
double myWeight = 70.0;

// 2. List
List<String> friends = ['อรุณ', 'สมหญิง', 'สมศักดิ์', 'วิชัย', 'มานะ'];
for (var friend in friends) {
  print(friend);
}

// 3. Function BMI
double calculateBMI(double weight, double heightInCm) {
  double heightInM = heightInCm / 100;
  return weight / (heightInM * heightInM);
}

void main() {
  double bmi = calculateBMI(70, 175);
  print('BMI: ${bmi.toStringAsFixed(2)}');
}

// 4. Class Student
class Student {
  String name;
  int age;
  String grade;
  
  Student(this.name, this.age, this.grade);
  
  void displayInfo() {
    print('ชื่อ: $name, อายุ: $age ปี, เกรด: $grade');
  }
}

void testStudent() {
  var student = Student('สมชาย', 20, 'A');
  student.displayInfo();
}

// 5. Null Safety
int getStringLength(String? text) {
  return text?.length ?? 0;
}

void testNullSafety() {
  print(getStringLength('Hello'));   // 5
  print(getStringLength(null));      // 0
}
```

## สรุป

ในบทนี้เราได้เรียนรู้:
- ตัวแปรและชนิดข้อมูลใน Dart (String, int, double, bool, List, Set, Map)
- การควบคุมโฟลว์ (if-else, switch, loops)
- Functions และ optional parameters
- Classes, Objects, Inheritance, Abstract classes, Mixins
- Null Safety และ null-aware operators

ทักษะเหล่านี้เป็นพื้นฐานสำคัญในการเขียน Flutter!

---

**หัวข้อก่อนหน้า**: [บทที่ 1: รู้จักกับ Flutter](../chapter-01-introduction/README.md)  
**หัวข้อถัดไป**: [บทที่ 3: Flutter Widgets พื้นฐาน](../chapter-03-widgets/README.md)
