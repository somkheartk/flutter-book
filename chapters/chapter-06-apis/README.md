# บทที่ 6: การทำงานกับ APIs

การเชื่อมต่อกับ REST API และจัดการข้อมูลจาก backend

## 6.1 HTTP Package

### การติดตั้ง

```yaml
# pubspec.yaml
dependencies:
  http: ^1.1.0
```

```bash
flutter pub get
```

### Import Package

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;
```

## 6.2 GET Requests

### ตัวอย่างพื้นฐาน

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

// Model class
class Post {
  final int id;
  final String title;
  final String body;
  
  Post({required this.id, required this.title, required this.body});
  
  factory Post.fromJson(Map<String, dynamic> json) {
    return Post(
      id: json['id'],
      title: json['title'],
      body: json['body'],
    );
  }
}

// Fetch data
Future<List<Post>> fetchPosts() async {
  final response = await http.get(
    Uri.parse('https://jsonplaceholder.typicode.com/posts'),
  );
  
  if (response.statusCode == 200) {
    List<dynamic> jsonData = jsonDecode(response.body);
    return jsonData.map((json) => Post.fromJson(json)).toList();
  } else {
    throw Exception('Failed to load posts');
  }
}

// UI
class PostsPage extends StatefulWidget {
  const PostsPage({super.key});
  
  @override
  State<PostsPage> createState() => _PostsPageState();
}

class _PostsPageState extends State<PostsPage> {
  late Future<List<Post>> _posts;
  
  @override
  void initState() {
    super.initState();
    _posts = fetchPosts();
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Posts')),
      body: FutureBuilder<List<Post>>(
        future: _posts,
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          } else if (snapshot.hasError) {
            return Center(child: Text('Error: ${snapshot.error}'));
          } else if (!snapshot.hasData || snapshot.data!.isEmpty) {
            return const Center(child: Text('No data'));
          }
          
          return ListView.builder(
            itemCount: snapshot.data!.length,
            itemBuilder: (context, index) {
              final post = snapshot.data![index];
              return ListTile(
                title: Text(post.title),
                subtitle: Text(post.body, maxLines: 2),
                onTap: () {
                  // Navigate to detail
                },
              );
            },
          );
        },
      ),
    );
  }
}
```

### GET with Parameters

```dart
Future<List<Post>> fetchUserPosts(int userId) async {
  final response = await http.get(
    Uri.parse('https://jsonplaceholder.typicode.com/posts')
        .replace(queryParameters: {'userId': userId.toString()}),
  );
  
  if (response.statusCode == 200) {
    List<dynamic> jsonData = jsonDecode(response.body);
    return jsonData.map((json) => Post.fromJson(json)).toList();
  } else {
    throw Exception('Failed to load posts');
  }
}
```

## 6.3 POST, PUT, DELETE Requests

### POST - สร้างข้อมูลใหม่

```dart
Future<Post> createPost(String title, String body) async {
  final response = await http.post(
    Uri.parse('https://jsonplaceholder.typicode.com/posts'),
    headers: {'Content-Type': 'application/json'},
    body: jsonEncode({
      'title': title,
      'body': body,
      'userId': 1,
    }),
  );
  
  if (response.statusCode == 201) {
    return Post.fromJson(jsonDecode(response.body));
  } else {
    throw Exception('Failed to create post');
  }
}

// ตัวอย่างการใช้งาน
class CreatePostPage extends StatefulWidget {
  const CreatePostPage({super.key});
  
  @override
  State<CreatePostPage> createState() => _CreatePostPageState();
}

class _CreatePostPageState extends State<CreatePostPage> {
  final _titleController = TextEditingController();
  final _bodyController = TextEditingController();
  bool _isLoading = false;
  
  Future<void> _submitPost() async {
    setState(() {
      _isLoading = true;
    });
    
    try {
      final post = await createPost(
        _titleController.text,
        _bodyController.text,
      );
      
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('สร้าง post สำเร็จ: ${post.id}')),
        );
        Navigator.pop(context);
      }
    } catch (e) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('Error: $e')),
        );
      }
    } finally {
      setState(() {
        _isLoading = false;
      });
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Create Post')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            TextField(
              controller: _titleController,
              decoration: const InputDecoration(labelText: 'Title'),
            ),
            const SizedBox(height: 16),
            TextField(
              controller: _bodyController,
              decoration: const InputDecoration(labelText: 'Body'),
              maxLines: 5,
            ),
            const SizedBox(height: 24),
            ElevatedButton(
              onPressed: _isLoading ? null : _submitPost,
              child: _isLoading
                  ? const CircularProgressIndicator()
                  : const Text('Submit'),
            ),
          ],
        ),
      ),
    );
  }
  
  @override
  void dispose() {
    _titleController.dispose();
    _bodyController.dispose();
    super.dispose();
  }
}
```

