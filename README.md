//# Student-Portal
//I am using dart language for developing a student portal. After finishing the features will be updated.
import 'package:flutter/material.dart';

// --- MOCK DATA ---
// Data structures translated from JavaScript to Dart.
final Map<String, dynamic> mockData = {
  'students': {
    'S001': {
      'id': '242-35-176',
      'name': 'Gazi Tasnim Hasan',
      'department': 'Software Engineering',
      'batch': '43',
    }, // Updated batch as per prompt
    'S002': {
      'id': 'S002',
      'name': 'Bob Williams',
      'department': 'Mechanical Engineering',
      'batch': '2024',
    },
  },
  'courses': {
    'S001': [
      {
        'id': 'CS101',
        'name': 'Introduction to Programming',
        'faculty': 'Dr. Smith',
      },
      {'id': 'MA202', 'name': 'Advanced Calculus', 'faculty': 'Dr. Davis'},
    ],
  },
  'fees': {
    'S001': {
      'total': 349571,
      'paid': 259571,
      'due': 45560,
    }, // Updated values as per prompt
    'S002': {'total': 4500, 'paid': 4500, 'due': 0},
  },
};

void main() {
  runApp(const StudentPortalApp());
}

class StudentPortalApp extends StatelessWidget {
  const StudentPortalApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Student Portal',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        fontFamily: 'Roboto',
        scaffoldBackgroundColor: const Color(0xFFECF0F1),
        appBarTheme: const AppBarTheme(
          backgroundColor: Color(0xFF2C3E50),
          elevation: 0,
        ),
      ),
      home: const AuthController(),
    );
  }
}

// --- CONTROLLER FOR AUTH AND VIEW MANAGEMENT ---
// This widget manages whether to show the Login or Portal screen.
class AuthController extends StatefulWidget {
  const AuthController({super.key});

  @override
  State<AuthController> createState() => _AuthControllerState();
}

class _AuthControllerState extends State<AuthController> {
  Map<String, dynamic>? _currentUser;
  String? _currentStudentKey; // Added to store the internal key (e.g., 'S001')

  void _login(String studentId) {
    Map<String, dynamic>? studentData;
    String? foundKey;

    mockData['students'].forEach((key, value) {
      if (value['id'] == studentId) {
        studentData = value;
        foundKey = key; // Capture the internal key
      }
    });

    if (studentData != null && foundKey != null) {
      setState(() {
        _currentUser = studentData;
        _currentStudentKey = foundKey; // Store the internal key
      });
    } else {
      // Show an error message (e.g., using a SnackBar)
      ScaffoldMessenger.of(
        context,
      ).showSnackBar(const SnackBar(content: Text('Invalid Student ID')));
    }
  }

  void _logout() {
    setState(() {
      _currentUser = null;
      _currentStudentKey = null; // Clear the internal key on logout
    });
  }

  @override
  Widget build(BuildContext context) {
    if (_currentUser == null || _currentStudentKey == null) {
      return LoginPage(onLogin: _login);
    } else {
      return PortalPage(
          user: _currentUser!,
          studentDataKey: _currentStudentKey!,
          onLogout: _logout);
    }
  }
}

// --- LOGIN PAGE WIDGET ---
class LoginPage extends StatefulWidget {
  final Function(String) onLogin;

  const LoginPage({super.key, required this.onLogin});

  @override
  State<LoginPage> createState() => _LoginPageState();
}

class _LoginPageState extends State<LoginPage> {
  final _studentIdController = TextEditingController();

