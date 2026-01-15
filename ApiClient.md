
```dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';

import 'package:http/http.dart' as http;

import 'api_logger.dart';
import 'network_response.dart';

class ApiClient {
  static const Duration _timeout = Duration(seconds: 30);

  // ─────────────────────────────────────────────
  // STATUS CODE → MESSAGE
  // ─────────────────────────────────────────────
  static String _getErrorMessage(int statusCode) {
    switch (statusCode) {
      case 400:
        return 'Bad request';
      case 401:
        return 'Unauthorized access';
      case 403:
        return 'Access forbidden';
      case 404:
        return 'Resource not found';
      case 500:
        return 'Server error';
      case 503:
        return 'Service unavailable';
      default:
        return 'Unexpected error: $statusCode';
    }
  }

  // ─────────────────────────────────────────────
  // SAFE JSON DECODER
  // ─────────────────────────────────────────────
  static dynamic _safeJson(String text) {
    try {
      return jsonDecode(text);
    } catch (_) {
      return text;
    }
  }

  // ─────────────────────────────────────────────
  // ERROR CLASSIFIER
  // ─────────────────────────────────────────────
  static String _classifyNetworkError(Object error) {
    final msg = error.toString().toLowerCase();

    if (error is TimeoutException) return 'TIMEOUT';

    if (error is SocketException) {
      if (msg.contains('failed host lookup') ||
          msg.contains('name lookup failed')) {
        return 'DNS_FAIL';
      }
      return 'NO_INTERNET';
    }

    return 'UNKNOWN_NETWORK_ERROR';
  }

  // ─────────────────────────────────────────────
  // HUMAN READABLE ERROR (LOG + UI)
  // ─────────────────────────────────────────────
  static String _humanReadableError(String errorType) {
    switch (errorType) {
      case 'DNS_FAIL':
        return 'Unable to reach the server. Please check your network.';
      case 'TIMEOUT':
        return 'The request took too long. Please try again.';
      case 'NO_INTERNET':
        return 'No internet connection. Please check Wi-Fi or mobile data.';
      default:
        return 'A network error occurred. Please try again.';
    }
  }

  // ─────────────────────────────────────────────
  // CORE REQUEST EXECUTOR
  // ─────────────────────────────────────────────
  static Future<NetworkResponse> _executeRequest({
    required String method,
    required String url,
    required Map<String, String> headers,
    String? body,
  }) async {
    final stopwatch = Stopwatch()..start();
    final bool shouldLogBody =
        method.toUpperCase() != 'GET' &&
        body != null &&
        body.isNotEmpty;

    try {
      late http.Response response;

      switch (method.toUpperCase()) {
        case 'POST':
          response = await http
              .post(Uri.parse(url), headers: headers, body: body)
              .timeout(_timeout);
          break;

        case 'GET':
          response = await http
              .get(Uri.parse(url), headers: headers)
              .timeout(_timeout);
          break;

        case 'DELETE':
          response = await http
              .delete(Uri.parse(url), headers: headers, body: body)
              .timeout(_timeout);
          break;

        default:
          throw UnsupportedError('HTTP method not supported: $method');
      }

      stopwatch.stop();

      ApiLogger.logApiCall(
        method: method,
        url: url,
        headers: headers,
        requestBody: shouldLogBody ? _safeJson(body!) : null,
        statusCode: response.statusCode,
        responseBody: _safeJson(response.body),
        durationMs: stopwatch.elapsedMilliseconds,
      );

      if (response.statusCode >= 200 && response.statusCode < 300) {
        return NetworkResponse(
          status: true,
          data: response.body.isNotEmpty ? response.body : null,
        );
      }

      return NetworkResponse(
        status: false,
        data: response.body,
        errorMessage: _getErrorMessage(response.statusCode),
      );
    }

    // ⏱ TIMEOUT
    on TimeoutException catch (e) {
      stopwatch.stop();

      const errorType = 'TIMEOUT';

      ApiLogger.logApiCall(
        method: method,
        url: url,
        headers: headers,
        requestBody: shouldLogBody ? _safeJson(body!) : null,
        statusCode: -1,
        responseBody: e.toString(),
        durationMs: stopwatch.elapsedMilliseconds,
        errorType: errorType,
        errorMessage: _humanReadableError(errorType),
      );

      return NetworkResponse(
        status: false,
        errorMessage: _humanReadableError(errorType),
      );
    }

    // 🌐 SOCKET / DNS
    on SocketException catch (e) {
      stopwatch.stop();

      final errorType = _classifyNetworkError(e);

      ApiLogger.logApiCall(
        method: method,
        url: url,
        headers: headers,
        requestBody: shouldLogBody ? _safeJson(body!) : null,
        statusCode: -1,
        responseBody: e.toString(),
        durationMs: stopwatch.elapsedMilliseconds,
        errorType: errorType,
        errorMessage: _humanReadableError(errorType),
      );

      return NetworkResponse(
        status: false,
        errorMessage: _humanReadableError(errorType),
      );
    }

    // ❌ UNKNOWN
    catch (e) {
      stopwatch.stop();

      const errorType = 'UNKNOWN_NETWORK_ERROR';

      ApiLogger.logApiCall(
        method: method,
        url: url,
        headers: headers,
        requestBody: shouldLogBody ? _safeJson(body!) : null,
        statusCode: -1,
        responseBody: e.toString(),
        durationMs: stopwatch.elapsedMilliseconds,
        errorType: errorType,
        errorMessage: _humanReadableError(errorType),
      );

      return NetworkResponse(
        status: false,
        errorMessage: _humanReadableError(errorType),
      );
    }
  }

  // ─────────────────────────────────────────────
  // PUBLIC APIs
  // ─────────────────────────────────────────────
  static Future<NetworkResponse> post({
    required String url,
    required Map<String, String> headers,
    String? body,
  }) =>
      _executeRequest(
        method: 'POST',
        url: url,
        headers: headers,
        body: body,
      );

  static Future<NetworkResponse> get({
    required String url,
    required Map<String, String> headers,
  }) =>
      _executeRequest(
        method: 'GET',
        url: url,
        headers: headers,
      );

  static Future<NetworkResponse> delete({
    required String url,
    required Map<String, String> headers,
    String? body,
  }) =>
      _executeRequest(
        method: 'DELETE',
        url: url,
        headers: headers,
        body: body,
      );
}
```


