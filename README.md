# android-security

[1] https://www.youtube.com/watch?v=ShkUoLy0ZFE  
https://androidsecurity.info/  
https://github.com/mplacona  
https://www.placona.co.uk/  

Encrypt all the values.  
Apps will get decompiled.  
No passwords in source code! Will probably be committed to GIT at some point.  
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

## Tampering Detection

