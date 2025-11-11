# บทที่ 5: Navigation และ Routing

การนำทางระหว่างหน้าต่างๆ ในแอป Flutter

## 5.1 Navigator.push และ Navigator.pop

วิธีการนำทางพื้นฐานใน Flutter ทำงานแบบ Stack (LIFO)

### ตัวอย่างพื้นฐาน

```dart
import 'package:flutter/material.dart';

// หน้าแรก
class HomePage extends StatelessWidget {
  const HomePage({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('หน้าแรก')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // ไปหน้าใหม่
            Navigator.push(
              context,
              MaterialPageRoute(
                builder: (context) => const SecondPage(),
              ),
            );
          },
          child: const Text('ไปหน้าถัดไป'),
        ),
      ),
    );
  }
}

// หน้าที่สอง
class SecondPage extends StatelessWidget {
  const SecondPage({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('หน้าที่สอง')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // กลับหน้าเดิม
            Navigator.pop(context);
          },
          child: const Text('กลับ'),
        ),
      ),
    );
  }
}
```

### Custom Transition

```dart
// Slide transition
Navigator.push(
  context,
  PageRouteBuilder(
    pageBuilder: (context, animation, secondaryAnimation) => SecondPage(),
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      const begin = Offset(1.0, 0.0);
      const end = Offset.zero;
      const curve = Curves.easeInOut;
      
      var tween = Tween(begin: begin, end: end).chain(
        CurveTween(curve: curve),
      );
      
      return SlideTransition(
        position: animation.drive(tween),
        child: child,
      );
    },
  ),
);

// Fade transition
Navigator.push(
  context,
  PageRouteBuilder(
    pageBuilder: (context, animation, secondaryAnimation) => SecondPage(),
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      return FadeTransition(
        opacity: animation,
        child: child,
      );
    },
  ),
);
```

## 5.2 Named Routes

ใช้ชื่อแทน route สะดวกในการจัดการ navigation

### กำหนด Named Routes

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
      title: 'Navigation Demo',
      initialRoute: '/',
      routes: {
        '/': (context) => const HomePage(),
        '/second': (context) => const SecondPage(),
        '/third': (context) => const ThirdPage(),
        '/profile': (context) => const ProfilePage(),
      },
      // สำหรับ route ที่ไม่พบ
      onUnknownRoute: (settings) {
        return MaterialPageRoute(
          builder: (context) => const NotFoundPage(),
        );
      },
    );
  }
}

