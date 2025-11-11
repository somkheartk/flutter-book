# บทที่ 3: Flutter Widgets พื้นฐาน

Widgets คือหัวใจของ Flutter - ทุกอย่างคือ Widget! ตั้งแต่ปุ่ม ข้อความ layout ไปจนถึงแอปทั้งหมด

## 3.1 StatelessWidget vs StatefulWidget

### StatelessWidget

Widget ที่ไม่เปลี่ยนแปลง (Immutable) ไม่มี state ภายใน

```dart
import 'package:flutter/material.dart';

class GreetingWidget extends StatelessWidget {
  final String name;
  
  const GreetingWidget({super.key, required this.name});
  
  @override
  Widget build(BuildContext context) {
    return Text(
      'สวัสดี $name!',
      style: const TextStyle(fontSize: 24),
    );
  }
}

// การใช้งาน
class MyApp extends StatelessWidget {
  const MyApp({super.key});
  
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Stateless Example')),
        body: const Center(
          child: GreetingWidget(name: 'สมชาย'),
        ),
      ),
    );
  }
}
```

### StatefulWidget

Widget ที่เปลี่ยนแปลงได้ มี state ภายในที่สามารถอัพเดทได้

```dart
import 'package:flutter/material.dart';

class CounterWidget extends StatefulWidget {
  const CounterWidget({super.key});
  
  @override
  State<CounterWidget> createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  int _count = 0;
  
  void _increment() {
    setState(() {
      _count++;
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        Text(
          'Count: $_count',
          style: const TextStyle(fontSize: 32),
        ),
        const SizedBox(height: 20),
        ElevatedButton(
          onPressed: _increment,
          child: const Text('เพิ่ม'),
        ),
      ],
    );
  }
}
```

### เมื่อไหร่ใช้อะไร?

- **StatelessWidget**: ใช้เมื่อ UI ไม่เปลี่ยนแปลง เช่น ข้อความคงที่, ไอคอน, layout ที่ไม่ต้องอัพเดท
- **StatefulWidget**: ใช้เมื่อ UI เปลี่ยนแปลงตาม interaction หรือ data เช่น form, counter, animation

## 3.2 Layout Widgets

### Row - จัด Widget ในแนวนอน

```dart
Row(
  mainAxisAlignment: MainAxisAlignment.spaceEvenly,
  crossAxisAlignment: CrossAxisAlignment.center,
  children: [
    Icon(Icons.star, color: Colors.yellow),
    Icon(Icons.star, color: Colors.yellow),
    Icon(Icons.star, color: Colors.yellow),
    Icon(Icons.star_border),
    Icon(Icons.star_border),
  ],
)
```

**MainAxisAlignment** (แกนหลัก - แนวนอน):
- `start`: ชิดซ้าย
- `end`: ชิดขวา
- `center`: กึ่งกลาง
- `spaceBetween`: เว้นระหว่าง
- `spaceAround`: เว้นรอบข้าง
- `spaceEvenly`: เว้นเท่าๆ กัน

**CrossAxisAlignment** (แกนรอง - แนวตั้ง):
- `start`: บนสุด
- `end`: ล่างสุด
- `center`: กึ่งกลาง
- `stretch`: ยืดเต็ม

### Column - จัด Widget ในแนวตั้ง

```dart
Column(
  mainAxisAlignment: MainAxisAlignment.center,
  crossAxisAlignment: CrossAxisAlignment.start,
  children: [
    Text('รายการสั่งซื้อ', style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold)),
    SizedBox(height: 10),
    Text('1. แอปเปิ้ล x 5'),
    Text('2. กล้วย x 3'),
    Text('3. ส้ม x 2'),
    SizedBox(height: 20),
    Text('รวม: 150 บาท', style: TextStyle(fontWeight: FontWeight.bold)),
  ],
)
```

### Stack - ซ้อน Widget ทับกัน

```dart
Stack(
  alignment: Alignment.center,
  children: [
    // Background
    Container(
      width: 200,
      height: 200,
      color: Colors.blue,
    ),
    // Foreground
    Container(
      width: 100,
      height: 100,
      color: Colors.red,
    ),
    // Text on top
    Text(
      'ซ้อนกัน',
      style: TextStyle(color: Colors.white, fontSize: 24),
    ),
  ],
)
```

