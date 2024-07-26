# Tanning-routine-
Get UV index, personalized tanning tips, and track skin health daily. Pro version allows color customization.
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';
import 'package:image_picker/image_picker.dart';
import 'dart:io';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Tanning Routine App',
      theme: ThemeData(
        primarySwatch: Colors.pink,
        visualDensity: VisualDensity.adaptivePlatformDensity,
      ),
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget {
  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  final TextEditingController _locationController = TextEditingController();
  String _uvIndex = '';
  String _tanningRoutine = '';
  File? _image;

  Future<void> _getUVIndex() async {
    final response = await http.get(Uri.parse(
        'https://api.weatherapi.com/v1/current.json?key=YOUR_API_KEY&q=${_locationController.text}&aqi=no'));

    if (response.statusCode == 200) {
      final data = json.decode(response.body);
      setState(() {
        _uvIndex = data['current']['uv'].toString();
        _tanningRoutine = _calculateTanningRoutine(double.parse(_uvIndex));
      });
    } else {
      throw Exception('Failed to load UV index');
    }
  }

  String _calculateTanningRoutine(double uvIndex) {
    if (uvIndex < 3) return "Low UV Index: Safe for tanning!";
    if (uvIndex < 6) return "Moderate UV Index: Use SPF 15-30.";
    if (uvIndex < 8) return "High UV Index: Use SPF 30-50, limit sun exposure.";
    return "Very High UV Index: Use SPF 50+, avoid sun exposure.";
  }

  Future<void> _pickImage() async {
    final pickedFile = await ImagePicker().pickImage(source: ImageSource.camera);

    setState(() {
      if (pickedFile != null) {
        _image = File(pickedFile.path);
      } else {
        print('No image selected.');
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Tanning Routine App'),
      ),
      body: Padding(
        padding: EdgeInsets.all(16.0),
        child: Column(
          children: <Widget>[
            TextField(
              controller: _locationController,
              decoration: InputDecoration(labelText: 'Enter Location'),
            ),
            ElevatedButton(
              onPressed: _getUVIndex,
              child: Text('Get UV Index'),
            ),
            SizedBox(height: 20),
            Text(
              'UV Index: $_uvIndex',
              style: TextStyle(fontSize: 20),
            ),
            SizedBox(height: 20),
            Text(
              'Tanning Routine: $_tanningRoutine',
              style: TextStyle(fontSize: 20),
            ),
            SizedBox(height: 20),
            _image == null
                ? Text('No image selected.')
                : Image.file(_image!),
            ElevatedButton(
              onPressed: _pickImage,
              child: Text('Capture Skin Image'),
            ),
          ],
        ),
      ),
    );
  }
}
