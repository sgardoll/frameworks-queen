# Research Brief Generator App

Implementation plan for a Flutter app that orchestrates AI research and content creation. Features a clean, modern UI that guides users through the content creation process in a joyful, effortless way.

## App Structure

```
lib/
├── main.dart
├── models/
│   ├── project.dart
│   ├── research_brief.dart
│   ├── content_outline.dart
│   └── graphics_plan.dart
├── services/
│   ├── ai_orchestrator.dart
│   ├── perplexity_service.dart
│   ├── openai_service.dart
│   └── claude_service.dart
├── screens/
│   ├── home_screen.dart
│   ├── project_setup_screen.dart
│   ├── research_screen.dart
│   ├── synthesis_screen.dart
│   ├── approach_screen.dart
│   ├── outline_screen.dart
│   ├── script_screen.dart
│   └── graphics_screen.dart
├── widgets/
│   ├── step_card.dart
│   ├── provider_selector.dart
│   └── citation_display.dart
└── utils/
    └── constants.dart
```

Let's start with the main files:

## `pubspec.yaml`
```yaml
name: research_brief_generator
description: A full-stack application that generates comprehensive research briefs for content creation.

publish_to: 'none'

version: 1.0.0+1

environment:
  sdk: '>=3.0.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.2
  provider: ^6.1.1
  http: ^0.17.0
  flutter_markdown: ^0.6.18
  url_launcher: ^6.2.5
  shared_preferences: ^2.2.2
  path_provider: ^2.1.2
  archive: ^3.4.10

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.0

flutter:
  uses-material-design: true
  assets:
    - assets/images/
```

## `lib/main.dart`
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'screens/home_screen.dart';
import 'models/project.dart';

void main() {
  runApp(
    ChangeNotifierProvider(
      create: (context) => ProjectModel(),
      child: const MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Research Brief Generator',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        visualDensity: VisualDensity.adaptivePlatformDensity,
        fontFamily: 'Roboto',
      ),
      home: const HomeScreen(),
      debugShowCheckedModeBanner: false,
    );
  }
}
```

## `lib/models/project.dart`
```dart
import 'package:flutter/foundation.dart';
import 'research_brief.dart';
import 'content_outline.dart';
import 'graphics_plan.dart';

class ProjectModel extends ChangeNotifier {
  String _topic = '';
  String _audience = '';
  String _purpose = '';
  String _constraints = '';
  
  ResearchBrief? _researchBrief;
  String? _recommendedApproach;
  ContentOutline? _outline;
  String? _script;
  GraphicsPlan? _graphicsPlan;
  
  // Getters
  String get topic => _topic;
  String get audience => _audience;
  String get purpose => _purpose;
  String get constraints => _constraints;
  ResearchBrief? get researchBrief => _researchBrief;
  String? get recommendedApproach => _recommendedApproach;
  ContentOutline? get outline => _outline;
  String? get script => _script;
  GraphicsPlan? get graphicsPlan => _graphicsPlan;
  
  // Setters
  void setTopic(String topic) {
    _topic = topic;
    notifyListeners();
  }
  
  void setAudience(String audience) {
    _audience = audience;
    notifyListeners();
  }
  
  void setPurpose(String purpose) {
    _purpose = purpose;
    notifyListeners();
  }
  
  void setConstraints(String constraints) {
    _constraints = constraints;
    notifyListeners();
  }
  
  void setResearchBrief(ResearchBrief brief) {
    _researchBrief = brief;
    notifyListeners();
  }
  
  void setRecommendedApproach(String approach) {
    _recommendedApproach = approach;
    notifyListeners();
  }
  
  void setOutline(ContentOutline outline) {
    _outline = outline;
    notifyListeners();
  }
  
  void setScript(String script) {
    _script = script;
    notifyListeners();
  }
  
  void setGraphicsPlan(GraphicsPlan plan) {
    _graphicsPlan = plan;
    notifyListeners();
  }
  
