# android-security

[1] https://www.youtube.com/watch?v=ShkUoLy0ZFE  
https://androidsecurity.info/  
https://github.com/mplacona  
https://www.placona.co.uk/  

Do not trust the device!  
Android Keystore since API level 23 with strong encryption (since API level 18 with "weak" encryption). TODO: Look into this more.    

## Certificate Pinning
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

Most of these checks can be removed by hackers if app is recompiled, so don't rely completely on those checks!  

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
