# Latest version
2.2.2

# Version 2.2.2
## Installation
You have to add some configuration into your `app`'s `build.gradle`

**Step 1:** Add Smartling plugin dependency
```groovy
buildscript {
  repositories {
    maven { url 'https://raw.githubusercontent.com/Smartling/android-repository/releases'}
    ...
  }
  dependencies {
    classpath 'com.smartling.android:plugin:{latest_version}'
    ...
  }
}
```
**Step 2:** Apply plugin

```groovy
apply plugin: 'com.android.application'
apply plugin: 'com.smartling.android.plugin'
```
Assure you add it after the Android application plugin.

**Step 3:** Configure the plugin

```groovy
smartling {
  projectId = "<Project ID>"
  otaServing {
    projectSecret = "<Project AES Key>"
  }
  auth {
    userIdentifier = "<User Identifier>"
    userSecret = "<User Secret>"
  }
  mode = "ota-serving"
  logLevel = "verbose"
}
```

**Step 4:** Add sdk dependency
```groovy
dependencies {
  compile "com.smartling.android:sdk:{latest_version}"
  ...
}
```

## Options
### projectId
The id of the project in Smartling dashboard

### projectSecret
Project AES key for the OTA updates.
> `otaServing` block is required only for `ota-serving` mode

### userIdentifier, userSecret
The credentials from the project-specific token.
> `auth` block is required for `context-capture` mode

### mode
- `ota-serving` - Published strings are served to the user in his language and displayed in the app
- `in-app-review` - Members of your team can log in to edit strings and review them in context inside the app
- `context-capture` - You are able to capture the screenshots with strings attached and upload them to the server
- `disabled` (default) - The SDK doesn't affect the app whatsoever

### logLevel
Defines the min level of the log messages the SDK exposes to Logcat
- none
- verbose
- debug
- info
- warn
- error

## Usage

Use your resource strings as usual
- `context.getString(R.string.some_string)` in the code
- `@string/some_string` in the xml layout files

and they will be localized according to the SDK mode.

## Mode-specific usage
### OTA (over-the-air)

#### Wait for the localized strings
Generally, the localized strings for certain language are fetching from the server after activity is created. Thus, for the very first launch user will see the strings in the project source language till he goes to the other activity or relaunches the app.

To prevent that and wait for the localized strings are loaded you can do the following.

Create `SplashActivity` or use existing one and implement `OTAListener` interface with it.

```java
public class SplashActivity extends Activity implements OTAListener {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_splash);
    }

    @Override
    public void onStringsLoaded() {
        Intent intent = new Intent(this, MainActivity.class);
        startActivity(intent);
    }
}
```
Method `onStringLoaded()` is calling:
- after strings are loaded if they don't exist for certain language or if it's the first launch after application was closed by the system or force-quitted by the user
- after activity's `onResume()` method in all other cases

### Context-capture
//TODO add 

# Version 1.9.1
## Installation
Insert following lines into your `build.gradle` file

```groovy
buildscript {
  repositories {
    maven { url 'https://raw.githubusercontent.com/Smartling/android-repository/releases'}
    ...
  }
  dependencies {
    classpath 'com.smartling.android:plugin:1.9.1'
    ...
  }
}

apply plugin: 'com.android.application'
apply plugin: 'com.smartling.android.plugin'

dependencies {
  compile "com.smartling.android:sdk:1.9.1"
  ...
}

smartling {
  projectId = "<Project ID>"
  projectSecret = "<Project AES Key>"
  mode = "ota-serving"
  logLevel = "verbose"
}
```
## Options
### projectId
The id of the project in Smartling dashboard

### projectSecret
Project AES key for the OTA updates (required for `ota-serving` mode)
### mode
- `ota-serving` - Published strings are served to the user in his language and displayed in the app
- `in-app-review` - Members of your team can log in to edit strings and review them in context inside the app
- `disabled` (default) - The SDK doesn't affect the app whatsoever

### logLevel
Defines the min level of the log messages the SDK exposes to Logcat
- none
- verbose
- debug
- info
- warn
- error

## Usage

Use your resource strings as usual
- `context.getString(R.string.some_string)` in the code
- `@string/some_string` in the xml layout files

and they will be localized according to the SDK mode.