### PUT - อัพเดทข้อมูล

```dart
Future<Post> updatePost(int id, String title, String body) async {
  final response = await http.put(
    Uri.parse('https://jsonplaceholder.typicode.com/posts/$id'),
    headers: {'Content-Type': 'application/json'},
    body: jsonEncode({
      'id': id,
      'title': title,
      'body': body,
      'userId': 1,
    }),
  );
  
  if (response.statusCode == 200) {
    return Post.fromJson(jsonDecode(response.body));
  } else {
    throw Exception('Failed to update post');
  }
}
```

### DELETE - ลบข้อมูล

```dart
Future<void> deletePost(int id) async {
  final response = await http.delete(
    Uri.parse('https://jsonplaceholder.typicode.com/posts/$id'),
  );
  
  if (response.statusCode != 200) {
    throw Exception('Failed to delete post');
  }
}
```

## 6.4 JSON Parsing

### JSON Models

```dart
class User {
  final int id;
  final String name;
  final String email;
  final Address address;
  
  User({
    required this.id,
    required this.name,
    required this.email,
    required this.address,
  });
  
  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'],
      name: json['name'],
      email: json['email'],
      address: Address.fromJson(json['address']),
    );
  }
  
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'email': email,
      'address': address.toJson(),
    };
  }
}

class Address {
  final String street;
  final String city;
  final String zipcode;
  
  Address({
    required this.street,
    required this.city,
    required this.zipcode,
  });
  
  factory Address.fromJson(Map<String, dynamic> json) {
    return Address(
      street: json['street'],
      city: json['city'],
      zipcode: json['zipcode'],
    );
  }
  
  Map<String, dynamic> toJson() {
    return {
      'street': street,
      'city': city,
      'zipcode': zipcode,
    };
  }
}
```

### JSON Serialization (json_serializable)

```yaml
# pubspec.yaml
dependencies:
  json_annotation: ^4.8.1

dev_dependencies:
  build_runner: ^2.4.6
  json_serializable: ^6.7.1
```

```dart
import 'package:json_annotation/json_annotation.dart';

part 'user.g.dart';

@JsonSerializable()
class User {
  final int id;
  final String name;
  final String email;
  
  User({required this.id, required this.name, required this.email});
  
  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
  Map<String, dynamic> toJson() => _$UserToJson(this);
}
```

```bash
# Generate code
flutter pub run build_runner build
```

## 6.5 Error Handling

### Try-Catch Error Handling

```dart
class ApiService {
  static const String baseUrl = 'https://api.example.com';
  
  Future<List<Post>> getPosts() async {
    try {
      final response = await http.get(
        Uri.parse('$baseUrl/posts'),
      ).timeout(const Duration(seconds: 10));
      
      if (response.statusCode == 200) {
        List<dynamic> jsonData = jsonDecode(response.body);
        return jsonData.map((json) => Post.fromJson(json)).toList();
      } else if (response.statusCode == 404) {
        throw NotFoundException('Posts not found');
      } else if (response.statusCode == 500) {
        throw ServerException('Server error');
      } else {
        throw HttpException('Failed with status: ${response.statusCode}');
      }
    } on TimeoutException {
      throw NetworkException('Request timeout');
    } on SocketException {
      throw NetworkException('No internet connection');
    } catch (e) {
      throw Exception('Unknown error: $e');
    }
  }
}

// Custom Exceptions
class NetworkException implements Exception {
  final String message;
  NetworkException(this.message);
  
  @override
  String toString() => message;
}

class NotFoundException implements Exception {
  final String message;
  NotFoundException(this.message);
  
  @override
  String toString() => message;
}

class ServerException implements Exception {
  final String message;
  ServerException(this.message);
  
  @override
  String toString() => message;
}
```

