import 'package:flutter/material.dart';
import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart';
import 'package:shared_preferences/shared_preferences.dart';
import 'package:url_launcher/url_launcher.dart';
import 'dart:convert';
import 'dart:io';
import 'dart:math' as math;

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final db = await initDatabase();
  runApp(MyApp(db: db));
}

class MyApp extends StatelessWidget {
  final Database db;

  const MyApp({super.key, required this.db});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'نظام توثيق الشهداء والجرحى والأسرى - قبيلة ذو محمد',
      theme: ThemeData(
        fontFamily: 'Cairo',
        colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF1A237E)),
        useMaterial3: true,
        appBarTheme: const AppBarTheme(
          foregroundColor: Colors.white,
          backgroundColor: Color(0xFF1A237E),
        ),
        textTheme: TextTheme(
          titleLarge: const TextStyle(
            fontSize: 30,
            fontWeight: FontWeight.bold,
            color: Color(0xFF1A237E),
          ),
          titleMedium: const TextStyle(
            fontSize: 25,
            fontWeight: FontWeight.bold,
            color: Color(0xFF1A237E),
          ),
          bodyLarge: const TextStyle(
            fontSize: 18,
            color: Colors.black,
          ),
          bodyMedium: const TextStyle(
            fontSize: 16,
            color: Colors.black,
          ),
        ),
      ),
      home: const LoginScreen(db: db),
    );
  }
}

// ألوان التصميم
final colors = {
  'primary': '#1a237e',
  'primary_light': '#534bae',
  'primary_dark': '#000051',
  'secondary': '#f50057',
  'secondary_light': '#ff5983',
  'secondary_dark': '#bb002f',
  'background': '#f5f5f5',
  'surface': '#ffffff',
  'on_primary': '#ffffff',
  'on_secondary': '#ffffff',
  'on_background': '#000000',
  'on_surface': '#000000',
  'success': '#4caf50',
  'warning': '#ff9800',
  'error': '#f44336',
};

// أنماط الأزرار
final buttonStyle = ElevatedButton.styleFrom(
  foregroundColor: Colors.white,
  backgroundColor: Color(int.parse(colors['primary']!, radix: 16)),
  hoverColor: Color(int.parse(colors['primary_light']!, radix: 16)),
  padding: const EdgeInsets.all(15),
  shape: RoundedRectangleBorder(
    borderRadius: BorderRadius.circular(10),
  ),
);

final secondaryButtonStyle = ElevatedButton.styleFrom(
  foregroundColor: Colors.white,
  backgroundColor: Color(int.parse(colors['secondary']!, radix: 16)),
  hoverColor: Color(int.parse(colors['secondary_light']!, radix: 16)),
  padding: const EdgeInsets.all(15),
  shape: RoundedRectangleBorder(
    borderRadius: BorderRadius.circular(10),
  ),
);

// دالة تهيئة قاعدة البيانات
Future<Database> initDatabase() async {
  final databasesPath = await getDatabasesPath();
  final path = join(databasesPath, 'martyrs.db');

  return await openDatabase(
    path,
    version: 1,
    onCreate: (db, version) async {
      await db.execute('''
        CREATE TABLE IF NOT EXISTS users (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          username TEXT UNIQUE NOT NULL,
          email TEXT UNIQUE NOT NULL,
          password TEXT NOT NULL,
          is_admin INTEGER DEFAULT 0
        )
      ''');

      await db.execute('''
        CREATE TABLE IF NOT EXISTS martyrs (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          name TEXT NOT NULL,
          age INTEGER,
          date_of_martyrdom TEXT,
          location TEXT,
          details TEXT,
          image_path TEXT,
          created_at TEXT DEFAULT CURRENT_TIMESTAMP
        )
      ''');

      await db.execute('''
        CREATE TABLE IF NOT EXISTS wounded (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          name TEXT NOT NULL,
          age INTEGER,
          injury_date TEXT,
          injury_location TEXT,
          injury_details TEXT,
          medical_status TEXT,
          image_path TEXT,
          created_at TEXT DEFAULT CURRENT_TIMESTAMP
        )
      ''');

      await db.execute('''
        CREATE TABLE IF NOT EXISTS prisoners (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          name TEXT NOT NULL,
          age INTEGER,
          arrest_date TEXT,
          arrest_location TEXT,
          prison_name TEXT,
          details TEXT,
          image_path TEXT,
          created_at TEXT DEFAULT CURRENT_TIMESTAMP
        )
      ''');

      await db.execute('''
        CREATE TABLE IF NOT EXISTS statistics (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          martyrs_count INTEGER DEFAULT 0,
          wounded_count INTEGER DEFAULT 0,
          prisoners_count INTEGER DEFAULT 0,
          last_updated TEXT DEFAULT CURRENT_TIMESTAMP
        )
      ''');

      // إضافة المستخدم المسؤول إذا لم يكن موجودًا
      final count = await db.rawQuery('SELECT COUNT(*) FROM users WHERE email = ?', ['admin@sture.com']);
      if (count.first[0] == 0) {
        await db.insert('users', {
          'username': 'admin',
          'email': 'admin@sture.com',
          'password': 'admin123',
          'is_admin': 1,
        });
      }

      // إضافة سجل إحصائي أولي
      final statsCount = await db.rawQuery('SELECT COUNT(*) FROM statistics');
      if (statsCount.first[0] == 0) {
        await db.insert('statistics', {
          'martyrs_count': 0,
          'wounded_count': 0,
          'prisoners_count': 0,
        });
      }
    },
  );
}

// دالة للتحقق من المنافذ المتاحة (في Flutter لا نستخدمها — نستخدم منفذ ثابت أو لا نستخدم)
int findAvailablePort() {
  return 8000;
}

// نموذج المستخدم
class User {
  final int? id;
  final String username;
  final String email;
  final bool isAdmin;

  User({
    this.id,
    required this.username,
    required this.email,
    required this.isAdmin,
  });

  factory User.fromMap(Map<String, dynamic> map) {
    return User(
      id: map['id'],
      username: map['username'],
      email: map['email'],
      isAdmin: map['is_admin'] == 1,
    );
  }

  Map<String, dynamic> toMap() {
    return {
      'id': id,
      'username': username,
      'email': email,
      'is_admin': isAdmin ? 1 : 0,
    };
  }
}

// نموذج الإحصائيات
class Statistics {
  final int martyrsCount;
  final int woundedCount;
  final int prisonersCount;
  final String lastUpdated;

  Statistics({
    required this.martyrsCount,
    required this.woundedCount,
    required this.prisonersCount,
    required this.lastUpdated,
  });

  factory Statistics.fromMap(Map<String, dynamic> map) {
    return Statistics(
      martyrsCount: map['martyrs_count'],
      woundedCount: map['wounded_count'],
      prisonersCount: map['prisoners_count'],
      lastUpdated: map['last_updated'] ?? '',
    );
  }
}

// وظائف المساعدة
class Helper {
  static Future<void> showSnackBar(BuildContext context, String message, [Color? color]) {
    final scaffold = ScaffoldMessenger.of(context);
    scaffold.hideCurrentSnackBar();
    scaffold.showSnackBar(
      SnackBar(
        content: Text(message),
        backgroundColor: color ?? const Color(0xFFF50057),
      ),
    );
  }

  static Future<Statistics> getStatistics(Database db) async {
    final List<Map<String, dynamic>> result = await db.query('statistics', orderBy: 'id DESC', limit: 1);
    if (result.isEmpty) {
      return Statistics(martyrsCount: 0, woundedCount: 0, prisonersCount: 0, lastUpdated: '');
    }
    return Statistics.fromMap(result.first);
  }

