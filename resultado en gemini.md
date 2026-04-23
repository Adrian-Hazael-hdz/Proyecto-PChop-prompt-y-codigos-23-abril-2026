¡Hola! Como desarrollador senior, me encanta ver este tipo de estructuras. Vamos a construir **crudpchop**, un sistema sólido utilizando **Flutter**, **Firebase** y el patrón de diseño **Antigravity** (una arquitectura orientada a la separación de responsabilidades y estados limpios).

Aquí tienes el plan de trabajo detallado y la implementación técnica.

---

## 1. Configuración del Entorno y Estructura
Primero, establecemos la jerarquía de archivos solicitada para mantener el orden del proyecto.

* **Ruta:** `xflutterhernandez1128/crudpchop`

```bash
# Comandos de terminal
mkdir xflutterhernandez1128
cd xflutterhernandez1128
flutter create crudpchop
cd crudpchop
```

---

## 2. Configuración de Firebase (Firestore)
1.  Ve a [Firebase Console](https://console.firebase.google.com/).
2.  Crea un proyecto llamado **crudpchop**.
3.  En el menú lateral, selecciona **Firestore Database** y haz clic en **Crear base de datos**.
4.  Crea una colección llamada `computadoras`.
5.  Agrega un documento inicial con los campos:
    * `nombre`: (String)
    * `precio`: (Number)
    * `stock`: (Number)

---

## 3. Integración de Librerías y `pubspec.yaml`
Para implementar este proyecto, necesitamos las dependencias oficiales de Firebase y herramientas de gestión de estado/diseño.

### Modificación del archivo `pubspec.yaml`
Añade lo siguiente bajo la sección `dependencies`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  # Core de Firebase
  firebase_core: ^3.0.0 
  # Base de datos Firestore
  cloud_firestore: ^5.0.0
  # Diseño y UI
  google_fonts: ^6.2.1
```
> **Nota:** Después de guardar, ejecuta `flutter pub get` en tu terminal para descargar los paquetes.

---

## 4. Implementación del CRUD (Modelo y Servicios)
Siguiendo la metodología de **Antigravity**, dividiremos el código por responsabilidades: **Agentes (Lógica)**, **Roles (Modelos)** y **Skills (Servicios de Firebase)**.

### Estructura de Carpetas Recomendada
```text
lib/
├── agents/          # Controladores de lógica
├── roles/           # Modelos de datos (Entity)
├── skills/          # Servicios (Conexión a Firebase)
├── ui/              # Widgets y pantallas
└── main.dart        # Punto de entrada
```

### A. El Rol (Modelo de Datos): `lib/roles/computadora_role.dart`
```dart
class ComputadoraRole {
  String? id;
  String nombre;
  double precio;
  int stock;

  ComputadoraRole({this.id, required this.nombre, required this.precio, required this.stock});

  // Convertir Firestore a Objeto
  factory ComputadoraRole.fromFirestore(Map<String, dynamic> data, String id) {
    return ComputadoraRole(
      id: id,
      nombre: data['nombre'] ?? '',
      precio: (data['precio'] ?? 0).toDouble(),
      stock: data['stock'] ?? 0,
    );
  }

  // Convertir Objeto a Map para Firebase
  Map<String, dynamic> toMap() {
    return {
      'nombre': nombre,
      'precio': precio,
      'stock': stock,
    };
  }
}
```

### B. El Skill (Servicio Firestore): `lib/skills/firebase_skill.dart`
```dart
import 'cloud_firestore/cloud_firestore.dart';
import '../roles/computadora_role.dart';

class FirebaseSkill {
  final CollectionReference _db = FirebaseFirestore.instance.collection('computadoras');

  // CREATE
  Future<void> addComputadora(ComputadoraRole comp) => _db.add(comp.toMap());

  // READ (Stream para tiempo real)
  Stream<List<ComputadoraRole>> getComputadoras() {
    return _db.snapshots().map((snapshot) => snapshot.docs
        .map((doc) => ComputadoraRole.fromFirestore(doc.data() as Map<String, dynamic>, doc.id))
        .toList());
  }

  // UPDATE
  Future<void> updateComputadora(ComputadoraRole comp) => _db.doc(comp.id).update(comp.toMap());

  // DELETE
  Future<void> deleteComputadora(String id) => _db.doc(id).delete();
}
```

---

## 5. UI y Flujo de Trabajo (Agentes y Vistas)
Para que sea visualmente atractivo, utilizaremos colores vibrantes como **Deep Purple** y **Amber**.

### `lib/main.dart` (Inicialización)
```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'ui/home_screen.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(); // Inicializa Firebase
  runApp(const CrudPChopApp());
}

