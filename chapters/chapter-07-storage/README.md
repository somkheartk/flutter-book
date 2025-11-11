# บทที่ 7: Local Storage

การจัดเก็บข้อมูลในเครื่องผู้ใช้

## 7.1 SharedPreferences

เหมาะสำหรับเก็บข้อมูลง่ายๆ เช่น การตั้งค่า, token

### การติดตั้ง

```yaml
# pubspec.yaml
dependencies:
  shared_preferences: ^2.2.2
```

### การใช้งาน

```dart
import 'package:shared_preferences/shared_preferences.dart';

class PreferencesService {
  // Save data
  Future<void> saveString(String key, String value) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString(key, value);
  }
  
  Future<void> saveInt(String key, int value) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setInt(key, value);
  }
  
  Future<void> saveBool(String key, bool value) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setBool(key, value);
  }
  
  Future<void> saveStringList(String key, List<String> value) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setStringList(key, value);
  }
  
  // Read data
  Future<String?> getString(String key) async {
    final prefs = await SharedPreferences.getInstance();
    return prefs.getString(key);
  }
  
  Future<int?> getInt(String key) async {
    final prefs = await SharedPreferences.getInstance();
    return prefs.getInt(key);
  }
  
  Future<bool?> getBool(String key) async {
    final prefs = await SharedPreferences.getInstance();
    return prefs.getBool(key);
  }
  
  // Remove data
  Future<void> remove(String key) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.remove(key);
  }
  
  // Clear all
  Future<void> clear() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.clear();
  }
}

// ตัวอย่างการใช้งาน
class SettingsPage extends StatefulWidget {
  @override
  State<SettingsPage> createState() => _SettingsPageState();
}

class _SettingsPageState extends State<SettingsPage> {
  final PreferencesService _prefs = PreferencesService();
  bool _darkMode = false;
  bool _notifications = true;
  String _username = '';
  
  @override
  void initState() {
    super.initState();
    _loadSettings();
  }
  
  Future<void> _loadSettings() async {
    final darkMode = await _prefs.getBool('darkMode') ?? false;
    final notifications = await _prefs.getBool('notifications') ?? true;
    final username = await _prefs.getString('username') ?? '';
    
    setState(() {
      _darkMode = darkMode;
      _notifications = notifications;
      _username = username;
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Settings')),
      body: ListView(
        children: [
          SwitchListTile(
            title: const Text('Dark Mode'),
            value: _darkMode,
            onChanged: (value) {
              setState(() {
                _darkMode = value;
              });
              _prefs.saveBool('darkMode', value);
            },
          ),
          SwitchListTile(
            title: const Text('Notifications'),
            value: _notifications,
            onChanged: (value) {
              setState(() {
                _notifications = value;
              });
              _prefs.saveBool('notifications', value);
            },
          ),
          ListTile(
            title: const Text('Username'),
            subtitle: Text(_username.isEmpty ? 'Not set' : _username),
            trailing: const Icon(Icons.edit),
            onTap: () async {
              final result = await showDialog<String>(
                context: context,
                builder: (context) => _UsernameDialog(initialValue: _username),
              );
              
              if (result != null) {
                setState(() {
                  _username = result;
                });
                _prefs.saveString('username', result);
              }
            },
          ),
        ],
      ),
    );
  }
}
```

## 7.2 SQLite Database

เหมาะสำหรับข้อมูลที่ซับซ้อนและมีปริมาณมาก

### การติดตั้ง

```yaml
# pubspec.yaml
dependencies:
  sqflite: ^2.3.0
  path: ^1.8.3
```

### Database Helper

