

```dart

Map<String, dynamic> buildCaseSafePayload({
  required String vid,
  required String kycToken,
  required String pid,
  required int bankSwitch,
}) {
  final Map<String, dynamic> payload = {};

  void addAllCases(String key, dynamic value) {
    payload[key] = value;                 // camelCase
    payload[_capitalize(key)] = value;    // PascalCase
    payload[key.toUpperCase()] = value;   // UPPERCASE
  }

  addAllCases('vid', vid);
  addAllCases('kycToken', kycToken);
  addAllCases('pid', pid);
  addAllCases('bankSwitch', bankSwitch);

  return payload;
}

String _capitalize(String value) {
  if (value.isEmpty) return value;
  return value[0].toUpperCase() + value.substring(1);
}

```