  void _submit() {
    if (_studentIdController.text.isNotEmpty) {
      widget.onLogin(_studentIdController.text.trim());
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: SingleChildScrollView(
          padding: const EdgeInsets.all(40.0),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Image.network(
                'https://www.gstatic.com/flutter-onestack-prototype/genui/example_1.jpg',
                width: 300,
                errorBuilder: (context, error, stackTrace) =>
                    const Icon(Icons.school, size: 80),
              ),
              const SizedBox(height: 24),
              Text(
                'Student Portal Login',
                style: Theme.of(context).textTheme.headlineSmall,
              ),
              const SizedBox(height: 24),
              TextField(
                controller: _studentIdController,
                decoration: const InputDecoration(
                  labelText: 'Student ID',
                  border: OutlineInputBorder(),
                ),
              ),
              const SizedBox(height: 20),
              ElevatedButton(
                onPressed: _submit,
                style: ElevatedButton.styleFrom(
                  minimumSize: const Size(double.infinity, 48),
                ),
                child: const Text('Login'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

// --- MAIN PORTAL PAGE WIDGET ---
class PortalPage extends StatefulWidget {
  final Map<String, dynamic> user;
  final String studentDataKey; // New parameter to pass the internal student key
  final VoidCallback onLogout;

  const PortalPage({
    super.key,
    required this.user,
    required this.studentDataKey,
    required this.onLogout,
  });

  @override
  State<PortalPage> createState() => _PortalPageState();
}

class _PortalPageState extends State<PortalPage> {
  int _selectedIndex = 0; // 0: Profile, 1: Courses, 2: Fees

  // Helper to get the current page widget
  Widget _getSelectedPage() {
    switch (_selectedIndex) {
      case 0:
        return ProfileView(user: widget.user);
      case 1:
        // Pass the internal studentDataKey to CoursesView
        return CoursesView(studentDataKey: widget.studentDataKey);
      case 2:
        // Pass the internal studentDataKey to FeesView
        return FeesView(studentDataKey: widget.studentDataKey);
      default:
        return ProfileView(user: widget.user);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Student Portal'),
        actions: [
          IconButton(
            icon: const Icon(Icons.logout),
            onPressed: widget.onLogout,
            tooltip: 'Logout',
          ),
        ],
      ),
      drawer: Drawer(
        child: ListView(
          padding: EdgeInsets.zero,
          children: [
            UserAccountsDrawerHeader(
              accountName: Text(widget.user['name'] as String),
              accountEmail: Text('ID: ${widget.user['id'] as String}'),
              currentAccountPicture: CircleAvatar(
                child: Text((widget.user['name'] as String)[0]),
              ),
              decoration: const BoxDecoration(color: Color(0xFF2C3E50)),
            ),
            ListTile(
              leading: const Icon(Icons.person),
              title: const Text('Profile'),
              selected: _selectedIndex == 0,
              onTap: () {
                setState(() => _selectedIndex = 0);
                Navigator.pop(context); // Close the drawer
              },
            ),
            ListTile(
              leading: const Icon(Icons.book),
              title: const Text('Courses'),
              selected: _selectedIndex == 1,
              onTap: () {
                setState(() => _selectedIndex = 1);
                Navigator.pop(context);
              },
            ),
            ListTile(
              leading: const Icon(Icons.payment),
              title: const Text('Fees'),
              selected: _selectedIndex == 2,
              onTap: () {
                setState(() => _selectedIndex = 2);
                Navigator.pop(context);
              },
            ),
          ],
        ),
      ),
      body: _getSelectedPage(),
    );
  }
}

// --- INDIVIDUAL PAGE VIEWS ---

class ProfileView extends StatelessWidget {
  final Map<String, dynamic> user;
  const ProfileView({super.key, required this.user});

  @override
  Widget build(BuildContext context) {
    return SingleChildScrollView(
      padding: const EdgeInsets.all(16),
      child: Card(
        child: Padding(
          padding: const EdgeInsets.all(16.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text(
                'Student Information',
                style: Theme.of(context).textTheme.titleLarge,
              ),
              const Divider(),
              ListTile(
                  title: const Text('Name'),
                  subtitle: Text(user['name'] as String)),
              ListTile(
                title: const Text('Student ID'),
                subtitle: Text(user['id'] as String),
              ),
              ListTile(
                title: const Text('Department'),
                subtitle: Text(user['department'] as String),
              ),
              ListTile(
                title: const Text('Batch'),
                subtitle: Text(user['batch'] as String),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

class CoursesView extends StatelessWidget {
  final String studentDataKey; // Changed from studentId
  const CoursesView({super.key, required this.studentDataKey});

  @override
  Widget build(BuildContext context) {
    // Use studentDataKey to fetch courses
    final courses = mockData['courses'][studentDataKey] as List<dynamic>? ?? [];
    return Card(
      margin: const EdgeInsets.all(16),
      child: ListView.builder(
        itemCount: courses.isEmpty ? 1 : courses.length,
        itemBuilder: (context, index) {
          if (courses.isEmpty) {
            return const ListTile(title: Text('No courses registered.'));
          }
          final Map<String, dynamic> course =
              courses[index] as Map<String, dynamic>;
          return ListTile(
            title:
                Text('${course['id'] as String} - ${course['name'] as String}'),
            subtitle: Text('Faculty: ${course['faculty'] as String}'),
          );
        },
      ),
    );
  }
}

class FeesView extends StatelessWidget {
  final String studentDataKey; // Changed from studentId
  const FeesView({super.key, required this.studentDataKey});

  @override
  Widget build(BuildContext context) {
    // Use studentDataKey to fetch fees
    final fees = mockData['fees'][studentDataKey] as Map<String, dynamic>? ??
        {'total': 0, 'paid': 0, 'due': 0};
    return Card(
      margin: const EdgeInsets.all(16),
      child: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          mainAxisSize: MainAxisSize.min,
          children: [
            Text('Fee Details', style: Theme.of(context).textTheme.titleLarge),
            const Divider(),
            ListTile(
              title: const Text('Total Payable'),
              trailing: Text('${fees['total'] as int}'),
            ),
            ListTile(
              title: const Text('Amount Paid'),
              trailing: Text(
                '${fees['paid'] as int}',
                style: const TextStyle(color: Colors.green),
              ),
            ),
            ListTile(
              title: const Text('Amount Due'),
              trailing: Text(
                '${fees['due'] as int}',
                style: const TextStyle(color: Colors.red),
              ),
            ),
            const SizedBox(height: 20),
            Center(
              child: ElevatedButton(
                onPressed: () {
                  // Placeholder for payment gateway
                  ScaffoldMessenger.of(context).showSnackBar(
                    const SnackBar(
                      content: Text('Payment gateway integration is pending.'),
                    ),
                  );
                },
                child: const Text('Make Payment'),
              ),
            ),
          ],
        ),
      ),
    );
  }
}


//author Gazi tasnim hasan 
