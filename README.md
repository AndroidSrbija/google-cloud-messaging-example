# Google Cloud Message Example

###Go to this [link](https://developers.google.com/cloud-messaging/android/client) and find GET A CONFIGURATION FILE, follow the steps and create your app for google analytics

###Add plugin and dependency with Gradle

```xml
classpath 'com.google.gms:google-services:1.5.0-beta2'
```

```xml
apply plugin: 'com.google.gms.google-services'
```

###Add permissions, services and receiver to your project manifest

```xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.WAKE_LOCK"/>
```

```xml
<receiver
    android:name="com.google.android.gms.gcm.GcmReceiver"
    android:exported="true"
    android:permission="com.google.android.c2dm.permission.SEND" >
    <intent-filter>
        <action android:name="com.google.android.c2dm.intent.RECEIVE" />
        <category android:name="gcm.play.android.samples.com.gcmquickstart" />
    </intent-filter>
</receiver>
<service
    android:name=".gcm.MyGcmListenerService"
    android:exported="false" >
    <intent-filter>
        <action android:name="com.google.android.c2dm.intent.RECEIVE" />
    </intent-filter>
</service>
<service
    android:name=".gcm.MyInstanceIDListenerService"
    android:exported="false">
    <intent-filter>
        <action android:name="com.google.android.gms.iid.InstanceID"/>
    </intent-filter>
</service>
<service
    android:name=".gcm.RegistrationIntentService"
    android:exported="false">
</service>
```

### Check if there is gps on device

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        if (checkPlayServices()) {
            Intent intent = new Intent(this, RegistrationIntentService.class);
            startService(intent);
        }

    private boolean checkPlayServices() {
        GoogleApiAvailability apiAvailability = GoogleApiAvailability.getInstance();
        int resultCode = apiAvailability.isGooglePlayServicesAvailable(this);
        if (resultCode != ConnectionResult.SUCCESS) {
            if (apiAvailability.isUserResolvableError(resultCode)) {
                apiAvailability.getErrorDialog(this, resultCode, PLAY_SERVICES_RESOLUTION_REQUEST).show();
            } else {
                finish();
            }
            return false;
        }
        return true;
    }
}
```

### Create intent service and override onHandleIntent
```java
public class RegistrationIntentService extends IntentService {

@Override
protected void onHandleIntent(Intent intent) {
    try {
        InstanceID instanceID = InstanceID.getInstance(this);
        String token = instanceID.getToken(/*put your sender id */, GoogleCloudMessaging.INSTANCE_ID_SCOPE, null);
        sendRegistrationToServer(token);
        subscribeTopics(token);
    } catch (Exception e) {
        Log.d(TAG, "Failed to complete token refresh", e);
    }
    Intent registrationComplete = new Intent(REGISTRATION_COMPLETE);
    LocalBroadcastManager.getInstance(this).sendBroadcast(registrationComplete);
}

}
```

### Create GcmListenerService and override onMessageReceived

```java
public class MyGcmListenerService extends GcmListenerService {
 @Override
    public void onMessageReceived(String from, Bundle data) {
        String message = data.getString("message");
        if (from.startsWith("/topics/")) {
            // message received from some topic.
        } else {
            // normal downstream message.
        }
        // you can replace sending notification with something you need
        sendNotification(message);
    }
 }
```

### Create InstanceIDListenerService and override onTokenRefresh

```java
public class MyInstanceIDListenerService extends InstanceIDListenerService {           
@Override
    public void onTokenRefresh() {
        Intent intent = new Intent(this, RegistrationIntentService.class);
        startService(intent);
    }
}
```

### Example how to send message

Host value: https://gcm-http.googleapis.com/gcm/send

Path value: /gcm/send

Authorization: key=app key here

specific user:

```json
{
    "to": "user token here",
    "data": {
      "message": "Message here",
     }
}
```


topic:

```json
{
    "to": "/topics/global",
    "data": {
      "message": "Message here",
     }
}
```