  static Future<void> updateStatistics(Database db) async {
    final martyrsCount = await db.rawQuery('SELECT COUNT(*) FROM martyrs');
    final woundedCount = await db.rawQuery('SELECT COUNT(*) FROM wounded');
    final prisonersCount = await db.rawQuery('SELECT COUNT(*) FROM prisoners');

    await db.update(
      'statistics',
      {
        'martyrs_count': martyrsCount.first[0],
        'wounded_count': woundedCount.first[0],
        'prisoners_count': prisonersCount.first[0],
        'last_updated': DateTime.now().toIso8601String(),
      },
      where: 'id = (SELECT MAX(id) FROM statistics)',
    );
  }

  static Future<List<Map<String, dynamic>>> queryTable(Database db, String table) async {
    return await db.query(table, orderBy: 'created_at DESC');
  }

  static Future<void> backupData(Database db) async {
    final martyrs = await db.query('martyrs');
    final wounded = await db.query('wounded');
    final prisoners = await db.query('prisoners');
    final users = await db.query('users');

    final backupData = {
      'martyrs': martyrs,
      'wounded': wounded,
      'prisoners': prisoners,
      'users': users,
      'backup_date': DateTime.now().toIso8601String(),
    };

    final jsonString = jsonEncode(backupData);
    final directory = Directory('/storage/emulated/0/Download');
    if (!await directory.exists()) {
      await directory.create(recursive: true);
    }
    final file = File('${directory.path}/backup_${DateTime.now().toString().replaceAll(':', '-').substring(0, 19)}.json');
    await file.writeAsString(jsonString, encoding: utf8);
  }

  static Future<User?> loginUser(Database db, String email, String password) async {
    final List<Map<String, dynamic>> result = await db.query(
      'users',
      where: 'email = ? AND password = ?',
      whereArgs: [email, password],
    );
    if (result.isNotEmpty) {
      final user = User.fromMap(result.first);
      return user;
    }
    return null;
  }

  static Future<bool> registerUser(Database db, String username, String email, String password, String confirmPassword) async {
    if (password != confirmPassword) {
      return false;
    }
    if (password.length < 6) {
      return false;
    }
    try {
      await db.insert(
        'users',
        {
          'username': username,
          'email': email,
          'password': password,
          'is_admin': 0,
        },
      );
      return true;
    } on DatabaseException catch (e) {
      if (e.message!.contains('UNIQUE constraint failed')) {
        return false;
      }
      rethrow;
    }
  }
}

// واجهات المستخدم
class LoginScreen extends StatefulWidget {
  final Database db;

  const LoginScreen({super.key, required this.db});

  @override
  State<LoginScreen> createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Color(int.parse(colors['background']!, radix: 16)),
      body: Center(
        child: Container(
          padding: const EdgeInsets.all(50),
          width: 500,
          decoration: BoxDecoration(
            color: Color(int.parse(colors['surface']!, radix: 16)),
            borderRadius: BorderRadius.circular(20),
            boxShadow: [
              BoxShadow(
                spreadRadius: 1,
                blurRadius: 15,
                color: Color(int.parse(colors['primary_dark']!, radix: 16)),
                offset: const Offset(0, 0),
                blurStyle: BlurStyle.outer,
              ),
            ],
          ),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.center,
            children: [
              Container(
                margin: const EdgeInsets.only(bottom: 30),
                child: Image.network(
                  'https://cdn-icons-png.flaticon.com/512/100/100493.png',
                  width: 100,
                  height: 100,
                ),
              ),
              const Text(
                'تسجيل الدخول',
                style: TextStyle(
                  fontSize: 30,
                  fontWeight: FontWeight.bold,
                  color: Color(0xFF1A237E),
                ),
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _emailController,
                decoration: InputDecoration(
                  labelText: 'البريد الإلكتروني',
                  hintText: 'أدخل بريدك الإلكتروني',
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(10),
                    borderSide: BorderSide(color: Color(int.parse(colors['primary']!, radix: 16))),
                  ),
                  prefixIcon: const Icon(Icons.email),
                ),
                style: const TextStyle(fontFamily: 'Cairo'),
                width: 400,
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _passwordController,
                decoration: InputDecoration(
                  labelText: 'كلمة المرور',
                  hintText: 'أدخل كلمة المرور',
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(10),
                    borderSide: BorderSide(color: Color(int.parse(colors['primary']!, radix: 16))),
                  ),
                  prefixIcon: const Icon(Icons.lock),
                  isPassword: true,
                ),
                style: const TextStyle(fontFamily: 'Cairo'),
                width: 400,
              ),
              const SizedBox(height: 20),
              ElevatedButton(
                onPressed: () async {
                  final user = await Helper.loginUser(widget.db, _emailController.text, _passwordController.text);
                  if (user != null) {
                    if (user.isAdmin) {
                      Navigator.pushReplacement(
                        context,
                        MaterialPageRoute(builder: (context) => AdminDashboardScreen(db: widget.db, user: user)),
                      );
                    } else {
                      Navigator.pushReplacement(
                        context,
                        MaterialPageRoute(builder: (context) => UserDashboardScreen(db: widget.db, user: user)),
                      );
                    }
                  } else {
                    Helper.showSnackBar(context, 'البريد الإلكتروني أو كلمة المرور غير صحيحة');
                  }
                },
                style: buttonStyle,
                child: const Text('تسجيل الدخول', style: TextStyle(fontFamily: 'Cairo')),
                width: 400,
              ),
              TextButton(
                onPressed: () {
                  Navigator.push(
                    context,
                    MaterialPageRoute(builder: (context) => SignupScreen(db: widget.db)),
                  );
                },
                child: const Text('ليس لديك حساب؟ إنشاء حساب', style: TextStyle(fontFamily: 'Cairo')),
              ),
            ],
          ),
        ),
      ),
    );
  }

  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }
}

class SignupScreen extends StatefulWidget {
  final Database db;

  const SignupScreen({super.key, required this.db});

  @override
  State<SignupScreen> createState() => _SignupScreenState();
}