**Positioned** - กำหนดตำแหน่งใน Stack:
```dart
Stack(
  children: [
    Container(width: 300, height: 300, color: Colors.grey[300]),
    Positioned(
      top: 20,
      left: 20,
      child: Text('มุมบนซ้าย'),
    ),
    Positioned(
      bottom: 20,
      right: 20,
      child: Text('มุมล่างขวา'),
    ),
    Positioned(
      top: 0,
      right: 0,
      child: Icon(Icons.close),
    ),
  ],
)
```

### Expanded และ Flexible

```dart
// Expanded - ขยายเต็มพื้นที่ว่าง
Row(
  children: [
    Container(width: 50, height: 50, color: Colors.red),
    Expanded(
      child: Container(height: 50, color: Colors.blue),  // ขยายเต็มที่
    ),
    Container(width: 50, height: 50, color: Colors.green),
  ],
)

// Flexible - ยืดหยุ่นตามเนื้อหา
Row(
  children: [
    Flexible(
      flex: 2,  // สัดส่วน 2:1
      child: Container(height: 50, color: Colors.blue),
    ),
    Flexible(
      flex: 1,
      child: Container(height: 50, color: Colors.red),
    ),
  ],
)
```

## 3.3 Container และ Padding

### Container

Container เป็น widget ที่ใช้บ่อยที่สุด ใช้สำหรับจัดการ decoration, padding, margin

```dart
Container(
  width: 200,
  height: 100,
  padding: EdgeInsets.all(16),
  margin: EdgeInsets.symmetric(horizontal: 20, vertical: 10),
  decoration: BoxDecoration(
    color: Colors.blue,
    borderRadius: BorderRadius.circular(12),
    boxShadow: [
      BoxShadow(
        color: Colors.black26,
        blurRadius: 10,
        offset: Offset(0, 5),
      ),
    ],
    gradient: LinearGradient(
      colors: [Colors.blue, Colors.purple],
    ),
  ),
  child: Text(
    'สวยงาม',
    style: TextStyle(color: Colors.white, fontSize: 20),
  ),
)
```

### Padding

```dart
// Padding ทุกด้านเท่ากัน
Padding(
  padding: EdgeInsets.all(16),
  child: Text('มี padding รอบข้าง'),
)

// Padding แต่ละด้าน
Padding(
  padding: EdgeInsets.only(
    left: 20,
    right: 20,
    top: 10,
    bottom: 10,
  ),
  child: Text('Padding กำหนดเอง'),
)

// Padding แนวนอนและแนวตั้ง
Padding(
  padding: EdgeInsets.symmetric(
    horizontal: 24,
    vertical: 12,
  ),
  child: Text('Symmetric padding'),
)
```

### BoxDecoration

```dart
Container(
  decoration: BoxDecoration(
    // สี
    color: Colors.white,
    
    // Border
    border: Border.all(color: Colors.blue, width: 2),
    
    // Border radius
    borderRadius: BorderRadius.circular(15),
    
    // Shadow
    boxShadow: [
      BoxShadow(
        color: Colors.grey.withOpacity(0.5),
        spreadRadius: 2,
        blurRadius: 5,
        offset: Offset(0, 3),
      ),
    ],
    
    // Gradient
    gradient: LinearGradient(
      begin: Alignment.topLeft,
      end: Alignment.bottomRight,
      colors: [Colors.blue, Colors.purple, Colors.pink],
    ),
    
    // Image background
    image: DecorationImage(
      image: NetworkImage('https://example.com/image.jpg'),
      fit: BoxFit.cover,
    ),
  ),
)
```

## 3.4 Text และ Image Widgets

### Text Widget