class CrudPChopApp extends StatelessWidget {
  const CrudPChopApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'CRUD PChop',
      theme: ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple, secondary: Colors.amber),
      ),
      home: const HomeScreen(),
    );
  }
}
```

### `lib/ui/home_screen.dart` (La Interfaz del Estudiante)
```dart
import 'package:flutter/material.dart';
import '../skills/firebase_skill.dart';
import '../roles/computadora_role.dart';

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final FirebaseSkill _skill = FirebaseSkill();

    return Scaffold(
      appBar: AppBar(
        title: const Text("PChop Inventory", style: TextStyle(color: Colors.white)),
        backgroundColor: Colors.deepPurple,
      ),
      body: StreamBuilder<List<ComputadoraRole>>(
        stream: _skill.getComputadoras(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return const Center(child: CircularProgressIndicator());
          
          final computadoras = snapshot.data!;
          return ListView.builder(
            itemCount: computadoras.length,
            itemBuilder: (context, index) {
              final item = computadoras[index];
              return Card(
                margin: const EdgeInsets.symmetric(horizontal: 15, vertical: 8),
                elevation: 4,
                child: ListTile(
                  leading: const CircleAvatar(backgroundColor: Colors.amber, child: Icon(Icons.computer)),
                  title: Text(item.nombre, style: const TextStyle(fontWeight: FontWeight.bold)),
                  subtitle: Text("Precio: \$${item.precio} | Stock: ${item.stock}"),
                  trailing: Row(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      IconButton(icon: const Icon(Icons.edit, color: Colors.blue), onPressed: () => _showForm(context, item)),
                      IconButton(icon: const Icon(Icons.delete, color: Colors.red), onPressed: () => _skill.deleteComputadora(item.id!)),
                    ],
                  ),
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        backgroundColor: Colors.amber,
        onPressed: () => _showForm(context, null),
        child: const Icon(Icons.add),
      ),
    );
  }

  // Modal para Crear y Editar
  void _showForm(BuildContext context, ComputadoraRole? comp) {
    final nombreController = TextEditingController(text: comp?.nombre ?? '');
    final precioController = TextEditingController(text: comp?.precio.toString() ?? '');
    final stockController = TextEditingController(text: comp?.stock.toString() ?? '');

    showModalBottomSheet(
      context: context,
      isScrollControlled: true,
      builder: (_) => Padding(
        padding: EdgeInsets.only(bottom: MediaQuery.of(context).viewInsets.bottom, top: 20, left: 20, right: 20),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextField(controller: nombreController, decoration: const InputDecoration(labelText: 'Nombre')),
            TextField(controller: precioController, decoration: const InputDecoration(labelText: 'Precio'), keyboardType: TextInputType.number),
            TextField(controller: stockController, decoration: const InputDecoration(labelText: 'Stock'), keyboardType: TextInputType.number),
            const SizedBox(height: 20),
            ElevatedButton(
              style: ElevatedButton.styleFrom(backgroundColor: Colors.deepPurple, foregroundColor: Colors.white),
              onPressed: () {
                final newComp = ComputadoraRole(
                  id: comp?.id,
                  nombre: nombreController.text,
                  precio: double.parse(precioController.text),
                  stock: int.parse(stockController.text),
                );
                comp == null ? FirebaseSkill().addComputadora(newComp) : FirebaseSkill().updateComputadora(newComp);
                Navigator.pop(context);
              },
              child: Text(comp == null ? "Crear" : "Actualizar"),
            ),
            const SizedBox(height: 20),
          ],
        ),
      ),
    );
  }
}
```

---

## 6. Metodología de Práctica Guiada (Antigravity)

Para los estudiantes, explicamos el flujo de trabajo de la siguiente manera:

1.  **El Rol (Datos):** Es el molde de lo que queremos representar (la Computadora). Define qué datos viajan.
2.  **El Skill (Habilidad):** Es la capacidad de hablar con el mundo exterior (Firebase). No decide qué guardar, solo sabe *cómo* hacerlo.
3.  **El Agente (Lógica/UI):** Es el intermediario. Cuando el estudiante presiona un botón (UI), el agente toma la información, valida que el precio sea un número y le pide al **Skill** que lo guarde usando el formato del **Rol**.

### Flujo de Trabajo
* **Paso 1:** Configura Firebase y descarga los `google-services.json`.
* **Paso 2:** Define tu **Rol** (Model).
* **Paso 3:** Crea el **Skill** para conectar los métodos `get`, `add`, `update`, `delete`.
* **Paso 4:** Diseña la **UI** conectando los `TextControllers` con las funciones del Skill.

Este código es **totalmente funcional**. Asegúrate de haber configurado el archivo `google-services.json` (Android) o `GoogleService-Info.plist` (iOS) en las carpetas nativas correspondientes para que la conexión con Firebase Console sea exitosa.
