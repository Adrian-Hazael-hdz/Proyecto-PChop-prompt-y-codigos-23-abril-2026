¡Hola! Entendido. Como creador de software, he diseñado este plan de trabajo estructurado para que tus estudiantes no solo aprendan a programar, sino a orquestar un proyecto profesional utilizando **Flutter**, **Firebase** y la metodología de **Antigravity**.

---

## 1. Configuración de Entorno y Proyecto
Primero, establecemos la jerarquía de carpetas solicitada.

* **Comandos en terminal:**
    ```bash
    mkdir xflutterhernandez1128
    cd xflutterhernandez1128
    flutter create crudpchop
    cd crudpchop
    ```

---

## 2. Metodología Antigravity: Agentes y Roles
En este proyecto, implementaremos **Antigravity** como un enfoque de arquitectura limpia donde cada "Agente" (clase/módulo) tiene un rol y habilidades específicas.

### Estructura de Trabajo (Agentes)
| Agente | Rol | Skill (Habilidad) |
| :--- | :--- | :--- |
| **Data Agent** | Repositorio | Comunicación directa con Firestore. |
| **Logic Agent** | Controlador | Gestión del estado y validación de datos. |
| **UI Agent** | Vista | Renderizado de widgets con colores atractivos. |

---

## 3. Configuración de Firebase Firestore
1.  Ve a [Firebase Console](https://console.firebase.google.com/).
2.  Crea un proyecto llamado `crudpchop`.
3.  En el menú lateral, selecciona **Firestore Database** y haz clic en **Crear base de datos**.
4.  Crea una colección llamada `computadoras`.
5.  Añade un documento de prueba con los campos: `nombre` (string), `precio` (number) y `stock` (number).

---

## 4 y 5. Integración de Librerías (pubspec.yaml)
Para que Firebase funcione, debemos añadir las dependencias. El archivo `pubspec.yaml` debe quedar así:

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^3.0.0      # Agente de conexión principal
  cloud_firestore: ^5.0.0    # Agente de base de datos
  cupertino_icons: ^1.0.6
```
*Para instalar, ejecuta `flutter pub get` en la terminal.*

---

## 6 y 8. Implementación del CRUD: Código Funcional
Aquí tienes la estructura de archivos propuesta bajo la lógica de **Flujo de Trabajo Antigravity**.

### Estructura de Carpetas
```text
crudpchop/
├── lib/
│   ├── agents/
│   │   ├── data_agent.dart    (Conexión Firestore)
│   │   └── logic_agent.dart   (Lógica de negocio)
│   ├── ui/
│   │   └── home_screen.dart   (Interfaz Visual)
│   └── main.dart              (Punto de entrada)
```

#### A. Agente de Datos (`lib/agents/data_agent.dart`)
Este agente se encarga exclusivamente de las operaciones CRUD.

```dart
import 'package:cloud_firestore/cloud_firestore.dart';

class DataAgent {
  final CollectionReference _collection = 
      FirebaseFirestore.instance.collection('computadoras');

  // CREATE
  Future<void> addPC(String n, double p, int s) => 
      _collection.add({'nombre': n, 'precio': p, 'stock': s});

  // READ (Stream)
  Stream<QuerySnapshot> getPCStream() => _collection.snapshots();

  // UPDATE
  Future<void> updatePC(String id, String n, double p, int s) =>
      _collection.doc(id).update({'nombre': n, 'precio': p, 'stock': s});

  // DELETE
  Future<void> deletePC(String id) => _collection.doc(id).delete();
}
```

#### B. Agente de Interfaz (`lib/ui/home_screen.dart`)
Utilizaremos colores vibrantes (Indigo y Amber) para una UI atractiva.

```dart
import 'package:flutter/material.dart';
import '../agents/data_agent.dart';

class HomeScreen extends StatelessWidget {
  final DataAgent _agent = DataAgent();
  final TextEditingController _nameCtrl = TextEditingController();
  final TextEditingController _priceCtrl = TextEditingController();
  final TextEditingController _stockCtrl = TextEditingController();

  void _showForm(BuildContext context, {String? id}) {
    showModalBottomSheet(
      context: context,
      isScrollControlled: true,
      backgroundColor: Colors.grey[50],
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.vertical(top: Radius.circular(25))),
      builder: (_) => Padding(
        padding: EdgeInsets.only(top: 20, left: 20, right: 20, bottom: MediaQuery.of(context).viewInsets.bottom + 20),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextField(controller: _nameCtrl, decoration: InputDecoration(labelText: 'Nombre PC', prefixIcon: Icon(Icons.computer, color: Colors.indigo))),
            TextField(controller: _priceCtrl, decoration: InputDecoration(labelText: 'Precio', prefixIcon: Icon(Icons.attach_money, color: Colors.green)), keyboardType: TextInputType.number),
            TextField(controller: _stockCtrl, decoration: InputDecoration(labelText: 'Stock', prefixIcon: Icon(Icons.inventory, color: Colors.amber)), keyboardType: TextInputType.number),
            SizedBox(height: 20),
            ElevatedButton(
              style: ElevatedButton.styleFrom(backgroundColor: Colors.indigo, foregroundColor: Colors.white),
              onPressed: () {
                if (id == null) {
                  _agent.addPC(_nameCtrl.text, double.parse(_priceCtrl.text), int.parse(_stockCtrl.text));
                } else {
                  _agent.updatePC(id, _nameCtrl.text, double.parse(_priceCtrl.text), int.parse(_stockCtrl.text));
                }
                Navigator.pop(context);
              },
              child: Text(id == null ? 'Agregar Computadora' : 'Actualizar Datos'),
            )
          ],
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('CRUD PCHOP - Antigravity'), backgroundColor: Colors.indigo),
      body: StreamBuilder(
        stream: _agent.getPCStream(),
        builder: (context, AsyncSnapshot<QuerySnapshot> snapshot) {
          if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
          return ListView(
            children: snapshot.data!.docs.map((doc) {
              return Card(
                elevation: 4, margin: EdgeInsets.all(8),
                child: ListTile(
                  leading: CircleAvatar(backgroundColor: Colors.amber, child: Icon(Icons.laptop, color: Colors.black)),
                  title: Text(doc['nombre'], style: TextStyle(fontWeight: FontWeight.bold)),
                  subtitle: Text('Precio: \$${doc['precio']} | Stock: ${doc['stock']}'),
                  trailing: Row(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      IconButton(icon: Icon(Icons.edit, color: Colors.blue), onPressed: () => _showForm(context, id: doc.id)),
                      IconButton(icon: Icon(Icons.delete, color: Colors.red), onPressed: () => _agent.deletePC(doc.id)),
                    ],
                  ),
                ),
              );
            }).toList(),
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        backgroundColor: Colors.indigo,
        onPressed: () => _showForm(context),
        child: Icon(Icons.add, color: Colors.white),
      ),
    );
  }
}
```

---

## 7. Guía Práctica para Estudiantes (Profundización)

Para que tus alumnos comprendan el **Flujo Antigravity**, pídeles que visualicen el sistema como una cadena de mando:

1.  **Activación:** El usuario presiona un botón (UI Agent).
2.  **Solicitud:** La UI no borra datos sola; le pide al **Data Agent** que realice la acción.
3.  **Respuesta:** Firestore cambia, el Stream detecta el cambio y el Agente de UI se actualiza automáticamente.

> **Reto para el estudiante:** Agreguen un nuevo campo llamado "Marca" en Firestore y modifiquen el `DataAgent` y el formulario en la UI para soportar este nuevo dato.



### Inicialización Final (`lib/main.dart`)
```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'ui/home_screen.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(); // Inicialización de Firebase
  runApp(MaterialApp(
    debugShowCheckedModeBanner: false,
    theme: ThemeData(primarySwatch: Colors.indigo),
    home: HomeScreen(),
  ));
}
```
