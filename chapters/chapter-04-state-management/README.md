# บทที่ 4: State Management

State Management เป็นหัวใจสำคัญของแอปพลิเคชัน Flutter ที่ซับซ้อน ช่วยจัดการข้อมูลและการอัพเดท UI

## 4.1 setState() แบบพื้นฐาน

วิธีการจัดการ state ที่ง่ายที่สุดใน Flutter

### ตัวอย่าง: Counter App

```dart
import 'package:flutter/material.dart';

class CounterApp extends StatefulWidget {
  const CounterApp({super.key});
  
  @override
  State<CounterApp> createState() => _CounterAppState();
}

class _CounterAppState extends State<CounterApp> {
  int _counter = 0;
  
  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Counter')),
      body: Center(
        child: Text(
          '$_counter',
          style: const TextStyle(fontSize: 48),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

### ข้อดีและข้อจำกัด

**ข้อดี:**
- เรียนรู้ง่าย
- เหมาะกับ widget เดี่ยวๆ
- ไม่ต้องติดตั้ง package เพิ่ม

**ข้อจำกัด:**
- ยากในการแชร์ state ระหว่าง widget
- สร้าง tight coupling
- ไม่เหมาะกับแอปขนาดใหญ่

## 4.2 InheritedWidget

Widget พิเศษที่ช่วยส่งข้อมูลลงไปยัง widget ลูกๆ

### ตัวอย่าง: Theme Provider

```dart
import 'package:flutter/material.dart';

// 1. สร้าง InheritedWidget
class ThemeProvider extends InheritedWidget {
  final ThemeData theme;
  final Function(ThemeData) updateTheme;
  
  const ThemeProvider({
    Key? key,
    required this.theme,
    required this.updateTheme,
    required Widget child,
  }) : super(key: key, child: child);
  
  static ThemeProvider? of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<ThemeProvider>();
  }
  
  @override
  bool updateShouldNotify(ThemeProvider oldWidget) {
    return theme != oldWidget.theme;
  }
}

// 2. สร้าง Stateful Widget ที่ wrap ThemeProvider
class MyApp extends StatefulWidget {
  const MyApp({super.key});
  
  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  ThemeData _currentTheme = ThemeData.light();
  
  void _toggleTheme(ThemeData newTheme) {
    setState(() {
      _currentTheme = newTheme;
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return ThemeProvider(
      theme: _currentTheme,
      updateTheme: _toggleTheme,
      child: MaterialApp(
        theme: _currentTheme,
        home: const HomePage(),
      ),
    );
  }
}

// 3. ใช้งานใน widget ลูก
class HomePage extends StatelessWidget {
  const HomePage({super.key});
  
  @override
  Widget build(BuildContext context) {
    final themeProvider = ThemeProvider.of(context);
    final isDark = themeProvider?.theme.brightness == Brightness.dark;
    
    return Scaffold(
      appBar: AppBar(title: const Text('Theme Demo')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            themeProvider?.updateTheme(
              isDark ? ThemeData.light() : ThemeData.dark(),
            );
          },
          child: Text(isDark ? 'Switch to Light' : 'Switch to Dark'),
        ),
      ),
    );
  }
}
```

## 4.3 Provider Pattern

Provider เป็น state management ที่แนะนำโดย Flutter team

### การติดตั้ง

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  provider: ^6.1.0
```

### ตัวอย่าง: Counter with Provider

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

// 1. สร้าง Model class
class Counter with ChangeNotifier {
  int _count = 0;
  
  int get count => _count;
  
  void increment() {
    _count++;
    notifyListeners();  // แจ้งเตือน listeners ว่า state เปลี่ยน
  }
  
  void decrement() {
    _count--;
    notifyListeners();
  }
  
  void reset() {
    _count = 0;
    notifyListeners();
  }
}

// 2. Provide model ที่ root
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (context) => Counter(),
      child: const MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});
  
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: const CounterPage(),
    );
  }
}

