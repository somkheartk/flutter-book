# บทที่ 8: สร้างแอปพลิเคชันจริง

สร้างแอป Todo List ที่สมบูรณ์ ครบทุกฟีเจอร์

## 8.1 วางแผนแอปพลิเคชัน

### Feature Requirements

**Core Features:**
- ✅ สร้าง Todo
- ✅ แก้ไข Todo
- ✅ ลบ Todo
- ✅ ทำเครื่องหมายเสร็จ/ยังไม่เสร็จ
- ✅ Filter (All/Active/Completed)
- ✅ Search Todo

**Advanced Features:**
- ✅ Categories
- ✅ Due Date
- ✅ Priority (High/Medium/Low)
- ✅ Local Storage
- ✅ Statistics

### Architecture

```
lib/
├── models/
│   ├── todo.dart
│   └── category.dart
├── services/
│   ├── database_service.dart
│   └── preferences_service.dart
├── providers/
│   └── todo_provider.dart
├── screens/
│   ├── home_screen.dart
│   ├── add_todo_screen.dart
│   └── statistics_screen.dart
├── widgets/
│   ├── todo_tile.dart
│   └── filter_chips.dart
└── main.dart
```

## 8.2 สร้าง UI/UX

### Models

```dart
// lib/models/todo.dart
import 'package:hive/hive.dart';

part 'todo.g.dart';

@HiveType(typeId: 0)
enum Priority {
  @HiveField(0)
  low,
  @HiveField(1)
  medium,
  @HiveField(2)
  high,
}

@HiveType(typeId: 1)
class Todo extends HiveObject {
  @HiveField(0)
  late String id;
  
  @HiveField(1)
  late String title;
  
  @HiveField(2)
  late String description;
  
  @HiveField(3)
  late bool isCompleted;
  
  @HiveField(4)
  late DateTime createdAt;
  
  @HiveField(5)
  DateTime? dueDate;
  
  @HiveField(6)
  late Priority priority;
  
  @HiveField(7)
  String? category;
  
  Todo({
    required this.id,
    required this.title,
    required this.description,
    this.isCompleted = false,
    DateTime? createdAt,
    this.dueDate,
    this.priority = Priority.medium,
    this.category,
  }) : createdAt = createdAt ?? DateTime.now();
}
```

### Provider for State Management

```dart
// lib/providers/todo_provider.dart
import 'package:flutter/material.dart';
import 'package:hive/hive.dart';
import '../models/todo.dart';

class TodoProvider with ChangeNotifier {
  final Box<Todo> _box = Hive.box<Todo>('todos');
  String _filter = 'all'; // all, active, completed
  String _searchQuery = '';
  
  List<Todo> get todos {
    List<Todo> allTodos = _box.values.toList();
    
    // Apply search
    if (_searchQuery.isNotEmpty) {
      allTodos = allTodos.where((todo) {
        return todo.title.toLowerCase().contains(_searchQuery.toLowerCase()) ||
               todo.description.toLowerCase().contains(_searchQuery.toLowerCase());
      }).toList();
    }
    
    // Apply filter
    if (_filter == 'active') {
      allTodos = allTodos.where((todo) => !todo.isCompleted).toList();
    } else if (_filter == 'completed') {
      allTodos = allTodos.where((todo) => todo.isCompleted).toList();
    }
    
    // Sort by priority and date
    allTodos.sort((a, b) {
      if (a.isCompleted != b.isCompleted) {
        return a.isCompleted ? 1 : -1;
      }
      if (a.priority != b.priority) {
        return b.priority.index.compareTo(a.priority.index);
      }
      return b.createdAt.compareTo(a.createdAt);
    });
    
    return allTodos;
  }
  
  String get filter => _filter;
  
  void setFilter(String filter) {
    _filter = filter;
    notifyListeners();
  }
  
  void setSearchQuery(String query) {
    _searchQuery = query;
    notifyListeners();
  }
  
  Future<void> addTodo(Todo todo) async {
    await _box.put(todo.id, todo);
    notifyListeners();
  }
  
  Future<void> updateTodo(Todo todo) async {
    await todo.save();
    notifyListeners();
  }
  
  Future<void> deleteTodo(String id) async {
    await _box.delete(id);
    notifyListeners();
  }
  
  Future<void> toggleComplete(String id) async {
    final todo = _box.get(id);
    if (todo != null) {
      todo.isCompleted = !todo.isCompleted;
      await todo.save();
      notifyListeners();
    }
  }
  
  // Statistics
  int get totalTodos => _box.length;
  int get completedTodos => _box.values.where((t) => t.isCompleted).length;
  int get activeTodos => _box.values.where((t) => !t.isCompleted).length;
  
  Map<Priority, int> get todosByPriority {
    return {
      Priority.high: _box.values.where((t) => t.priority == Priority.high && !t.isCompleted).length,
      Priority.medium: _box.values.where((t) => t.priority == Priority.medium && !t.isCompleted).length,
      Priority.low: _box.values.where((t) => t.priority == Priority.low && !t.isCompleted).length,
    };
  }
}
```

