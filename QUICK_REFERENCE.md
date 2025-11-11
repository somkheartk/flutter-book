# Quick Reference Guide

คู่มืออ้างอิงด่วนสำหรับ Flutter

## Widgets พื้นฐาน

### Layout
```dart
Column()              // แนวตั้ง
Row()                 // แนวนอน
Stack()               // ซ้อนทับ
Container()           // กล่องที่ปรับแต่งได้
Padding()             // เว้นระยะ
Center()              // จัดกลาง
Expanded()            // ขยายเต็มที่
SizedBox()            // กำหนดขนาด
```

### Text & Images
```dart
Text()                // ข้อความ
Image.asset()         // รูปจาก assets
Image.network()       // รูปจาก URL
Icon()                // ไอคอน
```

### Buttons
```dart
ElevatedButton()      // ปุ่มนูน
TextButton()          // ปุ่มข้อความ
OutlinedButton()      // ปุ่มมีกรอบ
IconButton()          // ปุ่มไอคอน
FloatingActionButton() // ปุ่มลอย
```

### Input
```dart
TextField()           // ช่องกรอกข้อความ
Checkbox()            // ช่องติ๊ก
Radio()               // ปุ่มเลือก
Switch()              // สวิตช์
Slider()              // แถบเลื่อน
DropdownButton()      // เมนูดรอปดาวน์
```

## State Management

### setState
```dart
setState(() {
  counter++;
});
```

### Provider
```dart
// Provide
ChangeNotifierProvider(
  create: (_) => MyModel(),
)

// Consume
Consumer<MyModel>(
  builder: (context, model, child) {
    return Text(model.data);
  },
)

// Read
context.read<MyModel>().method();

// Watch
context.watch<MyModel>().data;
```

## Navigation

```dart
// Push
Navigator.push(context, MaterialPageRoute(
  builder: (context) => SecondPage(),
));

// Pop
Navigator.pop(context);

// Named routes
Navigator.pushNamed(context, '/second');

// With result
final result = await Navigator.push(...);
Navigator.pop(context, 'result');
```

## API Calls

```dart
// GET
final response = await http.get(Uri.parse(url));

// POST
final response = await http.post(
  Uri.parse(url),
  headers: {'Content-Type': 'application/json'},
  body: jsonEncode(data),
);

// Parse JSON
final data = jsonDecode(response.body);
```

## Storage

### SharedPreferences
```dart
final prefs = await SharedPreferences.getInstance();
await prefs.setString('key', 'value');
final value = prefs.getString('key');
```

### Hive
```dart
// Init
await Hive.initFlutter();
await Hive.openBox('myBox');

// Save
final box = Hive.box('myBox');
await box.put('key', 'value');

// Read
final value = box.get('key');
```

## Common Patterns

### FutureBuilder
```dart
FutureBuilder<Data>(
  future: fetchData(),
  builder: (context, snapshot) {
    if (snapshot.hasData) {
      return DataWidget(snapshot.data!);
    }
    if (snapshot.hasError) {
      return ErrorWidget(snapshot.error);
    }
    return CircularProgressIndicator();
  },
)
```

### ListView.builder
```dart
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return ListTile(
      title: Text(items[index]),
    );
  },
)
```

### Form
```dart
final _formKey = GlobalKey<FormState>();

Form(
  key: _formKey,
  child: Column(
    children: [
      TextFormField(
        validator: (value) {
          if (value == null || value.isEmpty) {
            return 'กรุณากรอกข้อมูล';
          }
          return null;
        },
      ),
      ElevatedButton(
        onPressed: () {
          if (_formKey.currentState!.validate()) {
            // Process data
          }
        },
        child: Text('Submit'),
      ),
    ],
  ),
)
```

## CLI Commands

```bash
flutter create my_app     # สร้างโปรเจค
flutter run               # รันแอป
flutter pub get           # ติดตั้ง packages
flutter build apk         # Build Android
flutter build ios         # Build iOS
flutter test              # รัน tests
flutter clean             # Clean project
```

## Keyboard Shortcuts

### VS Code
- `r` - Hot reload
- `R` - Hot restart  
- `q` - Quit
- `Ctrl/Cmd + Space` - Autocomplete
- `Ctrl/Cmd + .` - Quick fixes

---

**See full book**: [README.md](README.md)