  void resetProject() {
    _topic = '';
    _audience = '';
    _purpose = '';
    _constraints = '';
    _researchBrief = null;
    _recommendedApproach = null;
    _outline = null;
    _script = null;
    _graphicsPlan = null;
    notifyListeners();
  }
}
```

## `lib/models/research_brief.dart`
```dart
class ResearchBrief {
  final List<String> keyQuestions;
  final String summary;
  final List<Claim> claims;
  final List<String> counterpoints;
  final List<String> risks;
  final List<Source> sources;
  final List<String> missingButImportant;

  ResearchBrief({
    required this.keyQuestions,
    required this.summary,
    required this.claims,
    required this.counterpoints,
    required this.risks,
    required this.sources,
    required this.missingButImportant,
  });
}

class Claim {
  final String text;
  final List<String> citations;

  Claim({required this.text, required this.citations});
}

class Source {
  final String title;
  final String url;
  final int qualityScore; // 1-5

  Source({
    required this.title,
    required this.url,
    required this.qualityScore,
  });
}
```

## `lib/models/content_outline.dart`
```dart
class ContentOutline {
  final String title;
  final List<Section> sections;
  final int totalRuntimeMinutes;

  ContentOutline({
    required this.title,
    required this.sections,
    required this.totalRuntimeMinutes,
  });
}

class Section {
  final String title;
  final String goal;
  final List<String> keyPoints;
  final String visualIdea;
  final List<String> evidenceToCite;
  final int estimatedTimeMinutes;

  Section({
    required this.title,
    required this.goal,
    required this.keyPoints,
    required this.visualIdea,
    required this.evidenceToCite,
    required this.estimatedTimeMinutes,
  });
}
```

## `lib/models/graphics_plan.dart`
```dart
class GraphicsPlan {
  final List<LowerThird> lowerThirds;
  final List<Diagram> diagrams;
  final List<Chart> charts;
  final List<ThumbnailConcept> thumbnailConcepts;

  GraphicsPlan({
    required this.lowerThirds,
    required this.diagrams,
    required this.charts,
    required this.thumbnailConcepts,
  });
}

class LowerThird {
  final String purpose;
  final String text;
  final String styleCues;
  final String generationPrompt;
  final String aspectRatio;

  LowerThird({
    required this.purpose,
    required this.text,
    required this.styleCues,
    required this.generationPrompt,
    required this.aspectRatio,
  });
}

class Diagram {
  final String purpose;
  final String type;
  final String description;
  final String generationPrompt;
  final String aspectRatio;

  Diagram({
    required this.purpose,
    required this.type,
    required this.description,
    required this.generationPrompt,
    required this.aspectRatio,
  });
}

class Chart {
  final String purpose;
  final String type;
  final String dataDescription;
  final String generationPrompt;
  final String aspectRatio;

  Chart({
    required this.purpose,
    required this.type,
    required this.dataDescription,
    required this.generationPrompt,
    required this.aspectRatio,
  });
}

class ThumbnailConcept {
  final String concept;
  final String description;
  final String styleCues;
  final String generationPrompt;
  final String aspectRatio;

  ThumbnailConcept({
    required this.concept,
    required this.description,
    required this.styleCues,
    required this.generationPrompt,
    required this.aspectRatio,
  });
}
```

## `lib/screens/home_screen.dart`
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'project_setup_screen.dart';
import '../models/project.dart';

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Research Brief Generator'),
        backgroundColor: Colors.blue[800],
        foregroundColor: Colors.white,
        elevation: 0,
      ),
      body: Container(
        decoration: BoxDecoration(
          gradient: LinearGradient(
            begin: Alignment.topCenter,
            end: Alignment.bottomCenter,
            colors: [
              Colors.blue[50]!,
              Colors.white,
            ],
          ),
        ),
        child: Padding(
          padding: const EdgeInsets.all(24.0),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              // App logo
              Container(
                width: 120,
                height: 120,
                decoration: BoxDecoration(
                  color: Colors.blue[100],
                  borderRadius: BorderRadius.circular(20),
                  boxShadow: [
                    BoxShadow(
                      color: Colors.blue.withOpacity(0.2),
                      blurRadius