class _SignupScreenState extends State<SignupScreen> {
  final _usernameController = TextEditingController();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  final _confirmPasswordController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Color(int.parse(colors['background']!, radix: 16)),
      body: Center(
        child: Container(
          padding: const EdgeInsets.all(50),
          width: 500,
          decoration: BoxDecoration(
            color: Color(int.parse(colors['surface']!, radix: 16)),
            borderRadius: BorderRadius.circular(20),
            boxShadow: [
              BoxShadow(
                spreadRadius: 1,
                blurRadius: 15,
                color: Color(int.parse(colors['primary_dark']!, radix: 16)),
                offset: const Offset(0, 0),
                blurStyle: BlurStyle.outer,
              ),
            ],
          ),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.center,
            children: [
              Container(
                margin: const EdgeInsets.only(bottom: 30),
                child: Image.network(
                  'https://cdn-icons-png.flaticon.com/512/100/100493.png',
                  width: 100,
                  height: 100,
                ),
              ),
              const Text(
                'إنشاء حساب',
                style: TextStyle(
                  fontSize: 30,
                  fontWeight: FontWeight.bold,
                  color: Color(0xFF1A237E),
                ),
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _usernameController,
                decoration: InputDecoration(
                  labelText: 'اسم المستخدم',
                  hintText: 'اختر اسم مستخدم',
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(10),
                    borderSide: BorderSide(color: Color(int.parse(colors['primary']!, radix: 16))),
                  ),
                  prefixIcon: const Icon(Icons.person),
                ),
                style: const TextStyle(fontFamily: 'Cairo'),
                width: 400,
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _emailController,
                decoration: InputDecoration(
                  labelText: 'البريد الإلكتروني',
                  hintText: 'أدخل بريدك الإلكتروني',
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(10),
                    borderSide: BorderSide(color: Color(int.parse(colors['primary']!, radix: 16))),
                  ),
                  prefixIcon: const Icon(Icons.email),
                ),
                style: const TextStyle(fontFamily: 'Cairo'),
                width: 400,
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _passwordController,
                decoration: InputDecoration(
                  labelText: 'كلمة المرور',
                  hintText: 'اختر كلمة مرور قوية',
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(10),
                    borderSide: BorderSide(color: Color(int.parse(colors['primary']!, radix: 16))),
                  ),
                  prefixIcon: const Icon(Icons.lock),
                  isPassword: true,
                ),
                style: const TextStyle(fontFamily: 'Cairo'),
                width: 400,
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _confirmPasswordController,
                decoration: InputDecoration(
                  labelText: 'تأكيد كلمة المرور',
                  hintText: 'أعد إدخال كلمة المرور',
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(10),
                    borderSide: BorderSide(color: Color(int.parse(colors['primary']!, radix: 16))),
                  ),
                  prefixIcon: const Icon(Icons.lock),
                  isPassword: true,
                ),
                style: const TextStyle(fontFamily: 'Cairo'),
                width: 400,
              ),
              const SizedBox(height: 20),
              ElevatedButton(
                onPressed: () async {
                  if (await Helper.registerUser(
                    widget.db,
                    _usernameController.text,
                    _emailController.text,
                    _passwordController.text,
                    _confirmPasswordController.text,
                  )) {
                    Navigator.pushReplacement(
                      context,
                      MaterialPageRoute(builder: (context) => LoginScreen(db: widget.db)),
                    );
                  } else {
                    if (_passwordController.text != _confirmPasswordController.text) {
                      Helper.showSnackBar(context, 'كلمات المرور غير متطابقة');
                    } else if (_passwordController.text.length < 6) {
                      Helper.showSnackBar(context, 'كلمة المرور يجب أن تكون على الأقل 6 أحرف');
                    } else {
                      Helper.showSnackBar(context, 'البريد الإلكتروني أو اسم المستخدم موجود مسبقاً');
                    }
                  }
                },
                style: buttonStyle,
                child: const Text('إنشاء حساب', style: TextStyle(fontFamily: 'Cairo')),
                width: 400,
              ),
              TextButton(
                onPressed: () {
                  Navigator.pushReplacement(
                    context,
                    MaterialPageRoute(builder: (context) => LoginScreen(db: widget.db)),
                  );
                },
                child: const Text('لديك حساب بالفعل؟ تسجيل الدخول', style: TextStyle(fontFamily: 'Cairo')),
              ),
            ],
          ),
        ),
      ),
    );
  }

  @override
  void dispose() {
    _usernameController.dispose();
    _emailController.dispose();
    _passwordController.dispose();
    _confirmPasswordController.dispose();
    super.dispose();
  }
}

class UserDashboardScreen extends StatefulWidget {
  final Database db;
  final User user;

  const UserDashboardScreen({super.key, required this.db, required this.user});

  @override
  State<UserDashboardScreen> createState() => _UserDashboardScreenState();
}

class _UserDashboardScreenState extends State<UserDashboardScreen> {
  late Statistics _stats;

  @override
  void initState() {
    super.initState();
    _loadStatistics();
  }