// การใช้งาน
class HomePage extends StatelessWidget {
  const HomePage({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('หน้าแรก')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () {
                Navigator.pushNamed(context, '/second');
              },
              child: const Text('ไปหน้าที่สอง'),
            ),
            ElevatedButton(
              onPressed: () {
                Navigator.pushNamed(context, '/profile');
              },
              child: const Text('ไปหน้าโปรไฟล์'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Generate Routes

```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});
  
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'App',
      onGenerateRoute: (settings) {
        // จัดการ route แบบ dynamic
        if (settings.name == '/') {
          return MaterialPageRoute(builder: (_) => const HomePage());
        }
        
        // Route ที่มี parameter
        if (settings.name == '/details') {
          final args = settings.arguments as Map<String, dynamic>;
          return MaterialPageRoute(
            builder: (_) => DetailsPage(
              id: args['id'],
              title: args['title'],
            ),
          );
        }
        
        // Route ไม่พบ
        return MaterialPageRoute(builder: (_) => const NotFoundPage());
      },
    );
  }
}
```

## 5.3 ส่งข้อมูลระหว่างหน้า

### ส่งข้อมูลไปหน้าใหม่

```dart
// User model
class User {
  final String name;
  final int age;
  final String email;
  
  User({required this.name, required this.age, required this.email});
}

// ส่งข้อมูลด้วย Constructor
class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('หน้าแรก')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            final user = User(
              name: 'สมชาย',
              age: 25,
              email: 'somchai@example.com',
            );
            
            Navigator.push(
              context,
              MaterialPageRoute(
                builder: (context) => ProfilePage(user: user),
              ),
            );
          },
          child: const Text('ดูโปรไฟล์'),
        ),
      ),
    );
  }
}

class ProfilePage extends StatelessWidget {
  final User user;
  
  const ProfilePage({super.key, required this.user});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('โปรไฟล์')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text('ชื่อ: ${user.name}', style: const TextStyle(fontSize: 20)),
            Text('อายุ: ${user.age} ปี', style: const TextStyle(fontSize: 20)),
            Text('อีเมล: ${user.email}', style: const TextStyle(fontSize: 20)),
          ],
        ),
      ),
    );
  }
}

// ส่งข้อมูลด้วย Named Route Arguments
Navigator.pushNamed(
  context,
  '/profile',
  arguments: user,
);

// รับข้อมูลใน ProfilePage
class ProfilePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final user = ModalRoute.of(context)!.settings.arguments as User;
    
    return Scaffold(
      appBar: AppBar(title: const Text('โปรไฟล์')),
      body: Text('ชื่อ: ${user.name}'),
    );
  }
}
```

### รับข้อมูลกลับจากหน้าที่เปิด

```dart
// หน้าแรก - เปิดหน้าแล้วรอผลลัพธ์
class HomePage extends StatefulWidget {
  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  String _result = 'ยังไม่มีผลลัพธ์';
  
  Future<void> _openSelectionPage() async {
    final result = await Navigator.push<String>(
      context,
      MaterialPageRoute(
        builder: (context) => const SelectionPage(),
      ),
    );
    
    if (result != null) {
      setState(() {
        _result = result;
      });
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('หน้าแรก')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('ผลลัพธ์: $_result'),
            const SizedBox(height: 20),
            ElevatedButton(
              onPressed: _openSelectionPage,
              child: const Text('เลือกตัวเลือก'),
            ),
          ],
        ),
      ),
    );
  }
}

// หน้าที่สอง - ส่งผลลัพธ์กลับ
class SelectionPage extends StatelessWidget {
  const SelectionPage({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('เลือกตัวเลือก')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () {
                Navigator.pop(context, 'ตัวเลือก A');
              },
              child: const Text('ตัวเลือก A'),
            ),
            ElevatedButton(
              onPressed: () {
                Navigator.pop(context, 'ตัวเลือก B');
              },
              child: const Text('ตัวเลือก B'),
            ),
            ElevatedButton(
              onPressed: () {
                Navigator.pop(context, 'ตัวเลือก C');
              },
              child: const Text('ตัวเลือก C'),
            ),
          ],
        ),
      ),
    );
  }
}
```

## 5.4 Bottom Navigation Bar

แถบเมนูด้านล่างสำหรับสลับระหว่างหน้าหลักๆ

```dart
import 'package:flutter/material.dart';

class MainPage extends StatefulWidget {
  const MainPage({super.key});
  
  @override
  State<MainPage> createState() => _MainPageState();
}

class _MainPageState extends State<MainPage> {
  int _currentIndex = 0;
  
  // รายการหน้าต่างๆ
  final List<Widget> _pages = [
    const HomePage(),
    const SearchPage(),
    const FavoritePage(),
    const ProfilePage(),
  ];
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: _pages[_currentIndex],
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _currentIndex,
        onTap: (index) {
          setState(() {
            _currentIndex = index;
          });
        },
        type: BottomNavigationBarType.fixed,
        selectedItemColor: Colors.blue,
        unselectedItemColor: Colors.grey,
        items: const [
          BottomNavigationBarItem(
            icon: Icon(Icons.home),
            label: 'หน้าแรก',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.search),
            label: 'ค้นหา',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.favorite),
            label: 'ชอบ',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.person),
            label: 'โปรไฟล์',
          ),
        ],
      ),
    );
  }
}

// หน้าต่างๆ
class HomePage extends StatelessWidget {
  const HomePage({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('หน้าแรก')),
      body: const Center(child: Text('หน้าแรก')),
    );
  }
}

class SearchPage extends StatelessWidget {
  const SearchPage({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('ค้นหา')),
      body: const Center(child: Text('ค้นหา')),
    );
  }
}
```

### Navigation Bar (Material 3)

```dart
class MainPage extends StatefulWidget {
  const MainPage({super.key});
  
  @override
  State<MainPage> createState() => _MainPageState();
}

class _MainPageState extends State<MainPage> {
  int _currentIndex = 0;
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: _pages[_currentIndex],
      bottomNavigationBar: NavigationBar(
        selectedIndex: _currentIndex,
        onDestinationSelected: (index) {
          setState(() {
            _currentIndex = index;
          });
        },
        destinations: const [
          NavigationDestination(
            icon: Icon(Icons.home_outlined),
            selectedIcon: Icon(Icons.home),
            label: 'หน้าแรก',
          ),
          NavigationDestination(
            icon: Icon(Icons.search_outlined),
            selectedIcon: Icon(Icons.search),
            label: 'ค้นหา',
          ),
          NavigationDestination(
            icon: Icon(Icons.favorite_outline),
            selectedIcon: Icon(Icons.favorite),
            label: 'ชอบ',
          ),
        ],
      ),
    );
  }
}
```

## 5.5 Drawer Navigation

เมนูแบบเลื่อนจากด้านข้าง

```dart
class HomePage extends StatelessWidget {
  const HomePage({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('หน้าแรก'),
      ),
      drawer: Drawer(
        child: ListView(
          padding: EdgeInsets.zero,
          children: [
            // Header
            const DrawerHeader(
              decoration: BoxDecoration(
                gradient: LinearGradient(
                  colors: [Colors.blue, Colors.purple],
                ),
              ),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                mainAxisAlignment: MainAxisAlignment.end,
                children: [
                  CircleAvatar(
                    radius: 30,
                    backgroundColor: Colors.white,
                    child: Icon(Icons.person, size: 40, color: Colors.blue),
                  ),
                  SizedBox(height: 10),
                  Text(
                    'สมชาย ใจดี',
                    style: TextStyle(
                      color: Colors.white,
                      fontSize: 20,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  Text(
                    'somchai@example.com',
                    style: TextStyle(color: Colors.white70),
                  ),
                ],
              ),
            ),
            
            // Menu items
            ListTile(
              leading: const Icon(Icons.home),
              title: const Text('หน้าแรก'),
              onTap: () {
                Navigator.pop(context);
              },
            ),
            ListTile(
              leading: const Icon(Icons.person),
              title: const Text('โปรไฟล์'),
              onTap: () {
                Navigator.pop(context);
                Navigator.pushNamed(context, '/profile');
              },
            ),
            ListTile(
              leading: const Icon(Icons.settings),
              title: const Text('ตั้งค่า'),
              onTap: () {
                Navigator.pop(context);
                Navigator.pushNamed(context, '/settings');
              },
            ),
            const Divider(),
            ListTile(
              leading: const Icon(Icons.help),
              title: const Text('ช่วยเหลือ'),
              onTap: () {},
            ),
            ListTile(
              leading: const Icon(Icons.info),
              title: const Text('เกี่ยวกับ'),
              onTap: () {},
            ),
            const Divider(),
            ListTile(
              leading: const Icon(Icons.logout, color: Colors.red),
              title: const Text('ออกจากระบบ', 
                style: TextStyle(color: Colors.red)),
              onTap: () {
                // ออกจากระบบ
              },
            ),
          ],
        ),
      ),
      body: const Center(
        child: Text('หน้าหลัก'),
      ),
    );
  }
}
```

## ตัวอย่างแอปจริง: E-commerce Navigation

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
      title: 'Shop App',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        useMaterial3: true,
      ),
      initialRoute: '/',
      routes: {
        '/': (context) => const MainPage(),
        '/product': (context) => const ProductDetailPage(),
        '/cart': (context) => const CartPage(),
        '/profile': (context) => const ProfilePage(),
      },
    );
  }
}