```dart

import 'dart:convert';

import 'package:logger/logger.dart';

class ApiLogger {
  ApiLogger._(); // private constructor

  static final Logger _logger = Logger(
    printer: PrettyPrinter(
      methodCount: 1,
      errorMethodCount: 1,
      lineLength: 120,
      colors: true,
      printEmojis: true,
    ),
  );

  // ─────────────────────────────────────────────
  // PRETTY JSON
  // ─────────────────────────────────────────────
  static String _prettyJson(Object value) {
    try {
      return const JsonEncoder.withIndent('  ').convert(value);
    } catch (_) {
      return value.toString();
    }
  }

  // ─────────────────────────────────────────────
  // SERVER TIMESTAMP
  // ─────────────────────────────────────────────
  static String serverTimestamp(DateTime dt) {
    String two(int n) => n.toString().padLeft(2, '0');
    String three(int n) => n.toString().padLeft(3, '0');

    return '${dt.year}-'
        '${two(dt.month)}-'
        '${two(dt.day)} '
        '${two(dt.hour)}:'
        '${two(dt.minute)}:'
        '${two(dt.second)}.'
        '${three(dt.millisecond)}';
  }

  // ─────────────────────────────────────────────
  // MAIN API LOG
  // ─────────────────────────────────────────────
  static void logApiCall({
    required String method,
    required String url,
    required Map<String, String> headers,
    Object? requestBody,
    required int statusCode,
    required Object responseBody,
    required int durationMs,
    String? errorType,
    String? errorMessage,
  }) {
    final logObject = {
      "meta": {
        "timestamp": serverTimestamp(DateTime.now()),
        "durationMs": durationMs,
        if (errorType != null) "errorType": errorType,
        if (errorMessage != null) "errorMessage": errorMessage,
      },
      "request": {
        "method": method,
        "url": url,
        "headers": headers,
        if (requestBody != null) "body": requestBody,
      },
      "response": {
        "statusCode": statusCode,
        "body": responseBody,
      },
    };

    if (statusCode == -1) {
      _logger.e(_prettyJson(logObject));
    } else {
      _logger.i(_prettyJson(logObject));
    }
  }
}

```