```dart
import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart';

class DatabaseHelper {
  static final DatabaseHelper instance = DatabaseHelper._init();
  static Database? _database;
  
  DatabaseHelper._init();
  
  Future<Database> get database async {
    if (_database != null) return _database!;
    _database = await _initDB('notes.db');
    return _database!;
  }
  
  Future<Database> _initDB(String filePath) async {
    final dbPath = await getDatabasesPath();
    final path = join(dbPath, filePath);
    
    return await openDatabase(
      path,
      version: 1,
      onCreate: _createDB,
    );
  }
  
  Future _createDB(Database db, int version) async {
    await db.execute('''
      CREATE TABLE notes (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT NOT NULL,
        content TEXT NOT NULL,
        createdAt TEXT NOT NULL
      )
    ''');
  }
  
  Future close() async {
    final db = await instance.database;
    db.close();
  }
}

// Model
class Note {
  final int? id;
  final String title;
  final String content;
  final DateTime createdAt;
  
  Note({
    this.id,
    required this.title,
    required this.content,
    required this.createdAt,
  });
  
  Map<String, dynamic> toMap() {
    return {
      'id': id,
      'title': title,
      'content': content,
      'createdAt': createdAt.toIso8601String(),
    };
  }
  
  factory Note.fromMap(Map<String, dynamic> map) {
    return Note(
      id: map['id'],
      title: map['title'],
      content: map['content'],
      createdAt: DateTime.parse(map['createdAt']),
    );
  }
}

// CRUD Operations
class NotesDatabase {
  final DatabaseHelper _dbHelper = DatabaseHelper.instance;
  
  Future<int> create(Note note) async {
    final db = await _dbHelper.database;
    return await db.insert('notes', note.toMap());
  }
  
  Future<List<Note>> readAll() async {
    final db = await _dbHelper.database;
    final result = await db.query('notes', orderBy: 'createdAt DESC');
    return result.map((map) => Note.fromMap(map)).toList();
  }
  
  Future<Note?> read(int id) async {
    final db = await _dbHelper.database;
    final maps = await db.query(
      'notes',
      where: 'id = ?',
      whereArgs: [id],
    );
    
    if (maps.isNotEmpty) {
      return Note.fromMap(maps.first);
    }
    return null;
  }
  
  Future<int> update(Note note) async {
    final db = await _dbHelper.database;
    return await db.update(
      'notes',
      note.toMap(),
      where: 'id = ?',
      whereArgs: [note.id],
    );
  }
  
  Future<int> delete(int id) async {
    final db = await _dbHelper.database;
    return await db.delete(
      'notes',
      where: 'id = ?',
      whereArgs: [id],
    );
  }
}

// UI
class NotesPage extends StatefulWidget {
  @override
  State<NotesPage> createState() => _NotesPageState();
}

class _NotesPageState extends State<NotesPage> {
  final NotesDatabase _notesDb = NotesDatabase();
  List<Note> _notes = [];
  
  @override
  void initState() {
    super.initState();
    _loadNotes();
  }
  
  Future<void> _loadNotes() async {
    final notes = await _notesDb.readAll();
    setState(() {
      _notes = notes;
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Notes')),
      body: ListView.builder(
        itemCount: _notes.length,
        itemBuilder: (context, index) {
          final note = _notes[index];
          return ListTile(
            title: Text(note.title),
            subtitle: Text(note.content, maxLines: 2),
            trailing: IconButton(
              icon: const Icon(Icons.delete),
              onPressed: () async {
                await _notesDb.delete(note.id!);
                _loadNotes();
              },
            ),
            onTap: () {
              // Edit note
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () async {
          await _notesDb.create(
            Note(
              title: 'New Note',
              content: 'Content...',
              createdAt: DateTime.now(),
            ),
          );
          _loadNotes();
        },
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

## 7.3 File Storage

เหมาะสำหรับไฟล์ เช่น รูปภาพ, เอกสาร

### การติดตั้ง

```yaml
# pubspec.yaml
dependencies:
  path_provider: ^2.1.1
```

### การใช้งาน

```dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';

class FileService {
  // Get file path
  Future<String> get _localPath async {
    final directory = await getApplicationDocumentsDirectory();
    return directory.path;
  }
  
  Future<File> _localFile(String filename) async {
    final path = await _localPath;
    return File('$path/$filename');
  }
  
  // Write file
  Future<File> writeFile(String filename, String content) async {
    final file = await _localFile(filename);
    return file.writeAsString(content);
  }
  