class MainPage extends StatefulWidget {
  const MainPage({super.key});
  
  @override
  State<MainPage> createState() => _MainPageState();
}

class _MainPageState extends State<MainPage> {
  int _currentIndex = 0;
  
  final List<Widget> _pages = [
    const ShopPage(),
    const CategoriesPage(),
    const FavoritesPage(),
    const AccountPage(),
  ];
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: _pages[_currentIndex],
      bottomNavigationBar: NavigationBar(
        selectedIndex: _currentIndex,
        onDestinationSelected: (index) {
          setState(() {
            _currentIndex = index;
          });
        },
        destinations: const [
          NavigationDestination(
            icon: Icon(Icons.shop_outlined),
            selectedIcon: Icon(Icons.shop),
            label: 'Shop',
          ),
          NavigationDestination(
            icon: Icon(Icons.category_outlined),
            selectedIcon: Icon(Icons.category),
            label: 'Categories',
          ),
          NavigationDestination(
            icon: Icon(Icons.favorite_outline),
            selectedIcon: Icon(Icons.favorite),
            label: 'Favorites',
          ),
          NavigationDestination(
            icon: Icon(Icons.person_outline),
            selectedIcon: Icon(Icons.person),
            label: 'Account',
          ),
        ],
      ),
    );
  }
}

class ShopPage extends StatelessWidget {
  const ShopPage({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Shop'),
        actions: [
          IconButton(
            icon: const Icon(Icons.shopping_cart),
            onPressed: () {
              Navigator.pushNamed(context, '/cart');
            },
          ),
        ],
      ),
      body: GridView.builder(
        padding: const EdgeInsets.all(8),
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          childAspectRatio: 0.75,
          crossAxisSpacing: 8,
          mainAxisSpacing: 8,
        ),
        itemCount: 10,
        itemBuilder: (context, index) {
          return ProductCard(
            name: 'สินค้า ${index + 1}',
            price: (index + 1) * 100.0,
            onTap: () {
              Navigator.pushNamed(
                context,
                '/product',
                arguments: {
                  'name': 'สินค้า ${index + 1}',
                  'price': (index + 1) * 100.0,
                },
              );
            },
          );
        },
      ),
    );
  }
}

class ProductCard extends StatelessWidget {
  final String name;
  final double price;
  final VoidCallback onTap;
  
  const ProductCard({
    super.key,
    required this.name,
    required this.price,
    required this.onTap,
  });
  