```dart
// Text พื้นฐาน
Text('สวัสดี')

// Text with style
Text(
  'ข้อความที่มีสไตล์',
  style: TextStyle(
    fontSize: 24,
    fontWeight: FontWeight.bold,
    fontStyle: FontStyle.italic,
    color: Colors.blue,
    letterSpacing: 2.0,
    wordSpacing: 5.0,
    decoration: TextDecoration.underline,
    decorationColor: Colors.red,
    decorationStyle: TextDecorationStyle.wavy,
  ),
)

// Text alignment
Text(
  'ข้อความยาวที่จัดชิดกลาง และมีหลายบรรทัด',
  textAlign: TextAlign.center,
  maxLines: 2,
  overflow: TextOverflow.ellipsis,  // ตัด... ถ้ายาวเกิน
)

// Rich Text
RichText(
  text: TextSpan(
    style: TextStyle(color: Colors.black, fontSize: 16),
    children: [
      TextSpan(text: 'ข้อความ '),
      TextSpan(
        text: 'สีแดง ',
        style: TextStyle(color: Colors.red, fontWeight: FontWeight.bold),
      ),
      TextSpan(text: 'และ '),
      TextSpan(
        text: 'สีน้ำเงิน',
        style: TextStyle(color: Colors.blue, fontStyle: FontStyle.italic),
      ),
    ],
  ),
)
```

### Image Widget

```dart
// รูปจาก assets
Image.asset(
  'assets/images/logo.png',
  width: 200,
  height: 200,
  fit: BoxFit.cover,
)

// รูปจาก network
Image.network(
  'https://picsum.photos/200',
  loadingBuilder: (context, child, loadingProgress) {
    if (loadingProgress == null) return child;
    return CircularProgressIndicator();
  },
  errorBuilder: (context, error, stackTrace) {
    return Icon(Icons.error);
  },
)

// รูปจาก file
Image.file(
  File('/path/to/image.jpg'),
)

// BoxFit options
Image.asset(
  'assets/image.png',
  fit: BoxFit.cover,    // เต็มพื้นที่ อาจถูกครอบ
  // fit: BoxFit.contain,  // พอดีในพื้นที่ แสดงทั้งหมด
  // fit: BoxFit.fill,     // ยืดเต็มพื้นที่
  // fit: BoxFit.fitWidth, // พอดีความกว้าง
  // fit: BoxFit.fitHeight,// พอดีความสูง
)

// Circle Avatar
CircleAvatar(
  radius: 50,
  backgroundImage: NetworkImage('https://picsum.photos/200'),
  child: Text('AB'),  // แสดงถ้าไม่มีรูป
)
```

## 3.5 Buttons และ Input Widgets

### Buttons

```dart
// ElevatedButton - ปุ่มนูน
ElevatedButton(
  onPressed: () {
    print('กดแล้ว!');
  },
  style: ElevatedButton.styleFrom(
    backgroundColor: Colors.blue,
    foregroundColor: Colors.white,
    padding: EdgeInsets.symmetric(horizontal: 32, vertical: 16),
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(8),
    ),
  ),
  child: Text('กดฉัน'),
)

// TextButton - ปุ่มแบบข้อความ
TextButton(
  onPressed: () {},
  child: Text('Text Button'),
)

// OutlinedButton - ปุ่มมีกรอบ
OutlinedButton(
  onPressed: () {},
  child: Text('Outlined Button'),
)

// IconButton - ปุ่มไอคอน
IconButton(
  icon: Icon(Icons.favorite),
  color: Colors.red,
  iconSize: 32,
  onPressed: () {},
)

// FloatingActionButton
FloatingActionButton(
  onPressed: () {},
  backgroundColor: Colors.blue,
  child: Icon(Icons.add),
)

// Button with icon
ElevatedButton.icon(
  onPressed: () {},
  icon: Icon(Icons.send),
  label: Text('ส่ง'),
)
```

### TextField - ช่องกรอกข้อความ

