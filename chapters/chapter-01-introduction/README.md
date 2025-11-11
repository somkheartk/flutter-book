# บทที่ 1: รู้จักกับ Flutter

## 1.1 Flutter คืออะไร

Flutter เป็น UI Framework แบบ Open Source ที่พัฒนาโดย Google สำหรับสร้างแอปพลิเคชันข้ามแพลตฟอร์ม (Cross-platform) จากโค้ดเบสเดียว คุณสามารถพัฒนาแอปสำหรับ:

- **Mobile**: iOS และ Android
- **Web**: Progressive Web Apps (PWA)
- **Desktop**: Windows, macOS, Linux
- **Embedded**: สำหรับอุปกรณ์ IoT

### จุดเด่นของ Flutter

1. **Hot Reload**: เห็นผลการเปลี่ยนแปลงทันทีโดยไม่ต้อง restart แอป
2. **Beautiful UI**: มี Material Design และ Cupertino widgets ในตัว
3. **High Performance**: ใช้ Dart language และ compile เป็น native code
4. **Single Codebase**: เขียนโค้ดครั้งเดียว รันได้หลายแพลตฟอร์ม
5. **Rich Ecosystem**: มี packages มากมายบน pub.dev

## 1.2 ทำไมต้องเรียน Flutter

### ข้อดี
- 🚀 **เร็ว**: Hot reload ทำให้พัฒนาได้เร็วขึ้น
- 💰 **ประหยัด**: ไม่ต้องพัฒนาแยกทีมสำหรับแต่ละแพลตฟอร์ม
- 🎨 **สวยงาม**: ควบคุม UI ได้ในระดับ pixel
- 📚 **เรียนรู้ง่าย**: Dart language มี syntax ที่เข้าใจง่าย
- 🌟 **Community ใหญ่**: มีคนใช้และพัฒนามากมาย

### กรณีการใช้งาน
- Startup ที่ต้องการ MVP เร็ว
- บริษัทที่ต้องการลดต้นทุนการพัฒนา
- นักพัฒนาที่ต้องการสร้างแอปหลายแพลตฟอร์ม
- โปรเจคที่ต้องการ UI ที่สวยงามและ custom ได้สูง

## 1.3 การติดตั้ง Flutter

### ความต้องการของระบบ

**Windows:**
- Operating System: Windows 10 หรือสูงกว่า (64-bit)
- Disk Space: 2.5 GB (ไม่รวม IDE/tools)
- Tools: Git for Windows

**macOS:**
- Operating System: macOS 10.14 (Mojave) หรือสูงกว่า
- Disk Space: 2.8 GB
- Tools: Xcode, CocoaPods

**Linux:**
- Operating System: Ubuntu 18.04 หรือสูงกว่า
- Disk Space: 600 MB
- Tools: bash, curl, git, mkdir, rm, unzip, which, xz-utils

### ขั้นตอนการติดตั้ง

#### Windows
```bash
# 1. ดาวน์โหลด Flutter SDK
# ไปที่ https://flutter.dev/docs/get-started/install/windows

# 2. แตกไฟล์ไปที่ C:\src\flutter

# 3. เพิ่ม Flutter ใน PATH
# เพิ่ม C:\src\flutter\bin ใน Environment Variables

# 4. ตรวจสอบการติดตั้ง
flutter doctor
```

#### macOS
```bash
# 1. ดาวน์โหลด Flutter SDK
cd ~/development
git clone https://github.com/flutter/flutter.git -b stable

# 2. เพิ่ม Flutter ใน PATH
export PATH="$PATH:`pwd`/flutter/bin"

# 3. ติดตั้ง Xcode
# ดาวน์โหลดจาก App Store

# 4. ติดตั้ง CocoaPods
sudo gem install cocoapods

# 5. ตรวจสอบการติดตั้ง
flutter doctor
```

#### Linux
```bash
# 1. ติดตั้ง dependencies
sudo apt-get update
sudo apt-get install curl git unzip xz-utils zip libglu1-mesa

# 2. ดาวน์โหลด Flutter SDK
cd ~/development
git clone https://github.com/flutter/flutter.git -b stable

# 3. เพิ่ม Flutter ใน PATH
export PATH="$PATH:`pwd`/flutter/bin"

# 4. ตรวจสอบการติดตั้ง
flutter doctor
```

### การติดตั้ง IDE

#### Visual Studio Code
```bash
# 1. ติดตั้ง VS Code
# ดาวน์โหลดจาก https://code.visualstudio.com/

# 2. ติดตั้ง Flutter Extension
# ค้นหา "Flutter" ใน Extensions marketplace
```

#### Android Studio
```bash
# 1. ติดตั้ง Android Studio
# ดาวน์โหลดจาก https://developer.android.com/studio

# 2. ติดตั้ง Flutter และ Dart plugins
# ไปที่ Preferences > Plugins
```

### ตรวจสอบการติดตั้ง

```bash
flutter doctor -v
```

