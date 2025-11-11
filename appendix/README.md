# ภาคผนวก

## A. Flutter CLI Commands

### Project Management

```bash
# สร้างโปรเจคใหม่
flutter create my_app
flutter create --org com.example my_app

# รันแอป
flutter run
flutter run -d chrome                    # รันบน web
flutter run --release                   # รันแบบ release mode
flutter run --profile                   # รันแบบ profile mode

# ตรวจสอบการติดตั้ง
flutter doctor
flutter doctor -v                       # แสดงรายละเอียดเพิ่ม

# ดู devices ที่เชื่อมต่อ
flutter devices

# Clean project
flutter clean
```

### Package Management

```bash
# ติดตั้ง packages
flutter pub get
flutter pub add package_name           # เพิ่ม package
flutter pub remove package_name        # ลบ package
flutter pub upgrade                    # อัพเดท packages

# ดู outdated packages
flutter pub outdated
```

### Build

```bash
# Android
flutter build apk                      # Build APK
flutter build appbundle               # Build App Bundle (สำหรับ Play Store)
flutter build apk --split-per-abi     # Build APK แยกตาม architecture

# iOS
flutter build ios                      # Build iOS
flutter build ipa                      # Build IPA

# Web
flutter build web                      # Build Web
flutter build web --release           # Build Web แบบ release

# Desktop
flutter build windows
flutter build macos
flutter build linux
```

### Testing

```bash
# รัน tests
flutter test
flutter test test/widget_test.dart    # รัน test เดียว
flutter test --coverage               # Generate coverage report

# Integration tests
flutter drive --target=test_driver/app.dart
```

### Code Generation

```bash
# Generate code (for packages like json_serializable)
flutter pub run build_runner build
flutter pub run build_runner watch     # Watch mode
flutter pub run build_runner build --delete-conflicting-outputs
```

### Other Commands

```bash
# Format code
flutter format lib/
dart format .

# Analyze code
flutter analyze

# Upgrade Flutter
flutter upgrade
flutter upgrade --force

# Channel management
flutter channel                        # ดู channels ทั้งหมด
flutter channel stable                # เปลี่ยนเป็น stable channel
flutter channel beta                  # เปลี่ยนเป็น beta channel
```

## B. Useful Packages

### UI & Widgets

| Package | Description | Usage |
|---------|-------------|-------|
| `google_fonts` | Google Fonts | Custom fonts |
| `flutter_svg` | SVG support | SVG images |
| `cached_network_image` | Image caching | Network images |
| `shimmer` | Shimmer effect | Loading placeholders |
| `flutter_slidable` | Slidable list tiles | Swipe actions |
| `badges` | Badge widgets | Notification badges |

### State Management

| Package | Description | Usage |
|---------|-------------|-------|
| `provider` | Provider pattern | Recommended by Flutter team |
| `flutter_riverpod` | Riverpod | Modern state management |
| `bloc` | BLoC pattern | Complex apps |
| `get` | GetX | All-in-one solution |
| `mobx` | MobX | Reactive state management |

### Navigation

| Package | Description | Usage |
|---------|-------------|-------|
| `go_router` | Declarative routing | Modern navigation |
| `auto_route` | Code generation routing | Type-safe routing |

### Storage

| Package | Description | Usage |
|---------|-------------|-------|
| `shared_preferences` | Key-value storage | Simple data |
| `hive` | NoSQL database | Fast local storage |
| `sqflite` | SQLite | Relational database |
| `path_provider` | File paths | File storage |
| `isar` | Super fast database | High performance |

### Network

| Package | Description | Usage |
|---------|-------------|-------|
| `http` | HTTP requests | Basic API calls |
| `dio` | Advanced HTTP | Interceptors, retry |
| `retrofit` | Type-safe API | Code generation |
| `graphql_flutter` | GraphQL | GraphQL APIs |

### Firebase

| Package | Description | Usage |
|---------|-------------|-------|
| `firebase_core` | Firebase core | Required for Firebase |
| `firebase_auth` | Authentication | User auth |
| `cloud_firestore` | Firestore | Cloud database |
| `firebase_storage` | Cloud storage | File storage |
| `firebase_messaging` | Push notifications | FCM |

