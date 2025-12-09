# List Wrapper Feature Implementation

## Summary
Successfully implemented a new checkbox feature in the JSON Input Dialog that allows users to generate a list wrapper class for their entities.

## Changes Made

### 1. Package Configuration (`package.json`)
- Added new configuration property `flutter-json-bean-factory.generateList` (boolean, default: false)
- This setting controls whether to generate a list wrapper class

### 2. UI Dialog (`src/ui/JsonInputDialog.ts`)
- Added `generateList?: boolean` to `JsonInputSettings` interface
- Added a new checkbox labeled "list" in the dialog UI
- Implemented event handler for the list checkbox
- The checkbox state is saved/loaded from VS Code settings
- The checkbox value is passed to the generator when creating the class

### 3. JSON Parser (`src/parsers/JsonParser.ts`)
- Modified `parseJson()` method to check for `config.generateList`
- When `generateList` is true:
  - Creates a wrapper class with name `{ClassName}List`
  - Adds a `list` property of type `List<{ClassName}>`
  - Sets default value to `const []`
  - The original entity class becomes a nested class
  - Returns the wrapper class instead of the original class

### 4. Code Generator (`src/generators/DartCodeGenerator.ts`)
- Fixed `collectAllClasses()` to prevent duplicate classes in the output
- Updated `getDefaultValue()` to return `const []` for arrays (required for const constructors)
- Updated `getDefaultValue()` to return `const ClassName()` for nested objects

## Example Output

### Input JSON:
```json
{
  "bankCode": "",
  "bankFullName": "",
  "bankName": "",
  "bankRegion": 0,
  "icon": "",
  "id": 0
}
```

### Without "list" checkbox (default):
```dart
@JsonSerializable()
class BankEntity {
  final String bankCode;
  final String bankFullName;
  final String bankName;
  final int bankRegion;
  final String icon;
  final int id;

  const BankEntity({
    this.bankCode = '',
    this.bankFullName = '',
    this.bankName = '',
    this.bankRegion = 0,
    this.icon = '',
    this.id = 0,
  });

  factory BankEntity.fromJson(Map<String, dynamic> json) => $BankEntityFromJson(json);
  Map<String, dynamic> toJson() => $BankEntityToJson(this);

  @override
  String toString() {
    return jsonEncode(this);
  }
}
```

### With "list" checkbox checked:
```dart
@JsonSerializable()
class BankEntityList {
  final List<BankEntity> list;

  const BankEntityList({
    this.list = const [],
  });

  factory BankEntityList.fromJson(Map<String, dynamic> json) => $BankEntityListFromJson(json);
  Map<String, dynamic> toJson() => $BankEntityListToJson(this);

  @override
  String toString() {
    return jsonEncode(this);
  }
}

@JsonSerializable()
class BankEntity {
  final String bankCode;
  final String bankFullName;
  final String bankName;
  final int bankRegion;
  final String icon;
  final int id;

  const BankEntity({
    this.bankCode = '',
    this.bankFullName = '',
    this.bankName = '',
    this.bankRegion = 0,
    this.icon = '',
    this.id = 0,
  });

  factory BankEntity.fromJson(Map<String, dynamic> json) => $BankEntityFromJson(json);
  Map<String, dynamic> toJson() => $BankEntityToJson(this);

  @override
  String toString() {
    return jsonEncode(this);
  }
}
```

## Usage
1. Open the JSON Input Dialog (Cmd+Alt+J on Mac)
2. Enter your class name (e.g., "BankEntity")
3. Paste your JSON
4. Check the "list" checkbox if you want a list wrapper class
5. Click "Generate"

## Technical Notes
- The wrapper class uses `const []` as the default value for the list property
- Both the wrapper class and the entity class use const constructors
- The feature integrates seamlessly with existing nullable and default value settings
- All generated helper files (.g.dart) and json_convert_content.dart are properly updated