### Result Pattern

```dart
class Result<T> {
  final T? data;
  final String? error;
  final bool isSuccess;
  
  Result.success(this.data)
      : error = null,
        isSuccess = true;
  
  Result.failure(this.error)
      : data = null,
        isSuccess = false;
}

class ApiService {
  Future<Result<List<Post>>> getPosts() async {
    try {
      final response = await http.get(
        Uri.parse('https://jsonplaceholder.typicode.com/posts'),
      );
      
      if (response.statusCode == 200) {
        List<dynamic> jsonData = jsonDecode(response.body);
        List<Post> posts = jsonData.map((json) => Post.fromJson(json)).toList();
        return Result.success(posts);
      } else {
        return Result.failure('Failed to load posts: ${response.statusCode}');
      }
    } catch (e) {
      return Result.failure('Error: $e');
    }
  }
}

// การใช้งาน
class PostsPage extends StatefulWidget {
  @override
  State<PostsPage> createState() => _PostsPageState();
}

class _PostsPageState extends State<PostsPage> {
  final ApiService _apiService = ApiService();
  List<Post> _posts = [];
  bool _isLoading = true;
  String? _error;
  
  @override
  void initState() {
    super.initState();
    _loadPosts();
  }
  
  Future<void> _loadPosts() async {
    setState(() {
      _isLoading = true;
      _error = null;
    });
    
    final result = await _apiService.getPosts();
    
    setState(() {
      _isLoading = false;
      if (result.isSuccess) {
        _posts = result.data!;
      } else {
        _error = result.error;
      }
    });
  }
  
  @override
  Widget build(BuildContext context) {
    if (_isLoading) {
      return const Center(child: CircularProgressIndicator());
    }
    
    if (_error != null) {
      return Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Error: $_error'),
            ElevatedButton(
              onPressed: _loadPosts,
              child: const Text('Retry'),
            ),
          ],
        ),
      );
    }
    
    return ListView.builder(
      itemCount: _posts.length,
      itemBuilder: (context, index) {
        return ListTile(
          title: Text(_posts[index].title),
        );
      },
    );
  }
}
```

## ตัวอย่างแอปจริง: News App

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

// Models
class Article {
  final String title;
  final String description;
  final String url;
  final String? imageUrl;
  final DateTime publishedAt;
  
  Article({
    required this.title,
    required this.description,
    required this.url,
    this.imageUrl,
    required this.publishedAt,
  });
  
  factory Article.fromJson(Map<String, dynamic> json) {
    return Article(
      title: json['title'] ?? 'No Title',
      description: json['description'] ?? '',
      url: json['url'] ?? '',
      imageUrl: json['urlToImage'],
      publishedAt: DateTime.parse(json['publishedAt']),
    );
  }
}

// API Service
class NewsApiService {
  static const String baseUrl = 'https://newsapi.org/v2';
  static const String apiKey = 'YOUR_API_KEY_HERE';
  
  Future<List<Article>> getTopHeadlines(String country) async {
    final response = await http.get(
      Uri.parse('$baseUrl/top-headlines?country=$country&apiKey=$apiKey'),
    );
    
    if (response.statusCode == 200) {
      final Map<String, dynamic> data = jsonDecode(response.body);
      final List<dynamic> articles = data['articles'];
      return articles.map((json) => Article.fromJson(json)).toList();
    } else {
      throw Exception('Failed to load news');
    }
  }
  
  Future<List<Article>> searchNews(String query) async {
    final response = await http.get(
      Uri.parse('$baseUrl/everything?q=$query&apiKey=$apiKey'),
    );
    
    if (response.statusCode == 200) {
      final Map<String, dynamic> data = jsonDecode(response.body);
      final List<dynamic> articles = data['articles'];
      return articles.map((json) => Article.fromJson(json)).toList();
    } else {
      throw Exception('Failed to search news');
    }
  }
}