### Utilities

| Package | Description | Usage |
|---------|-------------|-------|
| `intl` | Internationalization | Date, number formatting |
| `url_launcher` | Launch URLs | Open links, call, email |
| `share_plus` | Share content | Share to other apps |
| `image_picker` | Pick images | Camera, gallery |
| `permission_handler` | Permissions | Runtime permissions |
| `connectivity_plus` | Network connectivity | Check internet |
| `flutter_local_notifications` | Local notifications | Show notifications |

### Forms & Validation

| Package | Description | Usage |
|---------|-------------|-------|
| `flutter_form_builder` | Form builder | Complex forms |
| `form_validator` | Validation | Input validation |

### Animation

| Package | Description | Usage |
|---------|-------------|-------|
| `animations` | Material animations | Transitions |
| `flutter_animate` | Easy animations | Declarative animations |
| `lottie` | Lottie animations | JSON animations |

### Charts & Graphs

| Package | Description | Usage |
|---------|-------------|-------|
| `fl_chart` | Beautiful charts | Line, bar, pie charts |
| `syncfusion_flutter_charts` | Professional charts | Advanced charts |

## C. Resources และแหล่งเรียนรู้เพิ่มเติม

### Official Resources

- **Flutter Website**: https://flutter.dev
- **Flutter Documentation**: https://docs.flutter.dev
- **API Reference**: https://api.flutter.dev
- **Flutter YouTube**: https://www.youtube.com/@flutterdev
- **Dart Documentation**: https://dart.dev/guides

### Learning Platforms

- **Flutter Codelabs**: https://docs.flutter.dev/codelabs
- **DartPad**: https://dartpad.dev (Online Dart/Flutter editor)
- **Flutter Gallery**: https://gallery.flutter.dev
- **Widget Catalog**: https://docs.flutter.dev/ui/widgets

### Community

- **Stack Overflow**: Tag `flutter`
- **Reddit**: r/FlutterDev
- **Discord**: Flutter Community
- **Twitter**: #FlutterDev

### Thai Resources

- **Flutter Thailand**: Facebook Group
- **DevHub Thailand**: Discord Community
- **Medium**: Thai Flutter articles

### Courses & Tutorials

- **Flutter & Dart - The Complete Guide** (Udemy)
- **The Complete Flutter Development Bootcamp** (Udemy)
- **Flutter Tutorial for Beginners** (YouTube - The Net Ninja)
- **Reso Coder** (YouTube - Clean Architecture)

### Blogs & Articles

- **Flutter Medium**: https://medium.com/flutter
- **Flutter Community**: https://medium.com/flutter-community
- **Very Good Ventures Blog**: https://verygood.ventures/blog

### Tools

- **Flutter DevTools**: Performance, debugging
- **Zapp.run**: Online Flutter IDE
- **FlutLab**: Online Flutter IDE
- **Codemagic**: CI/CD for Flutter
- **Fastlane**: Automate deployments

## D. Best Practices

### Code Organization

```
lib/
├── core/
│   ├── constants/
│   ├── theme/
│   ├── utils/
│   └── errors/
├── features/
│   └── feature_name/
│       ├── data/
│       │   ├── models/
│       │   ├── repositories/
│       │   └── data_sources/
│       ├── domain/
│       │   ├── entities/
│       │   ├── repositories/
│       │   └── use_cases/
│       └── presentation/
│           ├── screens/
│           ├── widgets/
│           └── providers/
└── main.dart
```

### Naming Conventions

- **Files**: `snake_case.dart`
- **Classes**: `PascalCase`
- **Variables**: `camelCase`
- **Constants**: `SCREAMING_SNAKE_CASE` or `kCamelCase`
- **Private**: `_privateVariable`

### Widget Best Practices