### Main App

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'package:hive_flutter/hive_flutter.dart';
import 'models/todo.dart';
import 'providers/todo_provider.dart';
import 'screens/home_screen.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // Initialize Hive
  await Hive.initFlutter();
  Hive.registerAdapter(TodoAdapter());
  Hive.registerAdapter(PriorityAdapter());
  await Hive.openBox<Todo>('todos');
  
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});
  
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => TodoProvider(),
      child: MaterialApp(
        title: 'Todo App',
        theme: ThemeData(
          primarySwatch: Colors.blue,
          useMaterial3: true,
        ),
        darkTheme: ThemeData.dark(useMaterial3: true),
        home: const HomeScreen(),
      ),
    );
  }
}
```

### Home Screen

```dart
// lib/screens/home_screen.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/todo_provider.dart';
import '../widgets/todo_tile.dart';
import '../widgets/filter_chips.dart';
import 'add_todo_screen.dart';
import 'statistics_screen.dart';

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Todo List'),
        actions: [
          IconButton(
            icon: const Icon(Icons.bar_chart),
            onPressed: () {
              Navigator.push(
                context,
                MaterialPageRoute(builder: (_) => const StatisticsScreen()),
              );
            },
          ),
        ],
        bottom: PreferredSize(
          preferredSize: const Size.fromHeight(120),
          child: Column(
            children: [
              Padding(
                padding: const EdgeInsets.all(8.0),
                child: TextField(
                  decoration: InputDecoration(
                    hintText: 'ค้นหา...',
                    prefixIcon: const Icon(Icons.search),
                    border: OutlineInputBorder(
                      borderRadius: BorderRadius.circular(10),
                    ),
                    filled: true,
                  ),
                  onChanged: (value) {
                    context.read<TodoProvider>().setSearchQuery(value);
                  },
                ),
              ),
              const FilterChips(),
            ],
          ),
        ),
      ),
      body: Consumer<TodoProvider>(
        builder: (context, provider, child) {
          final todos = provider.todos;
          
          if (todos.isEmpty) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(
                    Icons.check_circle_outline,
                    size: 100,
                    color: Colors.grey[300],
                  ),
                  const SizedBox(height: 16),
                  Text(
                    'ไม่มี Todo',
                    style: TextStyle(
                      fontSize: 20,
                      color: Colors.grey[600],
                    ),
                  ),
                ],
              ),
            );
          }
          
          return ListView.builder(
            itemCount: todos.length,
            itemBuilder: (context, index) {
              return TodoTile(todo: todos[index]);
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton.extended(
        onPressed: () {
          Navigator.push(
            context,
            MaterialPageRoute(builder: (_) => const AddTodoScreen()),
          );
        },
        icon: const Icon(Icons.add),
        label: const Text('เพิ่ม Todo'),
      ),
    );
  }
}
```

### Todo Tile Widget

```dart
// lib/widgets/todo_tile.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../models/todo.dart';
import '../providers/todo_provider.dart';
import '../screens/add_todo_screen.dart';

class TodoTile extends StatelessWidget {
  final Todo todo;
  
  const TodoTile({super.key, required this.todo});
  