  Future<void> _loadStatistics() async {
    final stats = await Helper.getStatistics(widget.db);
    setState(() {
      _stats = stats;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Color(int.parse(colors['background']!, radix: 16)),
      body: Center(
        child: Container(
          padding: const EdgeInsets.all(20),
          width: 500,
          decoration: BoxDecoration(
            color: Color(int.parse(colors['surface']!, radix: 16)),
            borderRadius: BorderRadius.circular(20),
            boxShadow: [
              BoxShadow(
                spreadRadius: 1,
                blurRadius: 15,
                color: Color(int.parse(colors['primary_dark']!, radix: 16)),
                offset: const Offset(0, 0),
                blurStyle: BlurStyle.outer,
              ),
            ],
          ),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.center,
            children: [
              Container(
                margin: const EdgeInsets.only(bottom: 30),
                child: const Text(
                  "لوحة المستخدم",
                  style: TextStyle(
                    fontSize: 30,
                    fontWeight: FontWeight.bold,
                    color: Color(0xFF1A237E),
                  ),
                ),
              ),
              Container(
                margin: const EdgeInsets.only(bottom: 30),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    Container(
                      padding: const EdgeInsets.all(15),
                      margin: const EdgeInsets.all(10),
                      decoration: BoxDecoration(
                        color: Color(int.parse(colors['primary']!, radix: 16)),
                        borderRadius: BorderRadius.circular(10),
                      ),
                      child: Column(
                        mainAxisAlignment: MainAxisAlignment.center,
                        children: [
                          const Text(
                            "الشهداء",
                            style: TextStyle(
                              color: Colors.white,
                              fontSize: 16,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                          Text(
                            "${_stats.martyrsCount}",
                            style: const TextStyle(
                              color: Colors.white,
                              fontSize: 24,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                        ],
                      ),
                    ),
                    Container(
                      padding: const EdgeInsets.all(15),
                      margin: const EdgeInsets.all(10),
                      decoration: BoxDecoration(
                        color: Color(int.parse(colors['secondary']!, radix: 16)),
                        borderRadius: BorderRadius.circular(10),
                      ),
                      child: Column(
                        mainAxisAlignment: MainAxisAlignment.center,
                        children: [
                          const Text(
                            "الجرحى",
                            style: TextStyle(
                              color: Colors.white,
                              fontSize: 16,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                          Text(
                            "${_stats.woundedCount}",
                            style: const TextStyle(
                              color: Colors.white,
                              fontSize: 24,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                        ],
                      ),
                    ),
                    Container(
                      padding: const EdgeInsets.all(15),
                      margin: const EdgeInsets.all(10),
                      decoration: BoxDecoration(
                        color: Color(int.parse(colors['primary_dark']!, radix: 16)),
                        borderRadius: BorderRadius.circular(10),
                      ),
                      child: Column(
                        mainAxisAlignment: MainAxisAlignment.center,
                        children: [
                          const Text(
                            "الأسرى",
                            style: TextStyle(
                              color: Colors.white,
                              fontSize: 16,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                          Text(
                            "${_stats.prisonersCount}",
                            style: const TextStyle(
                              color: Colors.white,
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
              ElevatedButton(
                onPressed: () {
                  Navigator.push(
                    context,
                    MaterialPageRoute(builder: (context) => MartyrsFormScreen(db: widget.db)),
                  );
                },
                style: buttonStyle,
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    Icon(Icons.military_tech, color: Colors.white),
                    const Text("قسم الشهداء", style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold, color: Colors.white)),
                  ],
                ),
                width: 300,
              ),
              ElevatedButton(
                onPressed: () {
                  Navigator.push(
                    context,
                    MaterialPageRoute(builder: (context) => WoundedFormScreen(db: widget.db)),
                  );
                },
                style: buttonStyle,
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    Icon(Icons.healing, color: Colors.white),
                    const Text("قسم الجرحى", style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold, color: Colors.white)),
                  ],
                ),
                width: 300,
              ),
              ElevatedButton(
                onPressed: () {
                  Navigator.push(
                    context,
                    MaterialPageRoute(builder: (context) => PrisonersFormScreen(db: widget.db)),
                  );
                },
                style: buttonStyle,
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    Icon(Icons.lock_person, color: Colors.white),
                    const Text("قسم الأسرى", style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold, color: Colors.white)),
                  ],
                ),
                width: 300,
              ),
              ElevatedButton(
                onPressed: () {
                  Navigator.pushReplacement(
                    context,
                    MaterialPageRoute(builder: (context) => LoginScreen(db: widget.db)),
                  );
                },
                style: secondaryButtonStyle,
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    Icon(Icons.logout, color: Colors.white),
                    const Text("تسجيل الخروج", style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold, color: Colors.white)),
                  ],
                ),
                width: 300,
              ),
            ],
          ),
        ),
      ),
    );
  }
}

class AdminDashboardScreen extends StatefulWidget {
  final Database db;
  final User user;

  const AdminDashboardScreen({super.key, required this.db, required this.user});

  @override
  State<AdminDashboardScreen> createState() => _AdminDashboardScreenState();
}

class _AdminDashboardScreenState extends State<AdminDashboardScreen> {
  late Statistics _stats;

  @override
  void initState() {
    super.initState();
    _loadStatistics();
  }

  Future<void> _loadStatistics() async {
    final stats = await Helper.getStatistics(widget.db);
    setState(() {
      _stats = stats;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Color(int.parse(colors['background']!, radix: 16)),
      body: Center(
        child: Container(
          padding: const EdgeInsets.all(20),
          width: 600,
          decoration: BoxDecoration(
            color: Color(int.parse(colors['surface']!, radix: 16)),
            borderRadius: BorderRadius.circular(20),
            boxShadow: [
              BoxShadow(
                spreadRadius: 1,
                blurRadius: 15,
                color: Color(int.parse(colors['primary_dark']!, radix: 16)),
                offset: const Offset(0, 0),
                blurStyle: BlurStyle.outer,
              ),
            ],
          ),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.center,
            children: [
              Container(
                margin: const EdgeInsets.only(bottom: 30),
                child: const Text(
                  "لوحة التحكم الإدارية",
                  style: TextStyle(
                    fontSize: 30,
                    fontWeight: FontWeight.bold,
                    color: Color(0xFF1A237E),
                  ),
                ),
              ),
              Container(
                margin: const EdgeInsets.only(bottom: 30),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    Container(
                      padding: const EdgeInsets.all(15),
                      margin: const EdgeInsets.all(10),
                      decoration: BoxDecoration(
                        color: Color(int.parse(colors['primary']!, radix: 16)),
                        borderRadius: BorderRadius.circular(10),
                      ),
                      child: Column(
                        mainAxisAlignment: MainAxisAlignment.center,
                        children: [
                          const Text(
                            "الشهداء",
                            style: TextStyle(
                              color: Colors.white,
                              fontSize: 16,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                          Text(
                            "${_stats.martyrsCount}",
                            style: const TextStyle(
                              color: Colors.white,
                              fontSize: 24,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                        ],
                      ),
                    ),
                    Container(
                      padding: const EdgeInsets.all(15),
                      margin: const EdgeInsets.all(10),
                      decoration: BoxDecoration(
                        color: Color(int.parse(colors['secondary']!, radix: 16)),
                        borderRadius: BorderRadius.circular(10),
                      ),
                      child: Column(
                        mainAxisAlignment: MainAxisAlignment.center,
                        children: [
                          const Text(
                            "الجرحى",
                            style: TextStyle(
                              color: Colors.white,
                              fontSize: 16,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                          Text(
                            "${_stats.woundedCount}",
                            style: const TextStyle(
                              color: Colors.white,
                              fontSize: 24,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                        ],
                      ),
                    ),
                    Container(
                      padding: const EdgeInsets.all(15),
                      margin: const EdgeInsets.all(10),
                      decoration: BoxDecoration(
                        color: Color(int.parse(colors['primary_dark']!, radix: 16)),
                        borderRadius: BorderRadius.circular(10),
                      ),
                      child: Column(
                        mainAxisAlignment: MainAxisAlignment.center,
                        children: [
                          const Text(
                            "الأسرى",
                            style: TextStyle(
                              color: Colors.white,
                              fontSize: 16,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                          Text(
                            "${_stats.prisonersCount}",
                            style: const TextStyle(
                              color: Colors.white,
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
              Container(
                padding: const EdgeInsets.all(10),
                margin: const EdgeInsets.only(bottom: 30),
                width: 600,
                child: SingleChildScrollView(
                  scrollDirection: Axis.horizontal,
                  child: Row(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      ElevatedButton(
                        onPressed: () {
                          Navigator.push(
                            context,
                            MaterialPageRoute(builder: (context) => MartyrsFormScreen(db: widget.db)),
                          );
                        },
                        style: buttonStyle,
                        child: Column(
                          mainAxisAlignment: MainAxisAlignment.center,
                          children: [
                            Icon(Icons.military_tech, size: 30, color: Colors.white),
                            const Text("إضافة شهداء", style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold, color: Colors.white)),
                          ],
                        ),
                        width: 150,
                        height: 120,
                      ),
                      ElevatedButton(
                        onPressed: () {
                          Navigator.push(
                            context,
                            MaterialPageRoute(builder: (context) => WoundedFormScreen(db: widget.db)),
                          );
                        },
                        style: buttonStyle,
                        child: Column(
                          mainAxisAlignment: MainAxisAlignment.center,
                          children: [
                            Icon(Icons.healing, size: 30, color: Colors.white),
                            const Text("إضافة جرحى", style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold, color: Colors.white)),
                          ],
                        ),
                        width: 150,
                        height: 120,
                      ),
                      ElevatedButton(
                        onPressed: () {
                          Navigator.push(
                            context,
                            MaterialPageRoute(builder: (context) => PrisonersFormScreen(db: widget.db)),
                          );
                        },
                        style: buttonStyle,
                        child: Column(
                          mainAxisAlignment: MainAxisAlignment.center,
                          children: [
                            Icon(Icons.lock_person, size: 30, color: Colors.white),
                            const Text("إضافة أسرى", style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold, color: Colors.white)),
                          ],
                        ),
                        width: 150,
                        height: 120,
                      ),
                      ElevatedButton(
                        onPressed: () {
                          Navigator.push(
                            context,
                            MaterialPageRoute(builder: (context) => MartyrsListScreen(db: widget.db)),
                          );
                        },
                        style: buttonStyle,
                        child: Column(
                          mainAxisAlignment: MainAxisAlignment.center,
                          children: [
                            Icon(Icons.visibility, size: 30, color: Colors.white),
                            const Text("عرض الشهداء", style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold, color: Colors.white)),
                          ],
                        ),
                        width: 150,
                        height: 120,
                      ),
                      ElevatedButton(
                        onPressed: () {
                          Navigator.push(
                            context,
                            MaterialPageRoute(builder: (context) => WoundedListScreen(db: widget.db)),
                          );
                        },
                        style: buttonStyle,
                        child: Column(
                          mainAxisAlignment: MainAxisAlignment.center,
                          children: [
                            Icon(Icons.visibility, size: 30, color: Colors.white),
                            const Text("عرض الجرحى", style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold, color: Colors.white)),
                          ],
                        ),
                        width: 150,
                        height: 120,
                      ),
                      ElevatedButton(
                        onPressed: () {
                          Navigator.push(
                            context,
                            MaterialPageRoute(builder: (context) => PrisonersListScreen(db: widget.db)),
                          );
                        },
                        style: buttonStyle,
                        child: Column(
                          mainAxisAlignment: MainAxisAlignment.center,
                          children: [
                            Icon(Icons.visibility, size: 30, color: Colors.white),
                            const Text("عرض الأسرى", style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold, color: Colors.white)),
                          ],
                        ),
                        width: 150,
                        height: 120,
                      ),
                      ElevatedButton(
                        onPressed: () {
                          Navigator.push(
                            context,
                            MaterialPageRoute(builder: (context) => SettingsScreen(db: widget.db)),
                          );
                        },
                        style: buttonStyle,
                        child: Column(
                          mainAxisAlignment: MainAxisAlignment.center,
                          children: [
                            Icon(Icons.settings, size: 30, color: Colors.white),
                            const Text("الإعدادات", style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold, color: Colors.white)),
                          ],
                        ),
                        width: 150,
                        height: 120,
                      ),
                      ElevatedButton(
                        onPressed: () {
                          Helper.backupData(widget.db);
                          Helper.showSnackBar(context, "تم إنشاء نسخة احتياطية بنجاح");
                        },
                        style: buttonStyle,
                        child: Column(
                          mainAxisAlignment: MainAxisAlignment.center,
                          children: [
                            Icon(Icons.backup, size: 30, color: Colors.white),
                            const Text("النسخ الاحتياطي", style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold, color: Colors.white)),
                          ],
                        ),
                        width: 150,
                        height: 120,
                      ),
                      ElevatedButton(
                        onPressed: () {
                          Navigator.push(
                            context,
                            MaterialPageRoute(builder: (context) => UsersListScreen(db: widget.db)),
                          );
                        },
                        style: buttonStyle,
                        child: Column(
                          mainAxisAlignment: MainAxisAlignment.center,
                          children: [
                            Icon(Icons.people, size: 30, color: Colors.white),
                            const Text("المستخدمين", style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold, color: Colors.white)),
                          ],
                        ),
                        width: 150,
                        height: 120,
                      ),
                    ],
                  ),
                ),
              ),
              ElevatedButton(
                onPressed: () {
                  Navigator.pushReplacement(
                    context,
                    MaterialPageRoute(builder: (context) => LoginScreen(db: widget.db)),
                  );
                },
                style: secondaryButtonStyle,
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    Icon(Icons.logout, color: Colors.white),
                    const Text("تسجيل الخروج", style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold, color: Colors.white)),
                  ],
                ),
                width: 300,
              ),
            ],
          ),
        ),
      ),
    );
  }
}

class MartyrsFormScreen extends StatefulWidget {
  final Database db;

  const MartyrsFormScreen({super.key, required this.db});

  @override
  State<MartyrsFormScreen> createState() => _MartyrsFormScreenState();
}

class _MartyrsFormScreenState extends State<MartyrsFormScreen> {
  final _nameController = TextEditingController();
  final _ageController = TextEditingController();
  final _dateController = TextEditingController();
  final _locationController = TextEditingController();
  final _detailsController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Color(int.parse(colors['background']!, radix: 16)),
      body: Center(
        child: Container(
          padding: const EdgeInsets.all(30),
          width: 500,
          decoration: BoxDecoration(
            color: Color(int.parse(colors['surface']!, radix: 16)),
            borderRadius: BorderRadius.circular(20),
            boxShadow: [
              BoxShadow(
                spreadRadius: 1,
                blurRadius: 15,
                color: Color(int.parse(colors['primary_dark']!, radix: 16)),
                offset: const Offset(0, 0),
                blurStyle: BlurStyle.outer,
              ),
            ],
          ),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.center,
            children: [
              const Text(
                "إضافة بيانات شهيد",
                style: TextStyle(
                  fontSize: 25,
                  fontWeight: FontWeight.bold,
                  color: Color(0xFF1A237E),
                ),
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _nameController,
                decoration: InputDecoration(
                  labelText: "الاسم",
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(10),
                    borderSide: BorderSide(color: Color(int.parse(colors['primary']!, radix: 16))),
                  ),
                ),
                style: const TextStyle(fontFamily: 'Cairo'),
                width: 400,
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _ageController,
                decoration: InputDecoration(
                  labelText: "العمر",
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(10),
                    borderSide: BorderSide(color: Color(int.parse(colors['primary']!, radix: 16))),
                  ),
                ),
                style: const TextStyle(fontFamily: 'Cairo'),
                width: 400,
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _dateController,
                decoration: InputDecoration(
                  labelText: "تاريخ الاستشهاد",
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(10),
                    borderSide: BorderSide(color: Color(int.parse(colors['primary']!, radix: 16))),
                  ),
                ),
                style: const TextStyle(fontFamily: 'Cairo'),
                width: 400,
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _locationController,
                decoration: InputDecoration(
                  labelText: "مكان الاستشهاد",
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(10),
                    borderSide: BorderSide(color: Color(int.parse(colors['primary']!, radix: 16))),
                  ),
                ),
                style: const TextStyle(fontFamily: 'Cairo'),
                width: 400,
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _detailsController,
                decoration: InputDecoration(
                  labelText: "تفاصيل",
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(10),
                    borderSide: BorderSide(color: Color(int.parse(colors['primary']!, radix: 16))),
                  ),
                  hintText: "أدخل تفاصيل عن الشهيد...",
                ),
                maxLines: 3,
                style: const TextStyle(fontFamily: 'Cairo'),
                width: 400,
              ),
              const SizedBox(height: 30),
              Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  ElevatedButton(
                    onPressed: () async {
                      if (_nameController.text.isEmpty || _ageController.text.isEmpty || _dateController.text.isEmpty) {
                        Helper.showSnackBar(context, "يرجى ملء جميع الحقول المطلوبة");
                        return;
                      }
                      await widget.db.insert(
                        'martyrs',
                        {
                          'name': _nameController.text,
                          'age': int.tryParse(_ageController.text) ?? 0,
                          'date_of_martyrdom': _dateController.text,
                          'location': _locationController.text,
                          'details': _detailsController.text,
                        },
                      );
                      Helper.showSnackBar(context, "تم حفظ بيانات الشهيد بنجاح", Color(int.parse(colors['primary']!, radix: 16)));
                      Navigator.pop(context);
                    },
                    style: buttonStyle,
                    child: const Text("حفظ", style: TextStyle(fontFamily: 'Cairo')),
                  ),
                  const SizedBox(width: 20),
                  ElevatedButton(
                    onPressed: () {
                      Navigator.pop(context);
                    },
                    style: secondaryButtonStyle,
                    child: const Text("عودة", style: TextStyle(fontFamily: 'Cairo')),
                  ),
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }

  @override
  void dispose() {
    _nameController.dispose();
    _ageController.dispose();
    _dateController.dispose();
    _locationController.dispose();
    _detailsController.dispose();
    super.dispose();
  }
}

class WoundedFormScreen extends StatefulWidget {
  final Database db;

  const WoundedFormScreen({super.key, required this.db});

  @override
  State<WoundedFormScreen> createState() => _WoundedFormScreenState();
}

class _WoundedFormScreenState extends State<WoundedFormScreen> {
  final _nameController = TextEditingController();
  final _ageController = TextEditingController();
  final _dateController = TextEditingController();
  final _locationController = TextEditingController();
  final _detailsController = TextEditingController();
  final _medicalController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Color(int.parse(colors['background']!, radix: 16)),
      body: Center(
        child: Container(
          padding: const EdgeInsets.all(30),
          width: 500,
          decoration: BoxDecoration(
            color: Color(int.parse(colors['surface']!, radix: 16)),
            borderRadius: BorderRadius.circular(20),
            boxShadow: [
              BoxShadow(
                spreadRadius: 1,
                blurRadius: 15,
                color: Color(int.parse(colors['primary_dark']!, radix: 16)),
                offset: const Offset(0, 0),
                blurStyle: BlurStyle.outer,
              ),
            ],
          ),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.center,
            children: [
              const Text(
                "إضافة بيانات جريح",
                style: TextStyle(
                  fontSize: 25,
                  fontWeight: FontWeight.bold,
                  color: Color(0xFF1A237E),
                ),
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _nameController,
                decoration: InputDecoration(
                  labelText: "الاسم",
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(10),
                    borderSide: BorderSide(color: Color(int.parse(colors['primary']!, radix: 16))),
                  ),
                ),
                style: const TextStyle(fontFamily: 'Cairo'),
                width: 400,
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _ageController,
                decoration: InputDecoration(
                  labelText: "العمر",
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(10),
                    borderSide: BorderSide(color: Color(int.parse(colors['primary']!, radix: 16))),
                  ),
                ),
                style: const TextStyle(fontFamily: 'Cairo'),
                width: 400,
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _dateController,
                decoration: InputDecoration(
                  labelText: "تاريخ الإصابة",
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(10),
                    borderSide: BorderSide(color: Color(int.parse(colors['primary']!, radix: 16))),
                  ),
                ),
                style: const TextStyle(fontFamily: 'Cairo'),
                width: 400,
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _locationController,
                decoration: InputDecoration(
                  labelText: "مكان الإصابة",
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(10),
                    borderSide: BorderSide(color: Color(int.parse(colors['primary']!, radix: 16))),
                  ),
                ),
                style: const TextStyle(fontFamily: 'Cairo'),
                width: 400,
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _detailsController,
                decoration: InputDecoration(
                  labelText: "تفاصيل الإصابة",
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(10),
                    borderSide: BorderSide(color: Color(int.parse(colors['primary']!, radix: 16))),
                  ),
                  hintText: "أدخل تفاصيل عن الإصابة...",
                ),
                maxLines: 3,
                style: const TextStyle(fontFamily: 'Cairo'),
                width: 400,
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _medicalController,
                decoration: InputDecoration(
                  labelText: "الحالة الطبية",
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(10),
                    borderSide: BorderSide(color: Color(int.parse(colors['primary']!, radix: 16))),
                  ),
                ),
                style: const TextStyle(fontFamily: 'Cairo'),
                width: 400,
              ),
              const SizedBox(height: 30),
              Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  ElevatedButton(
                    onPressed: () async {
                      if (_nameController.text.isEmpty || _ageController.text.isEmpty || _dateController.text.isEmpty) {
                        Helper.showSnackBar(context, "يرجى ملء جميع الحقول المطلوبة");
                        return;
                      }
                      await widget.db.insert(
                        'wounded',
                        {
                          'name': _nameController.text,
                          'age': int.tryParse(_ageController.text) ?? 0,
                          'injury_date': _dateController.text,
                          'injury_location': _locationController.text,
                          'injury_details': _detailsController.text,
                          'medical_status': _medicalController.text,
                        },
                      );
                      Helper.showSnackBar(context, "تم حفظ بيانات الجريح بنجاح", Color(int.parse(colors['primary']!, radix: 16)));
                      Navigator.pop(context);
                    },
                    style: buttonStyle,
                    child: const Text("حفظ", style: TextStyle(fontFamily: 'Cairo')),
                  ),
                  const SizedBox(width: 20),
                  ElevatedButton(
                    onPressed: () {
                      Navigator.pop(context);
                    },
                    style: secondaryButtonStyle,
                    child: const Text("عودة", style: TextStyle(fontFamily: 'Cairo')),
                  ),
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }

  @override
  void dispose() {
    _nameController.dispose();
    _ageController.dispose();
    _dateController.dispose();
    _locationController.dispose();
    _detailsController.dispose();
    _medicalController.dispose();
    super.dispose();
  }
}

class PrisonersFormScreen extends StatefulWidget {
  final Database db;

  const PrisonersFormScreen({super.key, required this.db});

  @override
  State<PrisonersFormScreen> createState() => _PrisonersFormScreenState();
}

class _PrisonersFormScreenState extends State<PrisonersFormScreen> {
  final _nameController = TextEditingController();
  final _ageController = TextEditingController();
  final _dateController = TextEditingController();
  final _locationController = TextEditingController();
  final _prisonController = TextEditingController();
  final _detailsController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Color(int.parse(colors['background']!, radix: 16)),
      body: Center(
        child: Container(
          padding: const EdgeInsets.all(30),
          width: 500,
          decoration: BoxDecoration(
            color: Color(int.parse(colors['surface']!, radix: 16)),
            borderRadius: BorderRadius.circular(20),
            boxShadow: [
              BoxShadow(
                spreadRadius: 1,
                blurRadius: 15,
                color: Color(int.parse(colors['primary_dark']!, radix: 16)),
                offset: const Offset(0, 0),
                blurStyle: BlurStyle.outer,
              ),
            ],
          ),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.center,
            children: [
              const Text(
                "إضافة بيانات أسير",
                style: TextStyle(
                  fontSize: 25,
                  fontWeight: FontWeight.bold,
                  color: Color(0xFF1A237E),
                ),
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _nameController,
                decoration: InputDecoration(
                  labelText: "الاسم",
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(10),
                    borderSide: BorderSide(color: Color(int.parse(colors['primary']!, radix: 16))),
                  ),
                ),
                style: const TextStyle(fontFamily: 'Cairo'),
                width: 400,
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _ageController,
                decoration: InputDecoration(
                  labelText: "العمر",
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(10),
                    borderSide: BorderSide(color: Color(int.parse(colors['primary']!, radix: 16))),
                  ),
                ),
                style: const TextStyle(fontFamily: 'Cairo'),
                width: 400,
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _dateController,
                decoration: InputDecoration(
                  labelText: "تاريخ الاعتقال",
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(10),
                    borderSide: BorderSide(color: Color(int.parse(colors['primary']!, radix: 16))),
                  ),
                ),
                style: const TextStyle(fontFamily: 'Cairo'),
                width: 400,
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _locationController,
                decoration: InputDecoration(
                  labelText: "مكان الاعتقال",
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(10),
                    borderSide: BorderSide(color: Color(int.parse(colors['primary']!, radix: 16))),
                  ),
                ),
                style: const TextStyle(fontFamily: 'Cairo'),
                width: 400,
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _prisonController,
                decoration: InputDecoration(
                  labelText: "اسم السجن",
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(10),
                    borderSide: BorderSide(color: Color(int.parse(colors['primary']!, radix: 16))),
                  ),
                ),
                style: const TextStyle(fontFamily: 'Cairo'),
                width: 400,
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _detailsController,
                decoration: InputDecoration(
                  labelText: "تفاصيل",
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(10),
                    borderSide: BorderSide(color: Color(int.parse(colors['primary']!, radix: 16))),
                  ),
                  hintText: "أدخل تفاصيل عن الأسير...",
                ),
                maxLines: 3,
                style: const TextStyle(fontFamily: 'Cairo'),
                width: 400,
              ),
              const SizedBox(height: 30),
              Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  ElevatedButton(
                    onPressed: () async {
                      if (_nameController.text.isEmpty || _ageController.text.isEmpty || _dateController.text.isEmpty) {
                        Helper.showSnackBar(context, "يرجى ملء جميع الحقول المطلوبة");
                        return;
                      }
                      await widget.db.insert(
                        'prisoners',
                        {
                          'name': _nameController.text,
                          'age': int.tryParse(_ageController.text) ?? 0,
                          'arrest_date': _dateController.text,
                          'arrest_location': _locationController.text,
                          'prison_name': _prisonController.text,
                          'details': _detailsController.text,
                        },
                      );
                      Helper.showSnackBar(context, "تم حفظ بيانات الأسير بنجاح", Color(int.parse(colors['primary']!, radix: 16)));
                      Navigator.pop(context);
                    },
                    style: buttonStyle,
                    child: const Text("حفظ", style: TextStyle(fontFamily: 'Cairo')),
                  ),
                  const SizedBox(width: 20),
                  ElevatedButton(
                    onPressed: () {
                      Navigator.pop(context);
                    },
                    style: secondaryButtonStyle,
                    child: const Text("عودة", style: TextStyle(fontFamily: 'Cairo')),
                  ),
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }

  @override
  void dispose() {
    _nameController.dispose();
    _ageController.dispose();
    _dateController.dispose();
    _locationController.dispose();
    _prisonController.dispose();
    _detailsController.dispose();
    super.dispose();
  }
}