```dart
class TextFieldExample extends StatefulWidget {
  @override
  State<TextFieldExample> createState() => _TextFieldExampleState();
}

class _TextFieldExampleState extends State<TextFieldExample> {
  final TextEditingController _controller = TextEditingController();
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // TextField พื้นฐาน
        TextField(
          controller: _controller,
          decoration: InputDecoration(
            labelText: 'ชื่อ',
            hintText: 'กรอกชื่อของคุณ',
            prefixIcon: Icon(Icons.person),
            suffixIcon: Icon(Icons.check),
            border: OutlineInputBorder(),
          ),
          onChanged: (value) {
            print('กรอก: $value');
          },
        ),
        
        SizedBox(height: 20),
        
        // TextField สำหรับรหัสผ่าน
        TextField(
          obscureText: true,
          decoration: InputDecoration(
            labelText: 'รหัสผ่าน',
            prefixIcon: Icon(Icons.lock),
            border: OutlineInputBorder(),
          ),
        ),
        
        SizedBox(height: 20),
        
        // TextField หลายบรรทัด
        TextField(
          maxLines: 4,
          decoration: InputDecoration(
            labelText: 'ข้อความ',
            hintText: 'เขียนข้อความของคุณ...',
            border: OutlineInputBorder(),
          ),
        ),
      ],
    );
  }
}
```

### Checkbox

```dart
class CheckboxExample extends StatefulWidget {
  @override
  State<CheckboxExample> createState() => _CheckboxExampleState();
}

class _CheckboxExampleState extends State<CheckboxExample> {
  bool _isChecked = false;
  
  @override
  Widget build(BuildContext context) {
    return CheckboxListTile(
      title: Text('ยอมรับข้อตกลง'),
      value: _isChecked,
      onChanged: (bool? value) {
        setState(() {
          _isChecked = value ?? false;
        });
      },
      secondary: Icon(Icons.check_circle),
    );
  }
}
```

### Radio Button

```dart
class RadioExample extends StatefulWidget {
  @override
  State<RadioExample> createState() => _RadioExampleState();
}

class _RadioExampleState extends State<RadioExample> {
  String _selectedValue = 'option1';
  
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        RadioListTile(
          title: Text('ตัวเลือก 1'),
          value: 'option1',
          groupValue: _selectedValue,
          onChanged: (value) {
            setState(() {
              _selectedValue = value.toString();
            });
          },
        ),
        RadioListTile(
          title: Text('ตัวเลือก 2'),
          value: 'option2',
          groupValue: _selectedValue,
          onChanged: (value) {
            setState(() {
              _selectedValue = value.toString();
            });
          },
        ),
      ],
    );
  }
}
```

### Switch

```dart
class SwitchExample extends StatefulWidget {
  @override
  State<SwitchExample> createState() => _SwitchExampleState();
}

class _SwitchExampleState extends State<SwitchExample> {
  bool _isSwitched = false;
  
  @override
  Widget build(BuildContext context) {
    return SwitchListTile(
      title: Text('การแจ้งเตือน'),
      subtitle: Text(_isSwitched ? 'เปิด' : 'ปิด'),
      value: _isSwitched,
      onChanged: (bool value) {
        setState(() {
          _isSwitched = value;
        });
      },
      secondary: Icon(Icons.notifications),
    );
  }
}
```

### Slider

```dart
class SliderExample extends StatefulWidget {
  @override
  State<SliderExample> createState() => _SliderExampleState();
}

class _SliderExampleState extends State<SliderExample> {
  double _value = 50;
  
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('ค่า: ${_value.round()}'),
        Slider(
          value: _value,
          min: 0,
          max: 100,
          divisions: 10,
          label: _value.round().toString(),
          onChanged: (double value) {
            setState(() {
              _value = value;
            });
          },
        ),
      ],
    );
  }
}
```

## ตัวอย่างแอปจริง: Form Login