  @override
  Widget build(BuildContext context) {
    return Dismissible(
      key: Key(todo.id),
      background: Container(
        color: Colors.red,
        alignment: Alignment.centerRight,
        padding: const EdgeInsets.only(right: 20),
        child: const Icon(Icons.delete, color: Colors.white),
      ),
      direction: DismissDirection.endToStart,
      onDismissed: (_) {
        context.read<TodoProvider>().deleteTodo(todo.id);
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('ลบ "${todo.title}" แล้ว')),
        );
      },
      child: Card(
        margin: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
        child: ListTile(
          leading: Checkbox(
            value: todo.isCompleted,
            onChanged: (_) {
              context.read<TodoProvider>().toggleComplete(todo.id);
            },
          ),
          title: Text(
            todo.title,
            style: TextStyle(
              decoration: todo.isCompleted 
                  ? TextDecoration.lineThrough 
                  : null,
              color: todo.isCompleted ? Colors.grey : null,
            ),
          ),
          subtitle: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              if (todo.description.isNotEmpty)
                Text(
                  todo.description,
                  maxLines: 2,
                  overflow: TextOverflow.ellipsis,
                ),
              const SizedBox(height: 4),
              Row(
                children: [
                  _PriorityChip(priority: todo.priority),
                  const SizedBox(width: 8),
                  if (todo.dueDate != null)
                    _DueDateChip(dueDate: todo.dueDate!),
                ],
              ),
            ],
          ),
          trailing: IconButton(
            icon: const Icon(Icons.edit),
            onPressed: () {
              Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (_) => AddTodoScreen(todo: todo),
                ),
              );
            },
          ),
          isThreeLine: true,
        ),
      ),
    );
  }
}

class _PriorityChip extends StatelessWidget {
  final Priority priority;
  
  const _PriorityChip({required this.priority});
  
  @override
  Widget build(BuildContext context) {
    Color color;
    String label;
    
    switch (priority) {
      case Priority.high:
        color = Colors.red;
        label = 'สูง';
        break;
      case Priority.medium:
        color = Colors.orange;
        label = 'ปานกลาง';
        break;
      case Priority.low:
        color = Colors.green;
        label = 'ต่ำ';
        break;
    }
    
    return Chip(
      label: Text(label, style: const TextStyle(fontSize: 12)),
      backgroundColor: color.withOpacity(0.2),
      side: BorderSide(color: color),
      padding: EdgeInsets.zero,
      visualDensity: VisualDensity.compact,
    );
  }
}

class _DueDateChip extends StatelessWidget {
  final DateTime dueDate;
  
  const _DueDateChip({required this.dueDate});
  
  @override
  Widget build(BuildContext context) {
    final isOverdue = dueDate.isBefore(DateTime.now());
    final color = isOverdue ? Colors.red : Colors.blue;
    final label = '${dueDate.day}/${dueDate.month}/${dueDate.year}';
    
    return Chip(
      avatar: Icon(Icons.calendar_today, size: 16, color: color),
      label: Text(label, style: const TextStyle(fontSize: 12)),
      backgroundColor: color.withOpacity(0.1),
      side: BorderSide(color: color),
      padding: EdgeInsets.zero,
      visualDensity: VisualDensity.compact,
    );
  }
}
```

### Add/Edit Todo Screen

```dart
// lib/screens/add_todo_screen.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../models/todo.dart';
import '../providers/todo_provider.dart';

class AddTodoScreen extends StatefulWidget {
  final Todo? todo;
  
  const AddTodoScreen({super.key, this.todo});
  
  @override
  State<AddTodoScreen> createState() => _AddTodoScreenState();
}

class _AddTodoScreenState extends State<AddTodoScreen> {
  final _formKey = GlobalKey<FormState>();
  late TextEditingController _titleController;
  late TextEditingController _descriptionController;
  Priority _priority = Priority.medium;
  DateTime? _dueDate;
  
  @override
  void initState() {
    super.initState();
    _titleController = TextEditingController(text: widget.todo?.title);
    _descriptionController = TextEditingController(text: widget.todo?.description);
    _priority = widget.todo?.priority ?? Priority.medium;
    _dueDate = widget.todo?.dueDate;
  }
  
  @override
  void dispose() {
    _titleController.dispose();
    _descriptionController.dispose();
    super.dispose();
  }
  
  Future<void> _selectDate() async {
    final picked = await showDatePicker(
      context: context,
      initialDate: _dueDate ?? DateTime.now(),
      firstDate: DateTime.now(),
      lastDate: DateTime.now().add(const Duration(days: 365)),
    );
    
    if (picked != null) {
      setState(() {
        _dueDate = picked;
      });
    }
  }
  
  void _save() {
    if (_formKey.currentState!.validate()) {
      final todo = Todo(
        id: widget.todo?.id ?? DateTime.now().toString(),
        title: _titleController.text,
        description: _descriptionController.text,
        priority: _priority,
        dueDate: _dueDate,
        isCompleted: widget.todo?.isCompleted ?? false,
        createdAt: widget.todo?.createdAt,
      );
      
      context.read<TodoProvider>().addTodo(todo);
      Navigator.pop(context);
    }
  }
  