class MartyrsListScreen extends StatefulWidget {
  final Database db;

  const MartyrsListScreen({super.key, required this.db});

  @override
  State<MartyrsListScreen> createState() => _MartyrsListScreenState();
}

class _MartyrsListScreenState extends State<MartyrsListScreen> {
  late List<Map<String, dynamic>> _martyrs;

  @override
  void initState() {
    super.initState();
    _loadMartyrs();
  }

  Future<void> _loadMartyrs() async {
    final martyrs = await Helper.queryTable(widget.db, 'martyrs');
    setState(() {
      _martyrs = martyrs;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Color(int.parse(colors['background']!, radix: 16)),
      body: Center(
        child: Container(
          padding: const EdgeInsets.all(50),
          width: 600,
          decoration: BoxDecoration(
            color: Color(int.parse(colors['surface']!, radix: 16)),
            borderRadius: BorderRadius.circular(20),
            boxShadow: [
              BoxShadow(
                spreadRadius: 1,
                blurRadius: 15,
                color: Color(int.parse(colors['primary_dark']!, radix: 16)),
                offset: const Offset(0, 0),
                blurStyle: BlurStyle.outer,
              ),
            ],
          ),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.center,
            children: [
              const Text(
                "قائمة الشهداء",
                style: TextStyle(
                  fontSize: 25,
                  fontWeight: FontWeight.bold,
                  color: Color(0xFF1A237E),
                ),
              ),
              const SizedBox(height: 20),
              Container(
                height: 400,
                width: 600,
                decoration: BoxDecoration(
                  border: Border.all(color: Color(int.parse(colors['primary']!, radix: 16))),
                  borderRadius: BorderRadius.circular(10),
                ),
                child: ListView.builder(
                  itemCount: _martyrs.length,
                  itemBuilder: (context, index) {
                    final martyr = _martyrs[index];
                    return Container(
                      padding: const EdgeInsets.all(15),
                      margin: const EdgeInsets.only(bottom: 10),
                      decoration: BoxDecoration(
                        color: Color(int.parse(colors['surface']!, radix: 16)),
                        border: Border.all(color: Color(int.parse(colors['primary_light']!, radix: 16))),
                        borderRadius: BorderRadius.circular(10),
                      ),
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          Text("الاسم: ${martyr['name']}", style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                          Text("العمر: ${martyr['age']}", style: const TextStyle(fontSize: 16)),
                          Text("تاريخ الاستشهاد: ${martyr['date_of_martyrdom']}", style: const TextStyle(fontSize: 16)),
                          Text("المكان: ${martyr['location']}", style: const TextStyle(fontSize: 16)),
                          Text("التفاصيل: ${martyr['details']}", style: const TextStyle(fontSize: 16)),
                        ],
                      ),
                    );
                  },
                ),
              ),
              const SizedBox(height: 20),
              ElevatedButton(
                onPressed: () {
                  Navigator.pop(context);
                },
                style: secondaryButtonStyle,
                child: const Text("عودة", style: TextStyle(fontFamily: 'Cairo')),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

class WoundedListScreen extends StatefulWidget {
  final Database db;

  const WoundedListScreen({super.key, required this.db});

  @override
  State<WoundedListScreen> createState() => _WoundedListScreenState();
}

class _WoundedListScreenState extends State<WoundedListScreen> {
  late List<Map<String, dynamic>> _wounded;

  @override
  void initState() {
    super.initState();
    _loadWounded();
  }

  Future<void> _loadWounded() async {
    final wounded = await Helper.queryTable(widget.db, 'wounded');
    setState(() {
      _wounded = wounded;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Color(int.parse(colors['background']!, radix: 16)),
      body: Center(
        child: Container(
          padding: const EdgeInsets.all(50),
          width: 600,
          decoration: BoxDecoration(
            color: Color(int.parse(colors['surface']!, radix: 16)),
            borderRadius: BorderRadius.circular(20),
            boxShadow: [
              BoxShadow(
                spreadRadius: 1,
                blurRadius: 15,
                color: Color(int.parse(colors['primary_dark']!, radix: 16)),
                offset: const Offset(0, 0),
                blurStyle: BlurStyle.outer,
              ),
            ],
          ),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.center,
            children: [
              const Text(
                "قائمة الجرحى",
                style: TextStyle(
                  fontSize: 25,
                  fontWeight: FontWeight.bold,
                  color: Color(0xFF1A237E),
                ),
              ),
              const SizedBox(height: 20),
              Container(
                height: 400,
                width: 600,
                decoration: BoxDecoration(
                  border: Border.all(color: Color(int.parse(colors['primary']!, radix: 16))),
                  borderRadius: BorderRadius.circular(10),
                ),
                child: ListView.builder(
                  itemCount: _wounded.length,
                  itemBuilder: (context, index) {
                    final wounded = _wounded[index];
                    return Container(
                      padding: const EdgeInsets.all(15),
                      margin: const EdgeInsets.only(bottom: 10),
                      decoration: BoxDecoration(
                        color: Color(int.parse(colors['surface']!, radix: 16)),
                        border: Border.all(color: Color(int.parse(colors['primary_light']!, radix: 16))),
                        borderRadius: BorderRadius.circular(10),
                      ),
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          Text("الاسم: ${wounded['name']}", style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                          Text("العمر: ${wounded['age']}", style: const TextStyle(fontSize: 16)),
                          Text("تاريخ الإصابة: ${wounded['injury_date']}", style: const TextStyle(fontSize: 16)),
                          Text("مكان الإصابة: ${wounded['injury_location']}", style: const TextStyle(fontSize: 16)),
                          Text("تفاصيل الإصابة: ${wounded['injury_details']}", style: const TextStyle(fontSize: 16)),
                          Text("الحالة الطبية: ${wounded['medical_status']}", style: const TextStyle(fontSize: 16)),
                        ],
                      ),
                    );
                  },
                ),
              ),
              const SizedBox(height: 20),
              ElevatedButton(
                onPressed: () {
                  Navigator.pop(context);
                },
                style: secondaryButtonStyle,
                child: const Text("عودة", style: TextStyle(fontFamily: 'Cairo')),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

class PrisonersListScreen extends StatefulWidget {
  final Database db;

  const PrisonersListScreen({super.key, required this.db});

  @override
  State<PrisonersListScreen> createState() => _PrisonersListScreenState();
}

class _PrisonersListScreenState extends State<PrisonersListScreen> {
  late List<Map<String, dynamic>> _prisoners;

  @override
  void initState() {
    super.initState();
    _loadPrisoners();
  }

  Future<void> _loadPrisoners() async {
    final prisoners = await Helper.queryTable(widget.db, 'prisoners');
    setState(() {
      _prisoners = prisoners;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Color(int.parse(colors['background']!, radix: 16)),
      body: Center(
        child: Container(
          padding: const EdgeInsets.all(50),
          width: 600,
          decoration: BoxDecoration(
            color: Color(int.parse(colors['surface']!, radix: 16)),
            borderRadius: BorderRadius.circular(20),
            boxShadow: [
              BoxShadow(
                spreadRadius: 1,
                blurRadius: 15,
                color: Color(int.parse(colors['primary_dark']!, radix: 16)),
                offset: const Offset(0, 0),
                blurStyle: BlurStyle.outer,
              ),
            ],
          ),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.center,
            children: [
              const Text(
                "قائمة الأسرى",
                style: TextStyle(
                  fontSize: 25,
                  fontWeight: FontWeight.bold,
                  color: Color(0xFF1A237E),
                ),
              ),
              const SizedBox(height: 20),
              Container(
                height: 400,
                width: 600,
                decoration: BoxDecoration(
                  border: Border.all(color: Color(int.parse(colors['primary']!, radix: 16))),
                  borderRadius: BorderRadius.circular(10),
                ),
                child: ListView.builder(
                  itemCount: _prisoners.length,
                  itemBuilder: (context, index) {
                    final prisoner = _prisoners[index];
                    return Container(
                      padding: const EdgeInsets.all(15),
                      margin: const EdgeInsets.only(bottom: 10),
                      decoration: BoxDecoration(
                        color: Color(int.parse(colors['surface']!, radix: 16)),
                        border: Border.all(color: Color(int.parse(colors['primary_light']!, radix: 16))),
                        borderRadius: BorderRadius.circular(10),
                      ),
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          Text("الاسم: ${prisoner['name']}", style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                          Text("العمر: ${prisoner['age']}", style: const TextStyle(fontSize: 16)),
                          Text("تاريخ الاعتقال: ${prisoner['arrest_date']}", style: const TextStyle(fontSize: 16)),
                          Text("مكان الاعتقال: ${prisoner['arrest_location']}", style: const TextStyle(fontSize: 16)),
                          Text("اسم السجن: ${prisoner['prison_name']}", style: const TextStyle(fontSize: 16)),
                          Text("التفاصيل: ${prisoner['details']}", style: const TextStyle(fontSize: 16)),
                        ],
                      ),
                    );
                  },
                ),
              ),
              const SizedBox(height: 20),
              ElevatedButton(
                onPressed: () {
                  Navigator.pop(context);
                },
                style: secondaryButtonStyle,
                child: const Text("عودة", style: TextStyle(fontFamily: 'Cairo')),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

class SettingsScreen extends StatefulWidget {
  final Database db;

  const SettingsScreen({super.key, required this.db});

  @override
  State<SettingsScreen> createState() => _SettingsScreenState();
}

class _SettingsScreenState extends State<SettingsScreen> {
  bool _isDarkMode = false;

  @override
  void initState() {
    super.initState();
    _isDarkMode = false; // في Flutter لا ندعم التبديل في هذا الإصدار
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Color(int.parse(colors['background']!, radix: 16)),
      body: Center(
        child: Container(
          padding: const EdgeInsets.all(50),
          width: 500,
          decoration: BoxDecoration(
            color: Color(int.parse(colors['surface']!, radix: 16)),
            borderRadius: BorderRadius.circular(20),
            boxShadow: [
              BoxShadow(
                spreadRadius: 1,
                blurRadius: 15,
                color: Color(int.parse(colors['primary_dark']!, radix: 16)),
                offset: const Offset(0, 0),
                blurStyle: BlurStyle.outer,
              ),
            ],
          ),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.center,
            children: [
              const Text(
                "الإعدادات",
                style: TextStyle(
                  fontSize: 25,
                  fontWeight: FontWeight.bold,
                  color: Color(0xFF1A237E),
                ),
              ),
              const SizedBox(height: 20),
              const Text(
                "الوضع الليلي غير مدعوم في هذا الإصدار.",
                style: TextStyle(fontSize: 16),
              ),
              const SizedBox(height: 30),
              ElevatedButton(
                onPressed: () {
                  Navigator.pop(context);
                },
                style: secondaryButtonStyle,
                child: const Text("عودة", style: TextStyle(fontFamily: 'Cairo')),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

class UsersListScreen extends StatefulWidget {
  final Database db;

  const UsersListScreen({super.key, required this.db});

  @override
  State<UsersListScreen> createState() => _UsersListScreenState();
}

class _UsersListScreenState extends State<UsersListScreen> {
  late List<Map<String, dynamic>> _users;

  @override
  void initState() {
    super.initState();
    _loadUsers();
  }

  Future<void> _loadUsers() async {
    final users = await Helper.queryTable(widget.db, 'users');
    setState(() {
      _users = users;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Color(int.parse(colors['background']!, radix: 16)),
      body: Center(
        child: Container(
          padding: const EdgeInsets.all(50),
          width: 600,
          decoration: BoxDecoration(
            color: Color(int.parse(colors['surface']!, radix: 16)),
            borderRadius: BorderRadius.circular(20),
            boxShadow: [
              BoxShadow(
                spreadRadius: 1,
                blurRadius: 15,
                color: Color(int.parse(colors['primary_dark']!, radix: 16)),
                offset: const Offset(0, 0),
                blurStyle: BlurStyle.outer,
              ),
            ],
          ),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.center,
            children: [
              const Text(
                "قائمة المستخدمين",
                style: TextStyle(
                  fontSize: 25,
                  fontWeight: FontWeight.bold,
                  color: Color(0xFF1A237E),
                ),
              ),
              const SizedBox(height: 20),
              Container(
                height: 400,
                width: 600,
                decoration: BoxDecoration(
                  border: Border.all(color: Color(int.parse(colors['primary']!, radix: 16))),
                  borderRadius: BorderRadius.circular(10),
                ),
                child: ListView.builder(
                  itemCount: _users.length,
                  itemBuilder: (context, index) {
                    final user = _users[index];
                    return Container(
                      padding: const EdgeInsets.all(15),
                      margin: const EdgeInsets.only(bottom: 10),
                      decoration: BoxDecoration(
                        color: Color(int.parse(colors['surface']!, radix: 16)),
                        border: Border.all(color: Color(int.parse(colors['primary_light']!, radix: 16))),
                        borderRadius: BorderRadius.circular(10),
                      ),
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          Text("اسم المستخدم: ${user['username']}", style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                          Text("البريد الإلكتروني: ${user['email']}", style: const TextStyle(fontSize: 16)),
                          Text("صلاحية: ${user['is_admin'] == 1 ? 'مدير' : 'مستخدم عادي'}", style: const TextStyle(fontSize: 16)),
                        ],
                      ),
                    );
                  },
                ),
              ),
              const SizedBox(height: 20),
              ElevatedButton(
                onPressed: () {
                  Navigator.pop(context);
                },
                style: secondaryButtonStyle,
                child: const Text("عودة", style: TextStyle(fontFamily: 'Cairo')),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
