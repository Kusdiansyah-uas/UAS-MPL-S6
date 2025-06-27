# E-Commerce Flutter App

## Deskripsi Proyek
Proyek ini adalah aplikasi e-commerce berbasis Flutter yang menyediakan fitur login, katalog produk, detail produk, dan keranjang belanja. Data produk diambil dari API eksternal menggunakan HTTP request. Aplikasi ini dirancang dengan arsitektur berbasis widget yang modular untuk mempermudah pengelolaan kode.

---

## Struktur Proyek

```plaintext
lib/
|-- main.dart
|-- screens/
|   |-- login.dart
|-- pages/
|   |-- home.dart
|   |-- detail_page.dart
|-- network/
|   |-- api.dart
|-- data/
|   |-- fetch_data.dart
|-- models/
|   |-- product_model.dart
```

---

## Penjelasan File

### 1. **main.dart**
File ini adalah entry point aplikasi Flutter. Aplikasi dimulai dari kelas `MyApp` yang merujuk ke halaman login sebagai halaman utama.

#### Kode:
```dart
import 'package:flutter/material.dart';
import '/screens/login.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      title: 'E-Commerce App',
      home: LoginPage(),
    );
  }
}
```

---

### 2. **login.dart**
File ini berisi logika dan antarmuka untuk login pengguna. Login memanfaatkan API yang dikirimkan melalui metode POST.

#### Kode:
```dart
import 'package:flutter/material.dart';
import '/network/api.dart';
import '../screens/home.dart';

class LoginPage extends StatefulWidget {
  const LoginPage({super.key});

  @override
  _LoginPageState createState() => _LoginPageState();
}

class _LoginPageState extends State<LoginPage> {
  final TextEditingController _usernameController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();
  bool _isLoading = false;

  void _login() async {
    setState(() => _isLoading = true);

    final api = Api();
    final success = await api.login(
      _usernameController.text,
      _passwordController.text,
    );

    setState(() => _isLoading = false);

    if (success) {
      Navigator.pushReplacement(
        context,
        MaterialPageRoute(builder: (_) => Home()),
      );
    } else {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('Login Gagal.')),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Login'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            TextField(
              controller: _usernameController,
              decoration: InputDecoration(
                hintText: 'Username',
                filled: true,
              ),
            ),
            const SizedBox(height: 16),
            TextField(
              controller: _passwordController,
              obscureText: true,
              decoration: InputDecoration(
                hintText: 'Password',
                filled: true,
              ),
            ),
            const SizedBox(height: 24),
            ElevatedButton(
              onPressed: _isLoading ? null : _login,
              child: _isLoading
                  ? const CircularProgressIndicator()
                  : const Text('Login'),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

### 3. **home.dart**
File ini menampilkan katalog produk dalam bentuk grid menggunakan `FutureBuilder` untuk menampilkan data produk dari API.

#### Kode:
```dart
import 'package:flutter/material.dart';
import '../data/fetch_data.dart';
import '../models/product_model.dart';
import 'detail_page.dart';

class Home extends StatefulWidget {
  @override
  _HomeState createState() => _HomeState();
}

class _HomeState extends State<Home> {
  late Future<List<Product>> futureProducts;

  @override
  void initState() {
    super.initState();
    futureProducts = FetchData().fetchProducts();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text("Katalog Produk"),
      ),
      body: FutureBuilder<List<Product>>(
        future: futureProducts,
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          } else if (snapshot.hasError) {
            return Center(child: Text("Error: ${snapshot.error}"));
          } else {
            final products = snapshot.data!;
            return GridView.builder(
              gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                crossAxisCount: 2,
              ),
              itemCount: products.length,
              itemBuilder: (context, index) {
                final product = products[index];
                return GestureDetector(
                  onTap: () {
                    Navigator.push(
                      context,
                      MaterialPageRoute(
                        builder: (context) => DetailPage(product: product),
                      ),
                    );
                  },
                  child: Card(
                    child: Column(
                      children: [
                        Image.network(product.thumbnail),
                        Text(product.title),
                      ],
                    ),
                  ),
                );
              },
            );
          }
        },
      ),
    );
  }
}
```

---

### 4. **detail_page.dart**
File ini menampilkan detail produk dan menyediakan tombol untuk menambahkannya ke keranjang belanja.

#### Kode:
```dart
import 'package:flutter/material.dart';
import '../models/product_model.dart';

class DetailPage extends StatelessWidget {
  final Product product;

  const DetailPage({Key? key, required this.product}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(product.title),
      ),
      body: Column(
        children: [
          Image.network(product.thumbnail),
          Text(product.title),
          Text('\$${product.price}'),
          ElevatedButton(
            onPressed: () {},
            child: const Text('Add to Cart'),
          ),
        ],
      ),
    );
  }
}
```

---

### 5. **api.dart**
File ini berisi fungsi untuk mengelola komunikasi dengan API, termasuk login.

#### Kode:
```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class Api {
  final String baseUrl = 'https://dummyjson.com/auth/login';

  Future<bool> login(String username, String password) async {
    final response = await http.post(
      Uri.parse(baseUrl),
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode({'username': username, 'password': password}),
    );

    if (response.statusCode == 200) {
      return true;
    } else {
      return false;
    }
  }
}
```

---

### 6. **fetch_data.dart**
File ini digunakan untuk mengambil data produk dari API.

#### Kode:
```dart
import 'dart:convert';
import 'package:http/http.dart' as http;
import '../models/product_model.dart';

class FetchData {
  Future<List<Product>> fetchProducts() async {
    final response =
        await http.get(Uri.parse("https://dummyjson.com/products"));
    if (response.statusCode == 200) {
      final List<dynamic> data = json.decode(response.body)['products'];
      return data.map((json) => Product.fromJson(json)).toList();
    } else {
      throw Exception("Failed to load products");
    }
  }
}
```

---

### 7. **product_model.dart**
Model ini merepresentasikan struktur data produk.

#### Kode:
```dart
class Product {
  final int id;
  final String title;
  final String description;
  final double price;
  final String thumbnail;

  Product({
    required this.id,
    required this.title,
    required this.description,
    required this.price,
    required this.thumbnail,
  });

  factory Product.fromJson(Map<String, dynamic> json) {
    return Product(
      id: json['id'],
      title: json['title'],
      description: json['description'],
      price: (json['price'] as num).toDouble(),
      thumbnail: json['thumbnail'],
    );
  }
}
```

---

## Cara Menjalankan Proyek
1. Pastikan Flutter sudah terinstal di komputer Anda.
2. Clone repository ini.
3. Jalankan perintah `flutter pub get` untuk menginstal dependencies.
4. Jalankan aplikasi dengan perintah `flutter run`. 

---

## API yang Digunakan
- **Login:** `https://dummyjson.com/auth/login`
- **Produk:** `https://dummyjson.com/products`

---

## Data Diri
Nama: Kusdiansyah
NPM: 20122018