  @override
  Widget build(BuildContext context) {
    final isEditing = widget.todo != null;
    
    return Scaffold(
      appBar: AppBar(
        title: Text(isEditing ? 'แก้ไข Todo' : 'เพิ่ม Todo'),
      ),
      body: Form(
        key: _formKey,
        child: ListView(
          padding: const EdgeInsets.all(16),
          children: [
            TextFormField(
              controller: _titleController,
              decoration: const InputDecoration(
                labelText: 'หัวข้อ',
                border: OutlineInputBorder(),
              ),
              validator: (value) {
                if (value == null || value.isEmpty) {
                  return 'กรุณากรอกหัวข้อ';
                }
                return null;
              },
            ),
            const SizedBox(height: 16),
            TextFormField(
              controller: _descriptionController,
              decoration: const InputDecoration(
                labelText: 'รายละเอียด',
                border: OutlineInputBorder(),
              ),
              maxLines: 4,
            ),
            const SizedBox(height: 16),
            const Text('ลำดับความสำคัญ', style: TextStyle(fontSize: 16)),
            const SizedBox(height: 8),
            SegmentedButton<Priority>(
              segments: const [
                ButtonSegment(
                  value: Priority.low,
                  label: Text('ต่ำ'),
                  icon: Icon(Icons.arrow_downward),
                ),
                ButtonSegment(
                  value: Priority.medium,
                  label: Text('ปานกลาง'),
                  icon: Icon(Icons.remove),
                ),
                ButtonSegment(
                  value: Priority.high,
                  label: Text('สูง'),
                  icon: Icon(Icons.arrow_upward),
                ),
              ],
              selected: {_priority},
              onSelectionChanged: (Set<Priority> selected) {
                setState(() {
                  _priority = selected.first;
                });
              },
            ),
            const SizedBox(height: 16),
            ListTile(
              title: const Text('วันที่ครบกำหนด'),
              subtitle: Text(
                _dueDate != null
                    ? '${_dueDate!.day}/${_dueDate!.month}/${_dueDate!.year}'
                    : 'ไม่ระบุ',
              ),
              trailing: Row(
                mainAxisSize: MainAxisSize.min,
                children: [
                  if (_dueDate != null)
                    IconButton(
                      icon: const Icon(Icons.clear),
                      onPressed: () {
                        setState(() {
                          _dueDate = null;
                        });
                      },
                    ),
                  IconButton(
                    icon: const Icon(Icons.calendar_today),
                    onPressed: _selectDate,
                  ),
                ],
              ),
              tileColor: Colors.grey[100],
              shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(8),
              ),
            ),
            const SizedBox(height: 24),
            ElevatedButton(
              onPressed: _save,
              style: ElevatedButton.styleFrom(
                padding: const EdgeInsets.symmetric(vertical: 16),
              ),
              child: Text(
                isEditing ? 'บันทึก' : 'เพิ่ม Todo',
                style: const TextStyle(fontSize: 16),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Filter Chips Widget

```dart
// lib/widgets/filter_chips.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/todo_provider.dart';

class FilterChips extends StatelessWidget {
  const FilterChips({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Consumer<TodoProvider>(
      builder: (context, provider, child) {
        return Padding(
          padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 8),
          child: Row(
            children: [
              FilterChip(
                label: const Text('ทั้งหมด'),
                selected: provider.filter == 'all',
                onSelected: (_) => provider.setFilter('all'),
              ),
              const SizedBox(width: 8),
              FilterChip(
                label: const Text('ยังไม่เสร็จ'),
                selected: provider.filter == 'active',
                onSelected: (_) => provider.setFilter('active'),
              ),
              const SizedBox(width: 8),
              FilterChip(
                label: const Text('เสร็จแล้ว'),
                selected: provider.filter == 'completed',
                onSelected: (_) => provider.setFilter('completed'),
              ),
            ],
          ),
        );
      },
    );
  }
}
```

### Statistics Screen

```dart
// lib/screens/statistics_screen.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../models/todo.dart';
import '../providers/todo_provider.dart';

class StatisticsScreen extends StatelessWidget {
  const StatisticsScreen({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('สถิติ')),
      body: Consumer<TodoProvider>(
        builder: (context, provider, child) {
          final total = provider.totalTodos;
          final completed = provider.completedTodos;
          final active = provider.activeTodos;
          final completionRate = total > 0 ? (completed / total * 100).round() : 0;
          
          return ListView(
            padding: const EdgeInsets.all(16),
            children: [
              _StatCard(
                title: 'Todo ทั้งหมด',
                value: total.toString(),
                icon: Icons.list,
                color: Colors.blue,
              ),
              _StatCard(
                title: 'เสร็จแล้ว',
                value: completed.toString(),
                icon: Icons.check_circle,
                color: Colors.green,
              ),
              _StatCard(
                title: 'ยังไม่เสร็จ',
                value: active.toString(),
                icon: Icons.pending,
                color: Colors.orange,
              ),
              _StatCard(
                title: 'อัตราความสำเร็จ',
                value: '$completionRate%',
                icon: Icons.trending_up,
                color: Colors.purple,
              ),
              const SizedBox(height: 16),
              Card(
                child: Padding(
                  padding: const EdgeInsets.all(16),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      const Text(
                        'ตามลำดับความสำคัญ',
                        style: TextStyle(
                          fontSize: 18,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      const SizedBox(height: 16),
                      _PriorityRow(
                        label: 'สูง',
                        count: provider.todosByPriority[Priority.high] ?? 0,
                        color: Colors.red,
                      ),
                      _PriorityRow(
                        label: 'ปานกลาง',
                        count: provider.todosByPriority[Priority.medium] ?? 0,
                        color: Colors.orange,
                      ),
                      _PriorityRow(
                        label: 'ต่ำ',
                        count: provider.todosByPriority[Priority.low] ?? 0,
                        color: Colors.green,
                      ),
                    ],
                  ),
                ),
              ),
            ],
          );
        },
      ),
    );
  }
}

class _StatCard extends StatelessWidget {
  final String title;
  final String value;
  final IconData icon;
  final Color color;
  
  const _StatCard({
    required this.title,
    required this.value,
    required this.icon,
    required this.color,
  });
  
  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.only(bottom: 12),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Row(
          children: [
            Container(
              padding: const EdgeInsets.all(12),
              decoration: BoxDecoration(
                color: color.withOpacity(0.1),
                borderRadius: BorderRadius.circular(12),
              ),
              child: Icon(icon, color: color, size: 32),
            ),
            const SizedBox(width: 16),
            Expanded(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    title,
                    style: const TextStyle(
                      fontSize: 14,
                      color: Colors.grey,
                    ),
                  ),
                  Text(
                    value,
                    style: const TextStyle(
                      fontSize: 24,
                      fontWeight: FontWeight.bold,
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

class _PriorityRow extends StatelessWidget {
  final String label;
  final int count;
  final Color color;
  
  const _PriorityRow({
    required this.label,
    required this.count,
    required this.color,
  });
  
  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 8),
      child: Row(
        children: [
          Container(
            width: 12,
            height: 12,
            decoration: BoxDecoration(
              color: color,
              shape: BoxShape.circle,
            ),
          ),
          const SizedBox(width: 12),
          Expanded(child: Text(label)),
          Text(
            count.toString(),
            style: const TextStyle(
              fontWeight: FontWeight.bold,
              fontSize: 16,
            ),
          ),
        ],
      ),
    );
  }
}
```

## 8.3 Testing

```dart
// test/todo_provider_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:hive_test/hive_test.dart';

void main() {
  setUp(() async {
    await setUpTestHive();
  });
  
  tearDown(() async {
    await tearDownTestHive();
  });
  
  test('Add todo should increase count', () async {
    // Test code here
  });
}
```

## 8.4 Build และ Deploy

### Android

```bash
# Build APK
flutter build apk --release

# Build App Bundle
flutter build appbundle --release
```

### iOS

```bash
# Build iOS
flutter build ios --release
```

### Web

```bash
# Build Web
flutter build web --release
```

## สรุป

ในบทนี้เราได้:
- วางแผนและออกแบบแอป Todo ที่สมบูรณ์
- ใช้ Hive สำหรับ local storage
- ใช้ Provider สำหรับ state management
- สร้าง UI/UX ที่ดี มี filtering และ search
- เพิ่มฟีเจอร์ขั้นสูง เช่น priority, due date
- สร้างหน้าสถิติ
- Build แอปสำหรับ production

---

**หัวข้อก่อนหน้า**: [บทที่ 7: Local Storage](../chapter-07-storage/README.md)  
**ภาคผนวก**: [Appendix](../../appendix/README.md)