// News Page
class NewsPage extends StatefulWidget {
  const NewsPage({super.key});
  
  @override
  State<NewsPage> createState() => _NewsPageState();
}

class _NewsPageState extends State<NewsPage> {
  final NewsApiService _apiService = NewsApiService();
  List<Article> _articles = [];
  bool _isLoading = true;
  String? _error;
  
  @override
  void initState() {
    super.initState();
    _loadNews();
  }
  
  Future<void> _loadNews() async {
    setState(() {
      _isLoading = true;
      _error = null;
    });
    
    try {
      final articles = await _apiService.getTopHeadlines('us');
      setState(() {
        _articles = articles;
        _isLoading = false;
      });
    } catch (e) {
      setState(() {
        _error = e.toString();
        _isLoading = false;
      });
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('News'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: _loadNews,
          ),
        ],
      ),
      body: _buildBody(),
    );
  }
  
  Widget _buildBody() {
    if (_isLoading) {
      return const Center(child: CircularProgressIndicator());
    }
    
    if (_error != null) {
      return Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.error_outline, size: 60, color: Colors.red),
            const SizedBox(height: 16),
            Text('Error: $_error'),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: _loadNews,
              child: const Text('Retry'),
            ),
          ],
        ),
      );
    }
    
    if (_articles.isEmpty) {
      return const Center(child: Text('No articles'));
    }
    
    return RefreshIndicator(
      onRefresh: _loadNews,
      child: ListView.builder(
        itemCount: _articles.length,
        itemBuilder: (context, index) {
          final article = _articles[index];
          return ArticleCard(article: article);
        },
      ),
    );
  }
}

class ArticleCard extends StatelessWidget {
  final Article article;
  
  const ArticleCard({super.key, required this.article});
  
  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.all(8),
      child: InkWell(
        onTap: () {
          // Open article URL
        },
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            if (article.imageUrl != null)
              Image.network(
                article.imageUrl!,
                height: 200,
                width: double.infinity,
                fit: BoxFit.cover,
                errorBuilder: (context, error, stackTrace) {
                  return Container(
                    height: 200,
                    color: Colors.grey[300],
                    child: const Icon(Icons.image, size: 60),
                  );
                },
              ),
            Padding(
              padding: const EdgeInsets.all(12),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    article.title,
                    style: const TextStyle(
                      fontSize: 18,
                      fontWeight: FontWeight.bold,
                    ),
                    maxLines: 2,
                    overflow: TextOverflow.ellipsis,
                  ),
                  const SizedBox(height: 8),
                  Text(
                    article.description,
                    style: const TextStyle(color: Colors.grey),
                    maxLines: 3,
                    overflow: TextOverflow.ellipsis,
                  ),
                  const SizedBox(height: 8),
                  Text(
                    _formatDate(article.publishedAt),
                    style: const TextStyle(fontSize: 12, color: Colors.grey),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
  
  String _formatDate(DateTime date) {
    return '${date.day}/${date.month}/${date.year} ${date.hour}:${date.minute}';
  }
}
```

## แบบฝึกหัด

1. สร้างแอป Todo ที่เชื่อมต่อ JSONPlaceholder API
2. สร้างฟอร์มสร้าง/แก้ไข User ด้วย POST/PUT
3. เพิ่ม Loading และ Error handling
4. สร้างระบบ Search ข้อมูล
5. เพิ่ม Pull-to-refresh

## สรุป

ในบทนี้เราได้เรียนรู้:
- HTTP Package สำหรับเรียก API
- GET, POST, PUT, DELETE requests
- JSON Parsing และ Models
- Error Handling
- สร้าง News App ที่เชื่อมต่อ API จริง

---

**หัวข้อก่อนหน้า**: [บทที่ 5: Navigation และ Routing](../chapter-05-navigation/README.md)  
**หัวข้อถัดไป**: [บทที่ 7: Local Storage](../chapter-07-storage/README.md)