  // Read file
  Future<String> readFile(String filename) async {
    try {
      final file = await _localFile(filename);
      return await file.readAsString();
    } catch (e) {
      return '';
    }
  }
  
  // Delete file
  Future<void> deleteFile(String filename) async {
    final file = await _localFile(filename);
    await file.delete();
  }
  
  // Check if file exists
  Future<bool> fileExists(String filename) async {
    final file = await _localFile(filename);
    return await file.exists();
  }
  
  // List files
  Future<List<FileSystemEntity>> listFiles() async {
    final directory = await getApplicationDocumentsDirectory();
    return directory.listSync();
  }
}
```

## 7.4 Hive Database

NoSQL database ที่เร็วและใช้งานง่าย

### การติดตั้ง

```yaml
# pubspec.yaml
dependencies:
  hive: ^2.2.3
  hive_flutter: ^1.1.0

dev_dependencies:
  hive_generator: ^2.0.1
  build_runner: ^2.4.6
```

### การใช้งาน

```dart
import 'package:hive_flutter/hive_flutter.dart';

// 1. Initialize Hive
void main() async {
  await Hive.initFlutter();
  await Hive.openBox('settings');
  runApp(MyApp());
}

// 2. Simple usage
class HiveService {
  final Box _box = Hive.box('settings');
  
  // Save
  Future<void> save(String key, dynamic value) async {
    await _box.put(key, value);
  }
  
  // Read
  dynamic get(String key, {dynamic defaultValue}) {
    return _box.get(key, defaultValue: defaultValue);
  }
  
  // Delete
  Future<void> delete(String key) async {
    await _box.delete(key);
  }
  
  // Clear all
  Future<void> clear() async {
    await _box.clear();
  }
}

// 3. With Type Adapter
@HiveType(typeId: 0)
class Task extends HiveObject {
  @HiveField(0)
  late String title;
  
  @HiveField(1)
  late bool isCompleted;
  
  @HiveField(2)
  late DateTime createdAt;
}

// Generate adapter: flutter pub run build_runner build

// 4. Usage with Type Adapter
void main() async {
  await Hive.initFlutter();
  Hive.registerAdapter(TaskAdapter());
  await Hive.openBox<Task>('tasks');
  runApp(MyApp());
}

class TaskService {
  final Box<Task> _box = Hive.box<Task>('tasks');
  
  Future<void> addTask(Task task) async {
    await _box.add(task);
  }
  
  List<Task> getAllTasks() {
    return _box.values.toList();
  }
  
  Future<void> updateTask(int index, Task task) async {
    await _box.putAt(index, task);
  }
  
  Future<void> deleteTask(int index) async {
    await _box.deleteAt(index);
  }
}
```

## 7.5 เลือก Storage ที่เหมาะสม

### ตารางเปรียบเทียบ

| Storage | ข้อดี | ข้อเสีย | เหมาะกับ |
|---------|-------|---------|----------|
| **SharedPreferences** | ง่าย, รวดเร็ว | เก็บข้อมูลง่ายๆ เท่านั้น | การตั้งค่า, token |
| **SQLite** | Query ซับซ้อนได้, มาตรฐาน | ซับซ้อน, ช้ากว่า | ข้อมูลจำนวนมาก, complex queries |
| **File** | เก็บไฟล์ได้ | ต้องจัดการเอง | รูปภาพ, เอกสาร |
| **Hive** | เร็วมาก, ใช้ง่าย | ไม่มี query ซับซ้อน | แอป offline-first |

### คำแนะนำ

**ใช้ SharedPreferences เมื่อ:**
- เก็บการตั้งค่า (theme, language)
- เก็บ token, user preferences
- ข้อมูลน้อย, ง่าย

**ใช้ SQLite เมื่อ:**
- ข้อมูลจำนวนมาก
- ต้องการ complex queries (JOIN, GROUP BY)
- ต้องการ relational database

**ใช้ File Storage เมื่อ:**
- เก็บรูปภาพ, เอกสาร
- เก็บไฟล์ดาวน์โหลด

**ใช้ Hive เมื่อ:**
- ต้องการ performance สูง
- ข้อมูลไม่ซับซ้อนมาก
- ต้องการใช้งานง่าย

## ตัวอย่างแอปจริง: Todo App with Storage

```dart
import 'package:flutter/material.dart';
import 'package:hive_flutter/hive_flutter.dart';