// 3. Consume model ใน widget
class CounterPage extends StatelessWidget {
  const CounterPage({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Provider Counter')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text('คุณกดปุ่มไปแล้ว:'),
            // Consumer - rebuild เฉพาะส่วนนี้
            Consumer<Counter>(
              builder: (context, counter, child) {
                return Text(
                  '${counter.count}',
                  style: Theme.of(context).textTheme.headlineLarge,
                );
              },
            ),
            const SizedBox(height: 20),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                FloatingActionButton(
                  onPressed: () {
                    context.read<Counter>().decrement();
                  },
                  child: const Icon(Icons.remove),
                ),
                const SizedBox(width: 10),
                FloatingActionButton(
                  onPressed: () {
                    context.read<Counter>().reset();
                  },
                  child: const Icon(Icons.refresh),
                ),
                const SizedBox(width: 10),
                FloatingActionButton(
                  onPressed: () {
                    context.read<Counter>().increment();
                  },
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

### Multiple Providers

```dart
void main() {
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => Counter()),
        ChangeNotifierProvider(create: (_) => UserModel()),
        ChangeNotifierProvider(create: (_) => CartModel()),
      ],
      child: const MyApp(),
    ),
  );
}
```

### ตัวอย่าง: Shopping Cart

```dart
// Product model
class Product {
  final String id;
  final String name;
  final double price;
  
  Product({required this.id, required this.name, required this.price});
}

// Cart model
class CartModel with ChangeNotifier {
  final List<Product> _items = [];
  
  List<Product> get items => _items;
  
  int get itemCount => _items.length;
  
  double get totalPrice {
    return _items.fold(0, (sum, item) => sum + item.price);
  }
  
  void addItem(Product product) {
    _items.add(product);
    notifyListeners();
  }
  
  void removeItem(String productId) {
    _items.removeWhere((item) => item.id == productId);
    notifyListeners();
  }
  
  void clear() {
    _items.clear();
    notifyListeners();
  }
}

// Product List Page
class ProductListPage extends StatelessWidget {
  final List<Product> products = [
    Product(id: '1', name: 'แอปเปิ้ล', price: 50),
    Product(id: '2', name: 'กล้วย', price: 30),
    Product(id: '3', name: 'ส้ม', price: 40),
  ];
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('สินค้า'),
        actions: [
          Stack(
            children: [
              IconButton(
                icon: const Icon(Icons.shopping_cart),
                onPressed: () {
                  Navigator.push(
                    context,
                    MaterialPageRoute(builder: (_) => const CartPage()),
                  );
                },
              ),
              Positioned(
                right: 8,
                top: 8,
                child: Consumer<CartModel>(
                  builder: (context, cart, child) {
                    return cart.itemCount > 0
                        ? Container(
                            padding: const EdgeInsets.all(2),
                            decoration: BoxDecoration(
                              color: Colors.red,
                              borderRadius: BorderRadius.circular(10),
                            ),
                            constraints: const BoxConstraints(
                              minWidth: 16,
                              minHeight: 16,
                            ),
                            child: Text(
                              '${cart.itemCount}',
                              style: const TextStyle(
                                color: Colors.white,
                                fontSize: 10,
                              ),
                              textAlign: TextAlign.center,
                            ),
                          )
                        : const SizedBox();
                  },
                ),
              ),
            ],
          ),
        ],
      ),
      body: ListView.builder(
        itemCount: products.length,
        itemBuilder: (context, index) {
          final product = products[index];
          return ListTile(
            title: Text(product.name),
            subtitle: Text('${product.price} บาท'),
            trailing: IconButton(
              icon: const Icon(Icons.add_shopping_cart),
              onPressed: () {
                context.read<CartModel>().addItem(product);
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(
                    content: Text('เพิ่ม ${product.name} ลงตะกร้า'),
                    duration: const Duration(seconds: 1),
                  ),
                );
              },
            ),
          );
        },
      ),
    );
  }
}

// Cart Page
class CartPage extends StatelessWidget {
  const CartPage({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('ตะกร้าสินค้า')),
      body: Consumer<CartModel>(
        builder: (context, cart, child) {
          if (cart.itemCount == 0) {
            return const Center(
              child: Text('ตะกร้าว่าง'),
            );
          }
          
          return Column(
            children: [
              Expanded(
                child: ListView.builder(
                  itemCount: cart.itemCount,
                  itemBuilder: (context, index) {
                    final product = cart.items[index];
                    return ListTile(
                      title: Text(product.name),
                      subtitle: Text('${product.price} บาท'),
                      trailing: IconButton(
                        icon: const Icon(Icons.delete),
                        onPressed: () {
                          cart.removeItem(product.id);
                        },
                      ),
                    );
                  },
                ),
              ),
              Container(
                padding: const EdgeInsets.all(16),
                decoration: BoxDecoration(
                  color: Colors.grey[200],
                  boxShadow: [
                    BoxShadow(
                      color: Colors.black12,
                      blurRadius: 4,
                      offset: const Offset(0, -2),
                    ),
                  ],
                ),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    Text(
                      'รวม: ${cart.totalPrice} บาท',
                      style: const TextStyle(
                        fontSize: 20,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    ElevatedButton(
                      onPressed: () {
                        // ชำระเงิน
                        cart.clear();
                        Navigator.pop(context);
                      },
                      child: const Text('ชำระเงิน'),
                    ),
                  ],
                ),
              ),
            ],
          );
        },
      ),
    );
  }
}
```

## 4.4 Riverpod

Riverpod เป็น state management ที่พัฒนาโดยผู้สร้าง Provider แต่ปรับปรุงหลายอย่าง

### การติดตั้ง

```yaml
# pubspec.yaml
dependencies:
  flutter_riverpod: ^2.4.0
```

### ตัวอย่างพื้นฐาน

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// 1. สร้าง provider
final counterProvider = StateProvider<int>((ref) => 0);

// 2. Wrap app ด้วย ProviderScope
void main() {
  runApp(
    const ProviderScope(
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});
  
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: const CounterPage(),
    );
  }
}

// 3. ใช้งาน provider
class CounterPage extends ConsumerWidget {
  const CounterPage({super.key});
  
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    
    return Scaffold(
      appBar: AppBar(title: const Text('Riverpod Counter')),
      body: Center(
        child: Text(
          '$count',
          style: const TextStyle(fontSize: 48),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          ref.read(counterProvider.notifier).state++;
        },
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

### StateNotifier with Riverpod

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

// 1. สร้าง State class
class TodoState {
  final List<String> todos;
  final bool isLoading;
  
  TodoState({
    this.todos = const [],
    this.isLoading = false,
  });
  
  TodoState copyWith({
    List<String>? todos,
    bool? isLoading,
  }) {
    return TodoState(
      todos: todos ?? this.todos,
      isLoading: isLoading ?? this.isLoading,
    );
  }
}

// 2. สร้าง StateNotifier
class TodoNotifier extends StateNotifier<TodoState> {
  TodoNotifier() : super(TodoState());
  
  void addTodo(String todo) {
    state = state.copyWith(
      todos: [...state.todos, todo],
    );
  }
  
  void removeTodo(int index) {
    final newTodos = [...state.todos];
    newTodos.removeAt(index);
    state = state.copyWith(todos: newTodos);
  }
  
  Future<void> loadTodos() async {
    state = state.copyWith(isLoading: true);
    await Future.delayed(const Duration(seconds: 2));
    state = state.copyWith(
      todos: ['ทำการบ้าน', 'ซื้อของ', 'ออกกำลังกาย'],
      isLoading: false,
    );
  }
}

// 3. สร้าง provider
final todoProvider = StateNotifierProvider<TodoNotifier, TodoState>((ref) {
  return TodoNotifier();
});

// 4. ใช้งาน
class TodoPage extends ConsumerWidget {
  const TodoPage({super.key});
  
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todoState = ref.watch(todoProvider);
    
    if (todoState.isLoading) {
      return const Scaffold(
        body: Center(child: CircularProgressIndicator()),
      );
    }
    
    return Scaffold(
      appBar: AppBar(title: const Text('Todo List')),
      body: ListView.builder(
        itemCount: todoState.todos.length,
        itemBuilder: (context, index) {
          return ListTile(
            title: Text(todoState.todos[index]),
            trailing: IconButton(
              icon: const Icon(Icons.delete),
              onPressed: () {
                ref.read(todoProvider.notifier).removeTodo(index);
              },
            ),
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          ref.read(todoProvider.notifier).addTodo('งานใหม่');
        },
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

## 4.5 เปรียบเทียบ State Management Solutions

### ตารางเปรียบเทียบ

| คุณสมบัติ | setState | Provider | Riverpod |
|----------|----------|----------|----------|
| **ความยาก** | ง่าย | ปานกลาง | ปานกลาง |
| **Boilerplate** | น้อย | ปานกลาง | น้อย |
| **Performance** | ดี | ดีมาก | ดีมาก |
| **Testability** | ยาก | ง่าย | ง่ายมาก |
| **Compile-time safety** | ไม่มี | ไม่มี | มี |
| **เหมาะกับ** | Widget เดี่ยว | แอปขนาดกลาง-ใหญ่ | แอปทุกขนาด |

### คำแนะนำในการเลือก

**ใช้ setState เมื่อ:**
- State อยู่ใน widget เดียว
- แอปเล็กๆ หรือ prototype
- ไม่ต้องแชร์ state

**ใช้ Provider เมื่อ:**
- ต้องการ state management ที่แนะนำโดย Flutter team
- มี state ที่ต้องแชร์หลาย widget
- ต้องการ ecosystem ที่เติบโตแล้ว

**ใช้ Riverpod เมื่อ:**
- ต้องการ compile-time safety
- ต้องการ testing ที่ง่าย
- แอปขนาดใหญ่ที่ซับซ้อน
- ต้องการ provider ที่ไม่ผูกกับ BuildContext

## ตัวอย่างแอปจริง: Task Manager

```dart
// จะใช้ Provider pattern
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

// Task model
class Task {
  final String id;
  final String title;
  final String description;
  bool isCompleted;
  
  Task({
    required this.id,
    required this.title,
    required this.description,
    this.isCompleted = false,
  });
}

// Task manager
class TaskManager with ChangeNotifier {
  final List<Task> _tasks = [];
  
  List<Task> get tasks => _tasks;
  List<Task> get completedTasks => _tasks.where((t) => t.isCompleted).toList();
  List<Task> get pendingTasks => _tasks.where((t) => !t.isCompleted).toList();
  
  void addTask(Task task) {
    _tasks.add(task);
    notifyListeners();
  }
  
  void removeTask(String id) {
    _tasks.removeWhere((task) => task.id == id);
    notifyListeners();
  }
  
  void toggleTask(String id) {
    final task = _tasks.firstWhere((task) => task.id == id);
    task.isCompleted = !task.isCompleted;
    notifyListeners();
  }
}

void main() {
  runApp(
    ChangeNotifierProvider(
      create: (_) => TaskManager(),
      child: const MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});
  
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Task Manager',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: const TaskListPage(),
    );
  }
}

class TaskListPage extends StatelessWidget {
  const TaskListPage({super.key});
  
  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 2,
      child: Scaffold(
        appBar: AppBar(
          title: const Text('Task Manager'),
          bottom: const TabBar(
            tabs: [
              Tab(text: 'ยังไม่เสร็จ'),
              Tab(text: 'เสร็จแล้ว'),
            ],
          ),
        ),
        body: TabBarView(
          children: [
            _TaskList(completed: false),
            _TaskList(completed: true),
          ],
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () => _showAddTaskDialog(context),
          child: const Icon(Icons.add),
        ),
      ),
    );
  }
  
  void _showAddTaskDialog(BuildContext context) {
    final titleController = TextEditingController();
    final descController = TextEditingController();
    
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('เพิ่มงานใหม่'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextField(
              controller: titleController,
              decoration: const InputDecoration(labelText: 'หัวข้อ'),
            ),
            TextField(
              controller: descController,
              decoration: const InputDecoration(labelText: 'รายละเอียด'),
              maxLines: 3,
            ),
          ],
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('ยกเลิก'),
          ),
          TextButton(
            onPressed: () {
              if (titleController.text.isNotEmpty) {
                final task = Task(
                  id: DateTime.now().toString(),
                  title: titleController.text,
                  description: descController.text,
                );
                context.read<TaskManager>().addTask(task);
                Navigator.pop(context);
              }
            },
            child: const Text('เพิ่ม'),
          ),
        ],
      ),
    );
  }
}

class _TaskList extends StatelessWidget {
  final bool completed;
  
  const _TaskList({required this.completed});
  
  @override
  Widget build(BuildContext context) {
    return Consumer<TaskManager>(
      builder: (context, taskManager, child) {
        final tasks = completed 
            ? taskManager.completedTasks 
            : taskManager.pendingTasks;
        
        if (tasks.isEmpty) {
          return Center(
            child: Text(
              completed ? 'ไม่มีงานที่เสร็จ' : 'ไม่มีงานที่รอดำเนินการ',
              style: const TextStyle(fontSize: 18, color: Colors.grey),
            ),
          );
        }
        
        return ListView.builder(
          itemCount: tasks.length,
          itemBuilder: (context, index) {
            final task = tasks[index];
            return Dismissible(
              key: Key(task.id),
              onDismissed: (_) {
                taskManager.removeTask(task.id);
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(content: Text('ลบ "${task.title}" แล้ว')),
                );
              },
              background: Container(
                color: Colors.red,
                alignment: Alignment.centerRight,
                padding: const EdgeInsets.only(right: 20),
                child: const Icon(Icons.delete, color: Colors.white),
              ),
              child: CheckboxListTile(
                title: Text(
                  task.title,
                  style: TextStyle(
                    decoration: task.isCompleted 
                        ? TextDecoration.lineThrough 
                        : null,
                  ),
                ),
                subtitle: Text(task.description),
                value: task.isCompleted,
                onChanged: (_) {
                  taskManager.toggleTask(task.id);
                },
              ),
            );
          },
        );
      },
    );
  }
}
```

## แบบฝึกหัด

1. สร้าง Counter App ด้วย setState, Provider, และ Riverpod
2. สร้าง Shopping List ที่สามารถเพิ่ม/ลบสินค้า
3. สร้างระบบ Theme Switcher (Light/Dark mode)
4. สร้าง Todo App ที่มีการ filter (All/Active/Completed)
5. สร้าง User Profile Manager ที่เก็บข้อมูลผู้ใช้

## สรุป

ในบทนี้เราได้เรียนรู้:
- setState() สำหรับ state management แบบง่าย
- InheritedWidget สำหรับส่งข้อมูลลง widget tree
- Provider pattern ที่แนะนำโดย Flutter team
- Riverpod state management ที่ทันสมัย
- การเปรียบเทียบและเลือก state management ที่เหมาะสม

---

**หัวข้อก่อนหน้า**: [บทที่ 3: Flutter Widgets พื้นฐาน](../chapter-03-widgets/README.md)  
**หัวข้อถัดไป**: [บทที่ 5: Navigation และ Routing](../chapter-05-navigation/README.md)
