```dart

Future<void> _onProceed() async {  
  final bool isFormValid = _formKey.currentState?.validate() ?? false;  
  final bool isBankValid = _bankKey.currentState?.validate() ?? false;  
  final bool isRdValid = _rdDeviceKey.currentState?.validate() ?? false;  
  
  if (!isFormValid || !isBankValid || !isRdValid) {  
    return;  
  }  
  
  final NativeBridgeProvider nativeProvider = context.read<NativeBridgeProvider>();  
  final AepsProvider aepsProvider = context.read<AepsProvider>();  
  
  /// 1️⃣ Start biometric (TRANSACTION PID)  final String? biometricXml = await nativeProvider.startBiometricCapture(  
    packageName: selectedRdDevice!.id,  
    forKyc: false,  
  );  
  
  final RdResult rdResult = RdParser.parsePidXml(biometricXml!);  
  
  Logger().w(rdResult.toJson());  
  
  if (rdResult.isSuccess) {  
    final DeviceInfoModel? deviceInfo = nativeProvider.deviceInfo;  
    if (deviceInfo == null) {  
      ToastUtils.showError('Device info not available');  
      return;  
    }  
  
    /// 2️⃣ Build AEPS Cash Deposit request    final AepsTxnRequest request = AepsTxnRequest(  
      aadhaar: aadharController.text.trim(),  
      mobile: mobileController.text.trim(),  
      amount: operatorIncode == 'CW' ? amountController.text.trim() : '0.0',  
      latitude: deviceInfo.latitude.toString(),  
      longitude: deviceInfo.longitude.toString(),  
      type: operatorIncode,  
      iin: '${selectedBank!.iin},${selectedBank!.bankname}',  
      biometric: biometricXml,  
    );  
  
    /// 3️⃣ Call AEPS transaction    LoaderUtil.show(message: 'Processing...');  
  
    final ApiResponseHandler<AepsTransactionResponse> response = await aepsProvider.aepsTransaction(  
      context,  
      request: request,  
    );  
      
  
    Logger().w(response.toJson());  
  
    LoaderUtil.hide();  
    nativeProvider.resetBiometric();  
  
    if (!response.isSuccess || response.success == null) {  
      ToastUtils.showError(response.success!.message);  
      return;  
    }  
  
    final AepsTransactionResponse txnResponse = response.success!;  
    if (txnResponse.status == 1) {  
      _resetForm();  
      ToastUtils.showSuccess(txnResponse.message);  
      Navigator.push(context, MaterialPageRoute(builder: (context) => AepsSlipScreen(txn: txnResponse.data!)));  
    } else {  
      Navigator.push(context, MaterialPageRoute(builder: (context) => AepsSlipScreen(txn: txnResponse.data!)));  
      ToastUtils.showError(txnResponse.message);  
    }  
  } else {  
    ToastUtils.showError(rdResult.errInfo);  
  }  
}
```