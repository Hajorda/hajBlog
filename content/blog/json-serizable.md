---
external: false
title: "Embrace the Lazy Way: A Fun Dive into json_serializable in Flutter"
description: "json_serializable is a code generation library that automates the process of converting Dart objects to JSON and back again."
date: 2025-02-02
---
![jsonBanner](/images/jsonBanner.png "Banner")

Flutter development often involves lots of JSON wrangling—fetching it from APIs, converting it into Dart objects, sending it back, and all that jazz. Sure, you could write those boilerplate `fromJson` and `toJson` methods yourself. But why bother when the `json_serializable` package can do the heavy lifting for you? Let’s dig into this magical library and how it can save you time and headaches, with some playful examples to boot.

---

## What is json_serializable?

`json_serializable` is a code generation library that automates the process of converting Dart objects to JSON and back again. It takes care of all the nitty-gritty details while you focus on building your app—or sipping coffee.

With `json_serializable`, you’ll:

- Avoid writing repetitive serialization code.
- Handle edge cases like null values or unknown types easily.
- Keep your codebase clean and maintainable.

---

## Getting Started

Here’s how to set up `json_serializable` in your Flutter project and start using it like a pro.

### Step 1: Add Dependencies

First, include the necessary packages in your `pubspec.yaml` file:

And, there is the packages:
- [json_serializable](https://pub.dev/packages/json_serializable):
- [json_annotation](https://pub.dev/packages/json_annotation)
- [build_runner](https://pub.dev/packages/build_runner/install)

```yaml
dependencies:
  json_annotation: ^4.9.0

dev_dependencies:
  build_runner: ^2.4.14
  json_serializable: ^6.9.3
```

Run `flutter pub get` to fetch the dependencies.

---

### Step 2: Annotate Your Model

Let’s create a `CoffeeShop` class—because who doesn’t love coffee?

```dart
import 'package:json_annotation/json_annotation.dart';

part 'coffee_shop.g.dart';

@JsonSerializable()
class CoffeeShop {
  final String name;
  final String location;
  final double rating;
  final List<String> specialties;

  CoffeeShop({
    required this.name,
    required this.location,
    required this.rating,
    required this.specialties,
  });

  factory CoffeeShop.fromJson(Map<String, dynamic> json) => _$CoffeeShopFromJson(json);

  Map<String, dynamic> toJson() => _$CoffeeShopToJson(this);
}
```

A few things to note here:

- `@JsonSerializable()` tells the code generator to create serialization logic for this class.
- The `part` directive links the generated file.
- The `fromJson` and `toJson` methods will delegate the hard work to the generated code.

---

### Step 3: Generate the Code

Now for the fun part. Run the following command in your terminal:

```bash
flutter pub run build_runner build
```

This generates the `coffee_shop.g.dart` file, which contains the serialization code.

Here’s a peek at what’s inside:

```dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'coffee_shop.dart';

CoffeeShop _$CoffeeShopFromJson(Map<String, dynamic> json) {
  return CoffeeShop(
    name: json['name'] as String,
    location: json['location'] as String,
    rating: (json['rating'] as num).toDouble(),
    specialties: (json['specialties'] as List<dynamic>).map((e) => e as String).toList(),
  );
}

Map<String, dynamic> _$CoffeeShopToJson(CoffeeShop instance) => <String, dynamic>{
      'name': instance.name,
      'location': instance.location,
      'rating': instance.rating,
      'specialties': instance.specialties,
    };
```

Boom. All that boilerplate is taken care of for you.

---

## Going the Extra Mile

### Handling Null Values

What if your API sometimes returns null? Use the `@JsonKey` annotation to set default values:

```dart
@JsonSerializable()
class CoffeeShop {
  @JsonKey(defaultValue: 'Unnamed Coffee Shop')
  final String name;
  
  @JsonKey(defaultValue: 'Unknown Location')
  final String location;
  
  @JsonKey(defaultValue: 0.0)
  final double rating;

  @JsonKey(defaultValue: [])
  final List<String> specialties;

  CoffeeShop({
    required this.name,
    required this.location,
    required this.rating,
    required this.specialties,
  });

  factory CoffeeShop.fromJson(Map<String, dynamic> json) => _$CoffeeShopFromJson(json);

  Map<String, dynamic> toJson() => _$CoffeeShopToJson(this);
}
```

Now your app won’t break if some fields are missing.

---

### Custom Serialization Logic

Need to convert a timestamp to a `DateTime`? No problem. Here’s how:

```dart
@JsonSerializable()
class CoffeeShop {
  final String name;
  final String location;

  @JsonKey(fromJson: _fromTimestamp, toJson: _toTimestamp)
  final DateTime established;

  CoffeeShop({
    required this.name,
    required this.location,
    required this.established,
  });

  factory CoffeeShop.fromJson(Map<String, dynamic> json) => _$CoffeeShopFromJson(json);

  Map<String, dynamic> toJson() => _$CoffeeShopToJson(this);

  static DateTime _fromTimestamp(int timestamp) => DateTime.fromMillisecondsSinceEpoch(timestamp);

  static int _toTimestamp(DateTime date) => date.millisecondsSinceEpoch;
}
```

Now your dates are seamlessly converted.

---

### Handling Different Variable Names in Code and Database

Sometimes, the variable name in your Dart code might be different from the one in your database. You can use `@JsonKey(name: 'database_field_name')` to map them correctly:

```dart
@JsonSerializable()
class CoffeeShop {
  @JsonKey(name: 'shop_name')
  final String name;
  
  @JsonKey(name: 'shop_location')
  final String location;
  
  @JsonKey(name: 'avg_rating')
  final double rating;

  CoffeeShop({
    required this.name,
    required this.location,
    required this.rating,
  });

  factory CoffeeShop.fromJson(Map<String, dynamic> json) => _$CoffeeShopFromJson(json);

  Map<String, dynamic> toJson() => _$CoffeeShopToJson(this);
}
```


---

## Wrapping Up

`json_serializable` is a must-have for Flutter developers looking to streamline their JSON parsing. It eliminates boilerplate, handles edge cases, and keeps your codebase tidy.
Why not give it a spin? You’ll have more time for the important things—like finding the best coffee shop in town. Happy coding!