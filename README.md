# Lab 06

## Description

This hands-on lab introduces students to Firebase Authentication by implementing user registration, login, and session management in Flutter. Students will build a complete authentication system supporting both email/password credentials and Google Sign-In (OAuth). By the end of this lab, students will have a working authentication flow that can be integrated into any Firebase-powered application.

## Learning Objectives

By completing this lab, students will be able to:

- Configure Firebase Authentication in the Firebase Console for multiple sign-in methods
- Implement email/password registration and login with Firebase Auth
- Handle authentication state changes using StreamBuilder and auth state listeners
- Integrate Google Sign-In for OAuth-based authentication
- Manage user sessions with automatic login persistence
- Handle authentication errors gracefully with user-friendly error messages
- Link authenticated users with Firestore data using unique user IDs (UIDs)

## Setup Instructions

### Part 0: Connect your project with Firebase

Follow the same initial steps Part 1 -> 3 in [Lab 5](https://github.com/ULHT-SME/lab5)

### Part 1: Enable Authentication Methods in Firebase Console

1. **Open Firebase Console:** Go to [console.firebase.google.com](https://console.firebase.google.com)
2. **Select Your Project:** Choose the project you're using for this lab
3. **Navigate to Authentication:**
   - Click "Authentication" in the left sidebar
   - Click "Get Started" if it's your first time
4. **Enable Email/Password:**
   - Click "Sign-in method" tab
   - Click "Email/Password"
   - Toggle "Enable" ON
   - Save changes
5. **Enable Google Sign-In:**
   - Still in "Sign-in method" tab
   - Click "Google"
   - Toggle "Enable" ON
   - Enter your project support email (required)
   - Save changes

### Part 2: Add Flutter Dependencies

Open terminal in the project directory and run this command:

```bash
flutter pub add firebase_core firebase_auth cloud_firestore google_sign_in
```

Run to install:

```bash
flutter pub get
```

### Part 3: Configure Google Sign-In (Platform-Specific)

#### Android Configuration

1. **Download SHA-1 fingerprint:**
```bash
cd android
./gradlew signingReport
```

2. **Copy the SHA-1 from the debug section**

3. **Add to Firebase:**
   - Firebase Console → Project Settings → Your Apps → Android app
   - Click "Add fingerprint"
   - Paste your SHA-1
   - Download new `google-services.json`
   - Replace the old one in `android/app/`

#### iOS Configuration (if testing on iOS)

1. **Download GoogleService-Info.plist** from Firebase Console
2. **Add to Xcode project** (open `ios/Runner.xcworkspace`)
3. **Get REVERSED_CLIENT_ID** from the plist file
4. **Add URL scheme:**
   - In Xcode, select Runner → Info → URL Types
   - Add new URL scheme with the REVERSED_CLIENT_ID value

### Part 4: Verify Setup

Create a simple test to verify Firebase Auth is initialized:

```dart
import 'package:firebase_auth/firebase_auth.dart';

void checkAuth() {
  final auth = FirebaseAuth.instance;
  print('Current user: ${auth.currentUser?.email ?? "No user"}');
}
```

---

## Lab Tasks

### Task 1: Create Authentication Service

**Objective:** Build a reusable authentication service to handle all auth operations.

**Your Challenge:**

Create a new file `lib/services/auth_service.dart`:

```dart
import 'package:firebase_auth/firebase_auth.dart';
import 'package:google_sign_in/google_sign_in.dart';

class AuthService {
  final FirebaseAuth _auth = FirebaseAuth.instance;
  final GoogleSignIn _googleSignIn = GoogleSignIn();

  // Get current user
  User? get currentUser => _auth.currentUser;

  // Auth state stream
  Stream<User?> get authStateChanges => _auth.authStateChanges();

  // Sign up with email and password
  Future<UserCredential?> signUpWithEmail(String email, String password) async {
    try {
      // TODO: Implement email registration
      // Hint: Use _auth.createUserWithEmailAndPassword()
      
    } on FirebaseAuthException catch (e) {
      // TODO: Handle specific errors (weak password, email in use, etc.)
      throw _handleAuthException(e);
    }
  }

  // Sign in with email and password
  Future<UserCredential?> signInWithEmail(String email, String password) async {
    try {
      // TODO: Implement email login
      // Hint: Use _auth.signInWithEmailAndPassword()
      
    } on FirebaseAuthException catch (e) {
      throw _handleAuthException(e);
    }
  }

  // Sign in with Google
  Future<UserCredential?> signInWithGoogle() async {
    try {
      // TODO: Trigger Google Sign-In flow
      // Hint: 
      // 1. await _googleSignIn.signIn()
      // 2. Get authentication from GoogleSignInAccount
      // 3. Create credential with GoogleAuthProvider.credential()
      // 4. Sign in with _auth.signInWithCredential()
      
    } catch (e) {
      throw Exception('Google Sign-In failed: $e');
    }
  }

  // Sign out
  Future<void> signOut() async {
    // TODO: Sign out from both Firebase and Google
    // Hint: await both _auth.signOut() and _googleSignIn.signOut()
  }

  // Handle auth exceptions
  String _handleAuthException(FirebaseAuthException e) {
    switch (e.code) {
      case 'weak-password':
        return 'The password is too weak.';
      case 'email-already-in-use':
        return 'An account already exists for this email.';
      case 'user-not-found':
        return 'No user found with this email.';
      case 'wrong-password':
        return 'Wrong password provided.';
      case 'invalid-email':
        return 'The email address is invalid.';
      default:
        return 'Authentication error: ${e.message}';
    }
  }
}
```

**Implementation Hints:**

For `signInWithGoogle`:
```dart
final GoogleSignInAccount? googleUser = await _googleSignIn.signIn();
if (googleUser == null) return null; // User cancelled

final GoogleSignInAuthentication googleAuth = await googleUser.authentication;

final credential = GoogleAuthProvider.credential(
  accessToken: googleAuth.accessToken,
  idToken: googleAuth.idToken,
);

return await _auth.signInWithCredential(credential);
```

### Task 2: Build Login Screen

**Objective:** Create a login UI with email/password and Google Sign-In options.

**Your Challenge:**

Create `lib/screens/login_screen.dart`:

```dart
import 'package:flutter/material.dart';
import '../services/auth_service.dart';

class LoginScreen extends StatefulWidget {
  @override
  _LoginScreenState createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  final _authService = AuthService();
  bool _isLoading = false;

  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  Future<void> _signInWithEmail() async {
    if (!_formKey.currentState!.validate()) return;

    setState(() => _isLoading = true);

    try {
      // TODO: Call _authService.signInWithEmail()
      // On success, user will automatically navigate due to auth state
      
    } catch (e) {
      // TODO: Show error dialog or SnackBar
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(e.toString())),
      );
    } finally {
      setState(() => _isLoading = false);
    }
  }

  Future<void> _signInWithGoogle() async {
    setState(() => _isLoading = true);

    try {
      // TODO: Call _authService.signInWithGoogle()
      
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(e.toString())),
      );
    } finally {
      setState(() => _isLoading = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: Center(
          child: SingleChildScrollView(
            padding: EdgeInsets.all(24),
            child: Form(
              key: _formKey,
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  // App Logo or Title
                  Icon(
                    Icons.lock_outline,
                    size: 80,
                    color: Theme.of(context).primaryColor,
                  ),
                  SizedBox(height: 16),
                  Text(
                    'Welcome Back',
                    style: TextStyle(
                      fontSize: 32,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  Text(
                    'Sign in to continue',
                    style: TextStyle(color: Colors.grey),
                  ),
                  SizedBox(height: 40),

                  // Email field
                  TextFormField(
                    controller: _emailController,
                    keyboardType: TextInputType.emailAddress,
                    decoration: InputDecoration(
                      labelText: 'Email',
                      border: OutlineInputBorder(),
                      prefixIcon: Icon(Icons.email),
                    ),
                    validator: (value) {
                      // TODO: Add email validation
                      if (value == null || value.isEmpty) {
                        return 'Please enter your email';
                      }
                      if (!value.contains('@')) {
                        return 'Please enter a valid email';
                      }
                      return null;
                    },
                  ),
                  SizedBox(height: 16),

                  // Password field
                  TextFormField(
                    controller: _passwordController,
                    obscureText: true,
                    decoration: InputDecoration(
                      labelText: 'Password',
                      border: OutlineInputBorder(),
                      prefixIcon: Icon(Icons.lock),
                    ),
                    validator: (value) {
                      // TODO: Add password validation
                      if (value == null || value.isEmpty) {
                        return 'Please enter your password';
                      }
                      if (value.length < 6) {
                        return 'Password must be at least 6 characters';
                      }
                      return null;
                    },
                  ),
                  SizedBox(height: 24),

                  // Sign in button
                  SizedBox(
                    width: double.infinity,
                    child: ElevatedButton(
                      onPressed: _isLoading ? null : _signInWithEmail,
                      style: ElevatedButton.styleFrom(
                        padding: EdgeInsets.symmetric(vertical: 16),
                      ),
                      child: _isLoading
                          ? CircularProgressIndicator(color: Colors.white)
                          : Text('Sign In', style: TextStyle(fontSize: 16)),
                    ),
                  ),
                  SizedBox(height: 16),

                  // Divider
                  Row(
                    children: [
                      Expanded(child: Divider()),
                      Padding(
                        padding: EdgeInsets.symmetric(horizontal: 16),
                        child: Text('OR'),
                      ),
                      Expanded(child: Divider()),
                    ],
                  ),
                  SizedBox(height: 16),

                  // Google Sign-In button
                  SizedBox(
                    width: double.infinity,
                    child: OutlinedButton.icon(
                      onPressed: _isLoading ? null : _signInWithGoogle,
                      icon: Image.asset('assets/google_logo.png', height: 24), // Add Google logo asset
                      label: Text('Sign in with Google'),
                      style: OutlinedButton.styleFrom(
                        padding: EdgeInsets.symmetric(vertical: 16),
                      ),
                    ),
                  ),
                  SizedBox(height: 16),

                  // Register link
                  Row(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Text("Don't have an account? "),
                      TextButton(
                        onPressed: () {
                          // TODO: Navigate to RegisterScreen
                          Navigator.pushNamed(context, '/register');
                        },
                        child: Text('Sign Up'),
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

**Note:** For Google Sign-In button, you can:
- Use a placeholder icon: `Icon(Icons.g_mobiledata)`
- Download Google logo PNG and add to `assets/` folder
- Use a package like `flutter_signin_button`

### Task 3: Build Registration Screen 

**Objective:** Create a registration form for new users.

**Your Challenge:**

Create `lib/screens/register_screen.dart`:

```dart
import 'package:flutter/material.dart';
import '../services/auth_service.dart';

class RegisterScreen extends StatefulWidget {
  @override
  _RegisterScreenState createState() => _RegisterScreenState();
}

class _RegisterScreenState extends State<RegisterScreen> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  final _confirmPasswordController = TextEditingController();
  final _authService = AuthService();
  bool _isLoading = false;

  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    _confirmPasswordController.dispose();
    super.dispose();
  }

  Future<void> _register() async {
    if (!_formKey.currentState!.validate()) return;

    // Check if passwords match
    if (_passwordController.text != _confirmPasswordController.text) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Passwords do not match')),
      );
      return;
    }

    setState(() => _isLoading = true);

    try {
      // TODO: Call _authService.signUpWithEmail()
      await _authService.signUpWithEmail(
        _emailController.text,
        _passwordController.text,
      );
      
      // Navigate back or to home (auth state will handle routing)
      Navigator.pop(context);
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(e.toString())),
      );
    } finally {
      setState(() => _isLoading = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Create Account'),
      ),
      body: SafeArea(
        child: Center(
          child: SingleChildScrollView(
            padding: EdgeInsets.all(24),
            child: Form(
              key: _formKey,
              child: Column(
                children: [
                  Icon(
                    Icons.person_add_outlined,
                    size: 80,
                    color: Theme.of(context).primaryColor,
                  ),
                  SizedBox(height: 16),
                  Text(
                    'Create Account',
                    style: TextStyle(fontSize: 28, fontWeight: FontWeight.bold),
                  ),
                  SizedBox(height: 32),

                  // Email field
                  TextFormField(
                    controller: _emailController,
                    keyboardType: TextInputType.emailAddress,
                    decoration: InputDecoration(
                      labelText: 'Email',
                      border: OutlineInputBorder(),
                      prefixIcon: Icon(Icons.email),
                    ),
                    validator: (value) {
                      if (value == null || value.isEmpty) {
                        return 'Please enter your email';
                      }
                      if (!value.contains('@')) {
                        return 'Please enter a valid email';
                      }
                      return null;
                    },
                  ),
                  SizedBox(height: 16),

                  // Password field
                  TextFormField(
                    controller: _passwordController,
                    obscureText: true,
                    decoration: InputDecoration(
                      labelText: 'Password',
                      border: OutlineInputBorder(),
                      prefixIcon: Icon(Icons.lock),
                    ),
                    validator: (value) {
                      if (value == null || value.isEmpty) {
                        return 'Please enter a password';
                      }
                      if (value.length < 6) {
                        return 'Password must be at least 6 characters';
                      }
                      return null;
                    },
                  ),
                  SizedBox(height: 16),

                  // Confirm password field
                  TextFormField(
                    controller: _confirmPasswordController,
                    obscureText: true,
                    decoration: InputDecoration(
                      labelText: 'Confirm Password',
                      border: OutlineInputBorder(),
                      prefixIcon: Icon(Icons.lock_outline),
                    ),
                    validator: (value) {
                      if (value == null || value.isEmpty) {
                        return 'Please confirm your password';
                      }
                      return null;
                    },
                  ),
                  SizedBox(height: 24),

                  // Register button
                  SizedBox(
                    width: double.infinity,
                    child: ElevatedButton(
                      onPressed: _isLoading ? null : _register,
                      style: ElevatedButton.styleFrom(
                        padding: EdgeInsets.symmetric(vertical: 16),
                      ),
                      child: _isLoading
                          ? CircularProgressIndicator(color: Colors.white)
                          : Text('Sign Up', style: TextStyle(fontSize: 16)),
                    ),
                  ),
                  SizedBox(height: 16),

                  // Login link
                  Row(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Text('Already have an account? '),
                      TextButton(
                        onPressed: () => Navigator.pop(context),
                        child: Text('Sign In'),
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

### Task 4: Create Home Screen with Auth State & Logout 

**Objective:** Build a home screen that displays user info and allows logout.

**Your Challenge:**

Create `lib/screens/home_screen.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:firebase_auth/firebase_auth.dart';
import '../services/auth_service.dart';

class HomeScreen extends StatelessWidget {
  final _authService = AuthService();

  @override
  Widget build(BuildContext context) {
    final user = _authService.currentUser;

    return Scaffold(
      appBar: AppBar(
        title: Text('Home'),
        actions: [
          IconButton(
            icon: Icon(Icons.logout),
            onPressed: () => _showLogoutDialog(context),
          ),
        ],
      ),
      body: Center(
        child: Padding(
          padding: EdgeInsets.all(24),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              // User avatar
              CircleAvatar(
                radius: 50,
                backgroundColor: Theme.of(context).primaryColor,
                backgroundImage: user?.photoURL != null
                    ? NetworkImage(user!.photoURL!)
                    : null,
                child: user?.photoURL == null
                    ? Icon(Icons.person, size: 50, color: Colors.white)
                    : null,
              ),
              SizedBox(height: 24),

              // Welcome message
              Text(
                'Welcome!',
                style: TextStyle(
                  fontSize: 28,
                  fontWeight: FontWeight.bold,
                ),
              ),
              SizedBox(height: 8),

              // User email
              Text(
                user?.email ?? 'No email',
                style: TextStyle(
                  fontSize: 18,
                  color: Colors.grey[600],
                ),
              ),
              SizedBox(height: 8),

              // User ID
              Text(
                'UID: ${user?.uid ?? "Unknown"}',
                style: TextStyle(
                  fontSize: 12,
                  color: Colors.grey,
                ),
              ),
              SizedBox(height: 32),

              // User info card
              Card(
                child: Padding(
                  padding: EdgeInsets.all(16),
                  child: Column(
                    children: [
                      _buildInfoRow(
                        'Email Verified',
                        user?.emailVerified == true ? 'Yes' : 'No',
                        user?.emailVerified == true
                            ? Icons.check_circle
                            : Icons.cancel,
                      ),
                      Divider(),
                      _buildInfoRow(
                        'Sign-in Method',
                        _getSignInMethod(user),
                        Icons.login,
                      ),
                      Divider(),
                      _buildInfoRow(
                        'Account Created',
                        _formatDate(user?.metadata.creationTime),
                        Icons.calendar_today,
                      ),
                    ],
                  ),
                ),
              ),
              SizedBox(height: 24),

              // Logout button
              ElevatedButton.icon(
                onPressed: () => _showLogoutDialog(context),
                icon: Icon(Icons.logout),
                label: Text('Sign Out'),
                style: ElevatedButton.styleFrom(
                  backgroundColor: Colors.red,
                  foregroundColor: Colors.white,
                  padding: EdgeInsets.symmetric(horizontal: 32, vertical: 12),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }

  Widget _buildInfoRow(String label, String value, IconData icon) {
    return Padding(
      padding: EdgeInsets.symmetric(vertical: 8),
      child: Row(
        children: [
          Icon(icon, size: 20, color: Colors.grey[600]),
          SizedBox(width: 12),
          Text(
            '$label: ',
            style: TextStyle(fontWeight: FontWeight.w500),
          ),
          Text(value),
        ],
      ),
    );
  }

  String _getSignInMethod(User? user) {
    if (user == null) return 'Unknown';
    
    for (var provider in user.providerData) {
      if (provider.providerId == 'google.com') return 'Google';
      if (provider.providerId == 'password') return 'Email/Password';
    }
    return 'Unknown';
  }

  String _formatDate(DateTime? date) {
    if (date == null) return 'Unknown';
    return '${date.month}/${date.day}/${date.year}';
  }

  void _showLogoutDialog(BuildContext context) {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('Sign Out'),
        content: Text('Are you sure you want to sign out?'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: Text('Cancel'),
          ),
          ElevatedButton(
            onPressed: () async {
              await _authService.signOut();
              Navigator.pop(context);
            },
            style: ElevatedButton.styleFrom(backgroundColor: Colors.red),
            child: Text('Sign Out', style: TextStyle(color: Colors.white)),
          ),
        ],
      ),
    );
  }
}
```

### Task 5: Wire Up Auth State with StreamBuilder 

**Objective:** Use StreamBuilder to automatically route users based on authentication state.

**Your Challenge:**

Update `lib/main.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'firebase_options.dart';
import 'services/auth_service.dart';
import 'screens/login_screen.dart';
import 'screens/register_screen.dart';
import 'screens/home_screen.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Firebase Auth Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        visualDensity: VisualDensity.adaptivePlatformDensity,
      ),
      home: AuthWrapper(),
      routes: {
        '/register': (context) => RegisterScreen(),
        '/home': (context) => HomeScreen(),
      },
    );
  }
}

class AuthWrapper extends StatelessWidget {
  final _authService = AuthService();

  @override
  Widget build(BuildContext context) {
    return StreamBuilder<User?>(
      stream: _authService.authStateChanges,
      builder: (context, snapshot) {
        // Show loading while checking auth state
        if (snapshot.connectionState == ConnectionState.waiting) {
          return Scaffold(
            body: Center(
              child: CircularProgressIndicator(),
            ),
          );
        }

        // If user is signed in, show Home
        if (snapshot.hasData) {
          return HomeScreen();
        }

        // Otherwise, show Login
        return LoginScreen();
      },
    );
  }
}
```

**Key Concept:**
- `authStateChanges` emits a new value whenever auth state changes
- StreamBuilder automatically rebuilds the UI
- No need to manually navigate after login - it happens automatically!

**Test Your App:**
- Run the app - you should see login screen
- Register a new account - automatically navigates to home
- Close and reopen app - you're still logged in (persistence!)
- Sign out - automatically returns to login screen

### Task 6: Challenge - Link User Data with Firestore 

**Objective:** Store additional user information in Firestore linked by UID.

**Your Challenge:**

1. When user registers, create a Firestore document in `users` collection
2. Use the user's UID as the document ID
3. Store additional user info (name, created date, etc.)

**Create User Profile Service:**

Create `lib/services/user_service.dart`:

```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:firebase_auth/firebase_auth.dart';

class UserService {
  final FirebaseFirestore _firestore = FirebaseFirestore.instance;

  // Create user profile in Firestore
  Future<void> createUserProfile(User user) async {
    try {
      await _firestore.collection('users').doc(user.uid).set({
        'email': user.email,
        'displayName': user.displayName ?? '',
        'photoURL': user.photoURL ?? '',
        'createdAt': FieldValue.serverTimestamp(),
      });
    } catch (e) {
      print('Error creating user profile: $e');
    }
  }

  // Get user profile
  Future<Map<String, dynamic>?> getUserProfile(String uid) async {
    // TODO ....
  }

  // Update user profile
  Future<void> updateUserProfile(String uid, Map<String, dynamic> data) async {
    // TODO ....
  }
}
```

**Update AuthService to create profile on registration:**

```dart
Future<UserCredential?> signUpWithEmail(String email, String password) async {
  try {
    final credential = await _auth.createUserWithEmailAndPassword(
      email: email,
      password: password,
    );
    
    // Create Firestore profile
    if (credential.user != null) {
      final userService = UserService();
      await userService.createUserProfile(credential.user!);
    }
    
    return credential;
  } on FirebaseAuthException catch (e) {
    throw _handleAuthException(e);
  }
}
```

**Display user profile data in HomeScreen:**

```dart
// In HomeScreen, replace static display with Firestore data
StreamBuilder<DocumentSnapshot>(
  stream: FirebaseFirestore.instance
      .collection('users')
      .doc(user?.uid)
      .snapshots(),
  builder: (context, snapshot) {
    if (snapshot.hasData) {
      final userData = snapshot.data!.data() as Map<String, dynamic>?;
      return Text(userData?['displayName'] ?? 'No name');
    }
    return CircularProgressIndicator();
  },
)
```

---

## Deliverables

You are required to .zip file this project and deliver it in Moodle before the next class. The .zip should contain all the content of inside project folder with the exception of the .dart_tool and build folders (these will make the .zip file extremely large and they are not necessary for evaluation).