ผลลัพธ์ที่ดี:
```
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 3.16.0, on macOS 14.0)
[✓] Android toolchain - develop for Android devices
[✓] Xcode - develop for iOS and macOS
[✓] Chrome - develop for the web
[✓] Android Studio (version 2023.1)
[✓] VS Code (version 1.85.0)
[✓] Connected device (3 available)
```

## 1.4 โครงสร้างโปรเจค Flutter

เมื่อสร้างโปรเจค Flutter ใหม่ จะได้โครงสร้างดังนี้:

```
my_app/
├── android/           # โค้ดสำหรับ Android
├── ios/              # โค้ดสำหรับ iOS
├── lib/              # โค้ด Dart หลัก
│   └── main.dart    # จุดเริ่มต้นของแอป
├── test/            # โค้ดสำหรับ testing
├── web/             # โค้ดสำหรับ Web
├── pubspec.yaml     # ไฟล์ config และ dependencies
└── README.md        # เอกสารโปรเจค
```

### ไฟล์สำคัญ

#### pubspec.yaml
```yaml
name: my_app
description: My Flutter application
version: 1.0.0+1

environment:
  sdk: '>=3.0.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.2

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^2.0.0

flutter:
  uses-material-design: true
```

#### lib/main.dart
```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}
```

## 1.5 สร้างแอปแรกของคุณ

### สร้างโปรเจคใหม่

```bash
# สร้างโปรเจค Flutter ใหม่
flutter create my_first_app

# เข้าไปในโฟลเดอร์
cd my_first_app

# รันแอป
flutter run
```

### ตัวอย่าง: Hello World App

สร้างแอปง่ายๆ ที่แสดงข้อความ "Hello, Flutter!"

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const HelloWorldApp());
}

class HelloWorldApp extends StatelessWidget {
  const HelloWorldApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Hello World',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: Scaffold(
        appBar: AppBar(
          title: const Text('My First App'),
        ),
        body: const Center(
          child: Text(
            'Hello, Flutter!',
            style: TextStyle(
              fontSize: 32,
              fontWeight: FontWeight.bold,
              color: Colors.blue,
            ),
          ),
        ),
      ),
    );
  }
}
```

### ตัวอย่าง: Counter App

แอปนับจำนวนที่มีปุ่มกดเพิ่มค่า

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const CounterApp());
}

class CounterApp extends StatelessWidget {
  const CounterApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Counter App',
      theme: ThemeData(
        primarySwatch: Colors.green,
      ),
      home: const CounterPage(),
    );
  }
}

class CounterPage extends StatefulWidget {
  const CounterPage({super.key});

  @override
  State<CounterPage> createState() => _CounterPageState();
}

class _CounterPageState extends State<CounterPage> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  void _decrementCounter() {
    setState(() {
      _counter--;
    });
  }

  void _resetCounter() {
    setState(() {
      _counter = 0;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Counter App'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text(
              'คุณกดปุ่มไปแล้ว:',
              style: TextStyle(fontSize: 20),
            ),
            Text(
              '$_counter',
              style: Theme.of(context).textTheme.headlineLarge,
            ),
            const SizedBox(height: 30),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                FloatingActionButton(
                  onPressed: _decrementCounter,
                  tooltip: 'ลด',
                  child: const Icon(Icons.remove),
                ),
                const SizedBox(width: 20),
                FloatingActionButton(
                  onPressed: _resetCounter,
                  tooltip: 'รีเซ็ต',
                  child: const Icon(Icons.refresh),
                ),
                const SizedBox(width: 20),
                FloatingActionButton(
                  onPressed: _incrementCounter,
                  tooltip: 'เพิ่ม',
                  child: const Icon(Icons.add),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

### การรันแอป

```bash
# รันบน emulator/device
flutter run

# รันบน web
flutter run -d chrome

# รันแบบ release mode
flutter run --release

# Hot reload (กด 'r' ใน terminal)
# Hot restart (กด 'R' ใน terminal)
# Quit (กด 'q' ใน terminal)
```

## แบบฝึกหัด

1. ติดตั้ง Flutter และตรวจสอบด้วย `flutter doctor`
2. สร้างโปรเจค Flutter แรกของคุณ
3. รัน Hello World App และทดสอบ Hot Reload
4. แก้ไข Counter App ให้เพิ่มปุ่มคูณ 2
5. เปลี่ยนสีธีมของแอปเป็นสีที่คุณชอบ

## สรุป

ในบทนี้เราได้เรียนรู้:
- Flutter คืออะไรและทำไมถึงน่าสนใจ
- วิธีการติดตั้ง Flutter และ tools ที่จำเป็น
- โครงสร้างโปรเจค Flutter
- สร้างแอปแรกและเข้าใจพื้นฐานของ Flutter

ในบทถัดไป เราจะเรียนรู้เกี่ยวกับ Dart Programming Language ซึ่งเป็นภาษาที่ใช้เขียน Flutter!

---

**หัวข้อถัดไป**: [บทที่ 2: พื้นฐาน Dart Programming](../chapter-02-dart-basics/README.md)
