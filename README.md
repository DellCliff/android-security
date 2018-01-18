# android-security

https://www.youtube.com/watch?v=ShkUoLy0ZFE  
https://androidsecurity.info/  
https://github.com/mplacona  
https://www.placona.co.uk/  
https://developer.android.com/training/safetynet/index.html  

Do not trust the device!  
Android Keystore since API level 23 with strong encryption (since API level 18 with "weak" encryption). TODO: Look into this more.    

## Certificate Pinning
Makes sure your app only communicates with your server.  
```
String hostname = "publicobject.com";
CertificatePinner certificatePinner = new CertificatePinner.Builder()
    .add(hostname, "sha256/AAAAAAAAAAAAAAAAAAAAA=")
    .build();
OkHttpClient client = OkHttpClient.Builder()
    .certificatePinner(certificatePinner)
    .build();

Request request = new Request.Builder()
    .url("https:// + hostname")
    .build();
client.newCall(request).execute();
```
## Decompiling

Apktool. Hack your own application to see what you give away.  
Encrypt all the values.  
Apps will get decompiled.  
No passwords in source code! Will probably be committed to GIT at some point.  

## Tampering Detection

Do not trust the device! Most of these checks can be removed by hackers if app is recompiled, so don't rely completely on those checks!  

Detect renaming.
```
private static final String PACKAGE_NAME = "info.androidsecurity.helloworld";
//...
if(mContext.getPackageName().compareTo(PACKAGE_NAME) != 0){
    Log.d(TAG, "this is a hack");
    /*
        Things you could do here:
        - show an alert to the user and refuse to proceed
        - make an HTTP request to your own server and alert you
    */            
}
```  
Published under a different market or APK?
``` 
String installer = mContext.getPackageManager().getInstallerPackageName(PACKAGE_NAME);
if (installer == null) {
    Log.d(TAG, "this is an APK");
    /*
        Things you could do here:
        - show an alert to the user and refuse to proceed
        - make an HTTP request to your own server and alert you
    */
}

String GOOGLE_PLAY = "com.android.vending";
String AMAZON_STORE = "com.amazon.venezia";
if (installer.compareTo(GOOGLE_PLAY) != 0 && installer.compareTo(AMAZON_STORE) != 0){
    Log.d(TAG, "not installed from either stores");
    /*
        Things you could do here:
        - show an alert to the user and refuse to proceed
        - make an HTTP request to your own server and alert you
    */
}
```  
Combined:
```
public boolean isHacked(Context context, String myPackageName, String google, String amazon)
{
  //Renamed?
  if (context.getPackageName().compareTo(myPackageName) != 0) {
      return true; // BOOM!
  }

  //Relocated?
  String installer = context.getPackageManager().getInstallerPackageName(myPackageName);

  if (installer == null){
      return true; // BOOM!
  }

  if (installer.compareTo(google) != 0 && installer.compareTo(amazon) != 0){
    return true; // BOOM!
  }
  return false; 
}
```  
Signature checking. Changes on recompiles, but again, can completely be removed or changed by hackers. No garantees here either.
```
final int VALID = 0;
final int INVALID = 1;
final String APP_SIGNATURE = "4920384CE4898347602CEEC";

public static int checkAppSignature(Context context) {
    try {
        PackageInfo packageInfo = context.getPackageManager()
            .getPackageInfo(context.getPackageName(), PackageManager.GET_SIGNATURES);
            
        for (Signature signature : packageInfo.signatures) {
            byte[] signatureBytes = signature.toByteArray();
            MessageDigest md = MessageDigest.getInstance("SHA");
            md.update(signature.toByteArray());
            // compare signatures
            if (SIGNATURE.equals(APP_SIGNATURE)) {
                return VALID;
            }
        }
    } catch (Exception ex) {
        // Assumes an issue in checking signature, but we let the caller decide on what to do.
    }
    return INVALID;
}
```  
Do not trust the device!  

## Root Detection

Do not trust the device!  

Rooted devices can execute "su". Again, a recompile can circumvent this check.   
```
static boolean canExecuteCommand(String command) {
    try {
        int exitValue = Runtime.getRuntime().exec(command).waitFor();
        if (exitValue != 0) {
            return false;
        } else {
            return true;
        }
    } catch (Exception e) {
        return false;
    }
}
```
XPosed injects a JAR file. Check for this JAR file.  
TODO  

Checking for emulators.  
```
Build.FINGERPRINT.startsWith("generic");
```  

Checking for app is debuggable.  
```
static boolean isDebuggable(Context context) {
    return (context.getApplicationInfo().flags & ApplicationInfo.FLAG_DEBUGGABLE) != 0;
}
```  

Do not trust the device!  

## Things to Use

Proguard:  
Name Obfuscation, Code Optimisation, Free  

DexGuard:  
Class Encryption, Call Hiding through Reflection, String Encryption, Certificate Checks, Debug Detection, Emulator Detection, Root Detection, Tamper Detection, Costs $$$

SafetyNet API:  
https://developer.android.com/training/safetynet/index.html  
Can check whether a given URL is known to be a threat.  
Can check:  
  Certified, genuine device that passes CTS  
  Certified device with unlocked bootloader  
  Genuine but uncertified device, such as when the manufacturer doesn't apply for certification  
  Device with custom ROM (not rooted)  
  Emulator  
  No device (protocol emulator script)  
  Signs of system integrity compromise, such as rooting  
  Signs of other active attacks, such as API hooking  