```dart
// ดี: แยก widget ย่อย
class MyPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          _Header(),
          _Content(),
          _Footer(),
        ],
      ),
    );
  }
}

// ไม่ดี: widget ใหญ่เกินไป
class MyPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          // 100+ lines of code...
        ],
      ),
    );
  }
}

// ดี: ใช้ const เมื่อเป็นไปได้
const Text('Hello')
const SizedBox(height: 16)
const Icon(Icons.home)

// ดี: ใช้ key เมื่อจำเป็น
ListView.builder(
  itemBuilder: (context, index) {
    return Card(
      key: ValueKey(items[index].id),
      child: Text(items[index].name),
    );
  },
)
```

### State Management

```dart
// ดี: แยก business logic จาก UI
class UserProvider extends ChangeNotifier {
  User? _user;
  
  User? get user => _user;
  
  Future<void> loadUser() async {
    _user = await repository.getUser();
    notifyListeners();
  }
}

// ดี: ใช้ Consumer เฉพาะที่จำเป็น
Consumer<UserProvider>(
  builder: (context, provider, child) {
    return Text(provider.user?.name ?? '');
  },
)

// ไม่ดี: rebuild ทั้งหน้า
class MyPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final provider = Provider.of<UserProvider>(context);
    // ทั้งหน้า rebuild ทุกครั้ง
  }
}
```

### Error Handling

```dart
// ดี: จัดการ error ที่เฉพาะเจาะจง
try {
  final data = await api.getData();
} on NetworkException {
  showError('ไม่มีอินเทอร์เน็ต');
} on ServerException {
  showError('เซิร์ฟเวอร์ขัดข้อง');
} catch (e) {
  showError('เกิดข้อผิดพลาด');
}

// ดี: ใช้ FutureBuilder/StreamBuilder
FutureBuilder<User>(
  future: getUser(),
  builder: (context, snapshot) {
    if (snapshot.hasError) {
      return ErrorWidget(snapshot.error);
    }
    if (snapshot.hasData) {
      return UserWidget(snapshot.data!);
    }
    return LoadingWidget();
  },
)
```

### Performance

```dart
// ดี: ใช้ const constructors
const Text('Hello')
const Padding(padding: EdgeInsets.all(8))

// ดี: ใช้ ListView.builder สำหรับ list ยาว
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return ItemWidget(items[index]);
  },
)

// ไม่ดี: สร้าง widget ทั้งหมดทันที
ListView(
  children: items.map((item) => ItemWidget(item)).toList(),
)

// ดี: dispose controllers
@override
void dispose() {
  _controller.dispose();
  super.dispose();
}
```

### Testing

```dart
// Widget test
testWidgets('Counter increments', (tester) async {
  await tester.pumpWidget(MyApp());
  
  expect(find.text('0'), findsOneWidget);
  
  await tester.tap(find.byIcon(Icons.add));
  await tester.pump();
  
  expect(find.text('1'), findsOneWidget);
});

// Unit test
test('Calculate BMI', () {
  final bmi = calculateBMI(70, 175);
  expect(bmi, closeTo(22.86, 0.01));
});
```

### Security

```dart
// ดี: ไม่เก็บ sensitive data ใน code
final apiKey = dotenv.env['API_KEY'];

// ดี: เข้ารหัสข้อมูลสำคัญ
await secureStorage.write(key: 'token', value: token);

// ดี: validate input
if (email.contains('@') && email.contains('.')) {
  // Valid email
}

// ดี: ใช้ HTTPS
final url = Uri.parse('https://api.example.com/data');
```

### Accessibility

```dart
// ดี: ให้ semantic labels
Semantics(
  label: 'ปุ่มส่ง',
  child: IconButton(
    icon: Icon(Icons.send),
    onPressed: () {},
  ),
)

// ดี: ขนาดที่สัมผัสได้
// ปุ่มควรมีขนาดอย่างน้อย 48x48 pixels
SizedBox(
  width: 48,
  height: 48,
  child: IconButton(icon: Icon(Icons.add)),
)
```

## สรุป

ภาคผนวกนี้รวบรวม:
- Flutter CLI commands ที่ใช้บ่อย
- Packages ที่มีประโยชน์
- แหล่งเรียนรู้และ resources
- Best practices สำหรับการพัฒนา Flutter

---

**กลับสู่**: [สารบัญ](../TABLE_OF_CONTENTS.md)