  @override
  Widget build(BuildContext context) {
    return Card(
      clipBehavior: Clip.antiAlias,
      child: InkWell(
        onTap: onTap,
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Expanded(
              child: Container(
                color: Colors.grey[300],
                child: const Center(
                  child: Icon(Icons.image, size: 60, color: Colors.grey),
                ),
              ),
            ),
            Padding(
              padding: const EdgeInsets.all(8),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    name,
                    style: const TextStyle(
                      fontWeight: FontWeight.bold,
                      fontSize: 16,
                    ),
                  ),
                  const SizedBox(height: 4),
                  Text(
                    '฿${price.toStringAsFixed(2)}',
                    style: const TextStyle(
                      color: Colors.blue,
                      fontSize: 14,
                    ),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class ProductDetailPage extends StatelessWidget {
  const ProductDetailPage({super.key});
  
  @override
  Widget build(BuildContext context) {
    final args = ModalRoute.of(context)!.settings.arguments as Map<String, dynamic>;
    
    return Scaffold(
      appBar: AppBar(
        title: Text(args['name']),
        actions: [
          IconButton(
            icon: const Icon(Icons.shopping_cart),
            onPressed: () {
              Navigator.pushNamed(context, '/cart');
            },
          ),
        ],
      ),
      body: Column(
        crossAxisAlignment: CrossAxisAlignment.stretch,
        children: [
          Expanded(
            child: Container(
              color: Colors.grey[300],
              child: const Center(
                child: Icon(Icons.image, size: 150, color: Colors.grey),
              ),
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(16),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  args['name'],
                  style: const TextStyle(
                    fontSize: 24,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                const SizedBox(height: 8),
                Text(
                  '฿${args['price'].toStringAsFixed(2)}',
                  style: const TextStyle(
                    fontSize: 20,
                    color: Colors.blue,
                  ),
                ),
                const SizedBox(height: 16),
                const Text(
                  'รายละเอียดสินค้า',
                  style: TextStyle(
                    fontSize: 18,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                const SizedBox(height: 8),
                const Text(
                  'นี่คือรายละเอียดของสินค้า Lorem ipsum dolor sit amet...',
                  style: TextStyle(fontSize: 16),
                ),
                const SizedBox(height: 24),
                ElevatedButton(
                  onPressed: () {
                    ScaffoldMessenger.of(context).showSnackBar(
                      SnackBar(
                        content: Text('เพิ่ม ${args['name']} ลงตะกร้า'),
                        action: SnackBarAction(
                          label: 'ดูตะกร้า',
                          onPressed: () {
                            Navigator.pushNamed(context, '/cart');
                          },
                        ),
                      ),
                    );
                  },
                  style: ElevatedButton.styleFrom(
                    padding: const EdgeInsets.symmetric(vertical: 16),
                  ),
                  child: const Text(
                    'เพิ่มลงตะกร้า',
                    style: TextStyle(fontSize: 18),
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

// Placeholder pages
class CategoriesPage extends StatelessWidget {
  const CategoriesPage({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Categories')),
      body: const Center(child: Text('Categories Page')),
    );
  }
}

class FavoritesPage extends StatelessWidget {
  const FavoritesPage({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Favorites')),
      body: const Center(child: Text('Favorites Page')),
    );
  }
}

class AccountPage extends StatelessWidget {
  const AccountPage({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Account')),
      body: const Center(child: Text('Account Page')),
    );
  }
}

class CartPage extends StatelessWidget {
  const CartPage({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Cart')),
      body: const Center(child: Text('Cart Page')),
    );
  }
}

class ProfilePage extends StatelessWidget {
  const ProfilePage({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Profile')),
      body: const Center(child: Text('Profile Page')),
    );
  }
}
```

## แบบฝึกหัด

1. สร้างแอปที่มี 3 หน้า เชื่อมด้วย Navigator.push
2. สร้างแอปด้วย Named Routes
3. สร้างแอปที่ส่งข้อมูล User ระหว่างหน้า
4. สร้างแอปด้วย Bottom Navigation Bar
5. สร้างแอปที่มี Drawer Navigation

## สรุป

ในบทนี้เราได้เรียนรู้:
- Navigator.push และ pop สำหรับนำทางพื้นฐาน
- Named Routes สำหรับจัดการ route แบบมีโครงสร้าง
- การส่งและรับข้อมูลระหว่างหน้า
- Bottom Navigation Bar
- Drawer Navigation
- สร้างแอป E-commerce navigation ที่สมบูรณ์

---

**หัวข้อก่อนหน้า**: [บทที่ 4: State Management](../chapter-04-state-management/README.md)  
**หัวข้อถัดไป**: [บทที่ 6: การทำงานกับ APIs](../chapter-06-apis/README.md)
