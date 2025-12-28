/*
Full Flutter + Firebase Starter Project: Daily Work MM App
- Job list
- Job detail
- Post job
- Firebase CRUD
- AdMob (optional)
*/

// pubspec.yaml (partial)
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^3.30.0
  cloud_firestore: ^4.10.0
  firebase_auth: ^4.6.0

flutter:
  assets:
    - assets/data/jobs.json

// lib/main.dart
import 'package:flutter/material.dart';
import 'screens/home_screen.dart';
import 'package:firebase_core/firebase_core.dart';
import 'firebase_options.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  runApp(DailyWorkApp());
}

class DailyWorkApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Daily Work MM',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: HomeScreen(),
    );
  }
}

// lib/models/job.dart
class Job {
  String title;
  String city;
  String salary;
  String type;
  String contact;
  String description;

  Job({required this.title, required this.city, required this.salary, required this.type, required this.contact, required this.description});

  factory Job.fromJson(Map<String, dynamic> json) => Job(
    title: json['title'],
    city: json['city'],
    salary: json['salary'],
    type: json['type'],
    contact: json['contact'],
    description: json['description'],
  );

  Map<String, dynamic> toJson() => {
    'title': title,
    'city': city,
    'salary': salary,
    'type': type,
    'contact': contact,
    'description': description,
  };
}

// lib/services/firebase_service.dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/job.dart';

class FirebaseService {
  final CollectionReference jobs = FirebaseFirestore.instance.collection('jobs');

  Future<void> addJob(Job job) async => await jobs.add(job.toJson());

  Stream<List<Job>> getJobs() => jobs.snapshots().map((snapshot) =>
      snapshot.docs.map((doc) => Job.fromJson(doc.data() as Map<String, dynamic>)).toList());
}

// lib/screens/home_screen.dart
import 'package:flutter/material.dart';
import '../models/job.dart';
import '../services/firebase_service.dart';
import 'job_detail_screen.dart';
import 'post_job_screen.dart';

class HomeScreen extends StatelessWidget {
  final FirebaseService firebaseService = FirebaseService();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Daily Work MM')),
      body: StreamBuilder<List<Job>>(
        stream: firebaseService.getJobs(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
          final jobs = snapshot.data!;
          return ListView.builder(
            itemCount: jobs.length,
            itemBuilder: (context, index) {
              final job = jobs[index];
              return ListTile(
                title: Text(job.title),
                subtitle: Text('${job.city} - ${job.salary}'),
                onTap: () => Navigator.push(
                  context,
                  MaterialPageRoute(builder: (_) => JobDetailScreen(job: job)),
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () => Navigator.push(context, MaterialPageRoute(builder: (_) => PostJobScreen())),
      ),
    );
  }
}

// lib/screens/job_detail_screen.dart
import 'package:flutter/material.dart';
import '../models/job.dart';

class JobDetailScreen extends StatelessWidget {
  final Job job;
  JobDetailScreen({required this.job});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(job.title)),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text('City: ${job.city}'),
            Text('Salary: ${job.salary}'),
            Text('Type: ${job.type}'),
            Text('Contact: ${job.contact}'),
            SizedBox(height: 10),
            Text('Description:'),
            Text(job.description),
          ],
        ),
      ),
    );
  }
}

// lib/screens/post_job_screen.dart
import 'package:flutter/material.dart';
import '../models/job.dart';
import '../services/firebase_service.dart';

class PostJobScreen extends StatefulWidget {
  @override
  _PostJobScreenState createState() => _PostJobScreenState();
}

class _PostJobScreenState extends State<PostJobScreen> {
  final _formKey = GlobalKey<FormState>();
  final _titleController = TextEditingController();
  final _cityController = TextEditingController();
  final _salaryController = TextEditingController();
  final _typeController = TextEditingController();
  final _contactController = TextEditingController();
  final _descriptionController = TextEditingController();

  final FirebaseService _firebaseService = FirebaseService();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Post Job')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Form(
          key: _formKey,
          child: ListView(
            children: [
              TextFormField(controller: _titleController, decoration: InputDecoration(labelText: 'Title')),
              TextFormField(controller: _cityController, decoration: InputDecoration(labelText: 'City')),
              TextFormField(controller: _salaryController, decoration: InputDecoration(labelText: 'Salary')),
              TextFormField(controller: _typeController, decoration: InputDecoration(labelText: 'Type')),
              TextFormField(controller: _contactController, decoration: InputDecoration(labelText: 'Contact')),
              TextFormField(controller: _descriptionController, decoration: InputDecoration(labelText: 'Description')),
              SizedBox(height: 20),
              ElevatedButton(
                child: Text('Post Job'),
                onPressed: () {
                  final job = Job(
                    title: _titleController.text,
                    city: _cityController.text,
                    salary: _salaryController.text,
                    type: _typeController.text,
                    contact: _contactController.text,
                    description: _descriptionController.text,
                  );
                  _firebaseService.addJob(job);
                  Navigator.pop(context);
                },
              )
            ],
          ),
        ),
      ),
    );
  }
}