part 'todo.g.dart';

@HiveType(typeId: 0)
class Todo extends HiveObject {
  @HiveField(0)
  late String title;
  
  @HiveField(1)
  late String description;
  
  @HiveField(2)
  late bool isCompleted;
  
  @HiveField(3)
  late DateTime createdAt;
  
  Todo({
    required this.title,
    required this.description,
    this.isCompleted = false,
    DateTime? createdAt,
  }) : createdAt = createdAt ?? DateTime.now();
}

void main() async {
  await Hive.initFlutter();
  Hive.registerAdapter(TodoAdapter());
  await Hive.openBox<Todo>('todos');
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});
  
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Todo App',
      home: const TodoPage(),
    );
  }
}

class TodoPage extends StatelessWidget {
  const TodoPage({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Todo List')),
      body: ValueListenableBuilder(
        valueListenable: Hive.box<Todo>('todos').listenable(),
        builder: (context, Box<Todo> box, _) {
          if (box.isEmpty) {
            return const Center(child: Text('No todos'));
          }
          
          return ListView.builder(
            itemCount: box.length,
            itemBuilder: (context, index) {
              final todo = box.getAt(index)!;
              return Dismissible(
                key: Key(index.toString()),
                onDismissed: (_) => box.deleteAt(index),
                background: Container(
                  color: Colors.red,
                  alignment: Alignment.centerRight,
                  padding: const EdgeInsets.only(right: 20),
                  child: const Icon(Icons.delete, color: Colors.white),
                ),
                child: CheckboxListTile(
                  title: Text(
                    todo.title,
                    style: TextStyle(
                      decoration: todo.isCompleted 
                          ? TextDecoration.lineThrough 
                          : null,
                    ),
                  ),
                  subtitle: Text(todo.description),
                  value: todo.isCompleted,
                  onChanged: (value) {
                    todo.isCompleted = value ?? false;
                    todo.save();
                  },
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _showAddDialog(context),
        child: const Icon(Icons.add),
      ),
    );
  }
  
  void _showAddDialog(BuildContext context) {
    final titleController = TextEditingController();
    final descController = TextEditingController();
    
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('Add Todo'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextField(
              controller: titleController,
              decoration: const InputDecoration(labelText: 'Title'),
            ),
            TextField(
              controller: descController,
              decoration: const InputDecoration(labelText: 'Description'),
            ),
          ],
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('Cancel'),
          ),
          TextButton(
            onPressed: () {
              if (titleController.text.isNotEmpty) {
                final box = Hive.box<Todo>('todos');
                box.add(
                  Todo(
                    title: titleController.text,
                    description: descController.text,
                  ),
                );
                Navigator.pop(context);
              }
            },
            child: const Text('Add'),
          ),
        ],
      ),
    );
  }
}
```

## แบบฝึกหัด

1. สร้างระบบ Login ที่จำ username ด้วย SharedPreferences
2. สร้าง Notes App ด้วย SQLite
3. สร้างแอปบันทึกรูปภาพด้วย File Storage
4. สร้าง Shopping List ด้วย Hive
5. เปรียบเทียบ performance ของ storage ต่างๆ

## สรุป

ในบทนี้เราได้เรียนรู้:
- SharedPreferences สำหรับข้อมูลง่ายๆ
- SQLite สำหรับ relational database
- File Storage สำหรับไฟล์
- Hive สำหรับ NoSQL database ที่เร็ว
- การเลือก storage ที่เหมาะสมกับงาน

---

**หัวข้อก่อนหน้า**: [บทที่ 6: การทำงานกับ APIs](../chapter-06-apis/README.md)  
**หัวข้อถัดไป**: [บทที่ 8: สร้างแอปพลิเคชันจริง](../chapter-08-complete-app/README.md)
