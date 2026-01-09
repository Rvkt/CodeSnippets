```kotlin

package com.rojipay

import android.app.Activity

import android.content.Intent

import android.net.Uri

import android.os.Bundle

import android.util.Log

import android.widget.Toast

import com.google.android.gms.location.FusedLocationProviderClient

import com.google.android.gms.location.LocationServices

import com.softmintindia.aepsauth.presentation.AepsAuthValidation

import io.flutter.embedding.android.FlutterActivity

import io.flutter.embedding.engine.FlutterEngine

import io.flutter.plugin.common.MethodChannel

class MainActivity : FlutterActivity() {

private val CHANNEL = "android_id_channel"

private val AEPS_REQUEST_CODE = 9001

private val RD_REQUEST_CODE = 1001

private lateinit var fusedLocationClient: FusedLocationProviderClient

private lateinit var methodChannel: MethodChannel

private var pendingResult: MethodChannel.Result? = null

companion object {

private const val TAG_ACTIVITY = "ACTIVITY_RESULT"

private const val TAG_RD = "RD_CALLBACK"

private const val TAG_AEPS = "AEPS"

}

override fun onCreate(savedInstanceState: Bundle?) {

super.onCreate(savedInstanceState)

fusedLocationClient = LocationServices.getFusedLocationProviderClient(this)

Log.d(TAG_ACTIVITY, "MainActivity created")

}

// ─────────────────────────────────────────────

// 🔌 Flutter Channel Setup

// ─────────────────────────────────────────────

override fun configureFlutterEngine(flutterEngine: FlutterEngine) {

super.configureFlutterEngine(flutterEngine)

methodChannel = MethodChannel(

flutterEngine.dartExecutor.binaryMessenger,

CHANNEL

)

methodChannel.setMethodCallHandler { call, result ->

when (call.method) {

"getDeviceAndAppInfo" -> {

DeviceInfoService.getDeviceAndAppInfo(this) {

result.success(it)

}

}

"launchAepsAuth" -> {

val token = call.argument<String>("token")

if (token.isNullOrBlank()) {

result.error("INVALID", "Token missing", null)

return@setMethodCallHandler

}

launchAepsAuth(token)

result.success(null)

}

"isAppInstalled" -> {

val pkg = call.argument<String>("pkg").orEmpty()

result.success(pkg.isNotBlank() && isAppInstalled(pkg))

}

"openPlayStore" -> {

val pkg = call.argument<String>("pkg")

if (pkg.isNullOrBlank()) {

result.error("INVALID_ARGS", "pkg missing", null)

} else {

openPlayStore(pkg)

result.success(null)

}

}

"launchRdApp" -> {

val pkg = call.argument<String>("pkg")

val pidXml = call.argument<String>("pidXml")

if (pkg.isNullOrBlank() || pidXml.isNullOrBlank()) {

result.error(

"INVALID_ARGS",

"pkg or pidXml missing",

null

)

return@setMethodCallHandler

}

pendingResult = result

launchRdApp(pkg, pidXml)

}

else -> result.notImplemented()

}

}

}

// ─────────────────────────────────────────────

// 📦 Utilities

// ─────────────────────────────────────────────

private fun isAppInstalled(packageName: String): Boolean =

try {

packageManager.getPackageInfo(packageName, 0)

true

} catch (e: Exception) {

false

}

private fun openPlayStore(packageName: String) {

try {

startActivity(

Intent(

Intent.ACTION_VIEW,

Uri.parse("market://details?id=$packageName")

)

)

} catch (e: Exception) {

startActivity(

Intent(

Intent.ACTION_VIEW,

Uri.parse("https://play.google.com/store/apps/details?id=$packageName")

)

)

}

}

private fun showToast(msg: String) {

Toast.makeText(this, msg, Toast.LENGTH_SHORT).show()

}

// ─────────────────────────────────────────────

// 🔐 RD SERVICE LAUNCH

// ─────────────────────────────────────────────

private fun launchRdApp(packageName: String, pidXml: String) {

val intentAction = when (packageName) {

"in.gov.uidai.facerd" ->

"in.gov.uidai.rdservice.face.CAPTURE"

"com.mantra.mis100v2.rdservice" ->

"in.gov.uidai.rdservice.iris.CAPTURE"

else ->

"in.gov.uidai.rdservice.fp.CAPTURE"

}

val isFaceRd =

intentAction == "in.gov.uidai.rdservice.face.CAPTURE"

val intent = Intent(intentAction).apply {

`package` = packageName

putExtra(

if (isFaceRd) "request" else "PID_OPTIONS",

pidXml

)

}

if (intent.resolveActivity(packageManager) != null) {

startActivityForResult(intent, RD_REQUEST_CODE)

Log.i(TAG_RD, "RD launched: $intentAction")

} else {

pendingResult?.error(

"RD_NOT_FOUND",

"RD Service app not available",

null

)

pendingResult = null

}

}

// ─────────────────────────────────────────────

// 💳 AEPS SDK LAUNCH

// ─────────────────────────────────────────────

private fun launchAepsAuth(token: String) {

Log.d(TAG_AEPS, "Launching AEPS SDK")

val intent = Intent(this, AepsAuthValidation::class.java).apply {

putExtra("TOKEN", token)

addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)

}

startActivityForResult(intent, AEPS_REQUEST_CODE)

}

// ─────────────────────────────────────────────

// 📲 Activity Result Handler

// ─────────────────────────────────────────────

override fun onActivityResult(

requestCode: Int,

resultCode: Int,

data: Intent?

) {

super.onActivityResult(requestCode, resultCode, data)

Log.i(

TAG_ACTIVITY,

"requestCode=$requestCode resultCode=$resultCode"

)

when (requestCode) {

RD_REQUEST_CODE -> handleRdResult(resultCode, data)

AEPS_REQUEST_CODE -> handleAepsResult(data)

else -> Log.w(

TAG_ACTIVITY,

"Ignoring unknown requestCode=$requestCode"

)

}

}

// ─────────────────────────────────────────────

// 🔐 RD RESULT HANDLER

// ─────────────────────────────────────────────

private fun handleRdResult(resultCode: Int, data: Intent?) {

if (pendingResult == null) {

Log.w(TAG_RD, "No pendingResult found")

return

}

if (resultCode != Activity.RESULT_OK || data == null) {

pendingResult?.error(

"RD_CANCELLED",

"RD capture cancelled or failed",

null

)

pendingResult = null

return

}

val extras = data.extras

var rawXml: String? = null

extras?.keySet()?.forEach { key ->

val value = extras.get(key)

Log.d(TAG_RD, "Extra[$key]=$value")

if (rawXml == null && value is String && value.isNotBlank()) {

rawXml = value

}

}

if (!rawXml.isNullOrBlank()) {

methodChannel.invokeMethod(

"getBiometricCallback",

rawXml

)

pendingResult?.success(null)

} else {

pendingResult?.error(

"EMPTY_RD_DATA",

"No RD XML returned",

null

)

}

pendingResult = null

}

// ─────────────────────────────────────────────

// 💳 AEPS RESULT HANDLER

// ─────────────────────────────────────────────

private fun handleAepsResult(data: Intent?) {

val extras = data?.extras

val status = extras?.getString("status") ?: "0"

val message = extras?.getString("message") ?: "AEPS Failed"

Log.d(TAG_AEPS, "status=$status message=$message")

methodChannel.invokeMethod(

"onAepsResult",

mapOf(

"status" to status,

"message" to message

)

)

}

}

```