```dart
import 'package:flutter/material.dart';

class LoginPage extends StatefulWidget {
  const LoginPage({super.key});
  
  @override
  State<LoginPage> createState() => _LoginPageState();
}

class _LoginPageState extends State<LoginPage> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  bool _rememberMe = false;
  
  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }
  
  void _login() {
    if (_formKey.currentState!.validate()) {
      // ทำการ login
      print('Email: ${_emailController.text}');
      print('Password: ${_passwordController.text}');
      print('Remember: $_rememberMe');
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: Center(
          child: SingleChildScrollView(
            padding: const EdgeInsets.all(24),
            child: Form(
              key: _formKey,
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                crossAxisAlignment: CrossAxisAlignment.stretch,
                children: [
                  // Logo
                  const Icon(
                    Icons.lock_outline,
                    size: 80,
                    color: Colors.blue,
                  ),
                  const SizedBox(height: 20),
                  
                  // Title
                  const Text(
                    'เข้าสู่ระบบ',
                    style: TextStyle(
                      fontSize: 32,
                      fontWeight: FontWeight.bold,
                    ),
                    textAlign: TextAlign.center,
                  ),
                  const SizedBox(height: 40),
                  
                  // Email Field
                  TextFormField(
                    controller: _emailController,
                    keyboardType: TextInputType.emailAddress,
                    decoration: const InputDecoration(
                      labelText: 'อีเมล',
                      prefixIcon: Icon(Icons.email),
                      border: OutlineInputBorder(),
                    ),
                    validator: (value) {
                      if (value == null || value.isEmpty) {
                        return 'กรุณากรอกอีเมล';
                      }
                      if (!value.contains('@')) {
                        return 'อีเมลไม่ถูกต้อง';
                      }
                      return null;
                    },
                  ),
                  const SizedBox(height: 20),
                  
                  // Password Field
                  TextFormField(
                    controller: _passwordController,
                    obscureText: true,
                    decoration: const InputDecoration(
                      labelText: 'รหัสผ่าน',
                      prefixIcon: Icon(Icons.lock),
                      border: OutlineInputBorder(),
                    ),
                    validator: (value) {
                      if (value == null || value.isEmpty) {
                        return 'กรุณากรอกรหัสผ่าน';
                      }
                      if (value.length < 6) {
                        return 'รหัสผ่านต้องมีอย่างน้อย 6 ตัวอักษร';
                      }
                      return null;
                    },
                  ),
                  const SizedBox(height: 10),
                  
                  // Remember Me
                  CheckboxListTile(
                    title: const Text('จดจำฉันไว้'),
                    value: _rememberMe,
                    onChanged: (value) {
                      setState(() {
                        _rememberMe = value ?? false;
                      });
                    },
                    controlAffinity: ListTileControlAffinity.leading,
                    contentPadding: EdgeInsets.zero,
                  ),
                  const SizedBox(height: 20),
                  
                  // Login Button
                  ElevatedButton(
                    onPressed: _login,
                    style: ElevatedButton.styleFrom(
                      padding: const EdgeInsets.symmetric(vertical: 16),
                      shape: RoundedRectangleBorder(
                        borderRadius: BorderRadius.circular(8),
                      ),
                    ),
                    child: const Text(
                      'เข้าสู่ระบบ',
                      style: TextStyle(fontSize: 18),
                    ),
                  ),
                  const SizedBox(height: 20),
                  
                  // Forgot Password
                  TextButton(
                    onPressed: () {},
                    child: const Text('ลืมรหัสผ่าน?'),
                  ),
                  
                  // Register
                  Row(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      const Text('ยังไม่มีบัญชี? '),
                      TextButton(
                        onPressed: () {},
                        child: const Text('สมัครสมาชิก'),
                      ),
                    ],
                  ),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

## แบบฝึกหัด

1. สร้าง Profile Card ที่แสดงรูปโปรไฟล์, ชื่อ, และรายละเอียด
2. สร้างฟอร์มลงทะเบียนที่มี TextField สำหรับ: ชื่อ, อีเมล, รหัสผ่าน, ยืนยันรหัสผ่าน
3. สร้าง Counter App ที่มีปุ่ม +, -, รีเซ็ต
4. สร้าง To-Do List โดยใช้ Checkbox
5. สร้างหน้าตั้งค่าที่มี Switch และ Slider

## สรุป

ในบทนี้เราได้เรียนรู้:
- ความแตกต่างระหว่าง StatelessWidget และ StatefulWidget
- Layout Widgets: Row, Column, Stack
- Container, Padding และ BoxDecoration
- Text และ Image Widgets
- Buttons และ Input Widgets ต่างๆ
- สร้างฟอร์ม Login ที่สมบูรณ์

---

**หัวข้อก่อนหน้า**: [บทที่ 2: พื้นฐาน Dart Programming](../chapter-02-dart-basics/README.md)  
**หัวข้อถัดไป**: [บทที่ 4: State Management](../chapter-04-state-management/README.md)
