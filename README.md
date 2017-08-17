# Latest version
2.5.1

> Starting from version 2.4 you can use functionality of Context-Capture and In-App-Review modes in the same build.
See below installation and usage instructions for that combined **Review** mode.

# Documentation
## Installation
You have to add some configuration into your `app`'s `build.gradle`

#### Step 1: Add Smartling plugin dependency
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
#### Step 2: Apply plugin

```groovy
apply plugin: 'com.android.application'
apply plugin: 'com.smartling.android.plugin'
```
Assure you add it after the Android application plugin.

#### Step 3: Configure the plugin

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

#### Step 4: Add sdk dependency

Add specific repository urls into your `repositories` block.
Either into `allprojects` block of your project's `build.gradle`:
```groovy
allprojects {
  repositories {
     ...
     maven { url 'https://raw.githubusercontent.com/Smartling/android-repository/releases'}
     maven { url "https://www.jitpack.io" }
  }
}
```
... or into `repositories` block of your application's `build.gradle`

```groovy
repositories {
   ...
   maven { url 'https://raw.githubusercontent.com/Smartling/android-repository/releases'}
   maven { url "https://www.jitpack.io" }
}
```
Add sdk dependency into `dependencies` block as usually:
```groovy
dependencies {
  compile "com.smartling.android:sdk:{latest_version}"
  ...
}
```
For *Over-the-Air* mode the instructions above are enough.

#### Step 5: Add review dependency to use either Context-Capture or In-App-Review mode

```groovy
dependencies {
  compile "com.smartling.android:sdk:{latest_version}"
  compile "com.smartling.android:review:{latest_version}"
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
> `auth` block is required for `context-capture` mode and `uploadSourceStrings` gradle task (see below)

### mode
- `ota-serving` - Published strings are served to the user in his language and displayed in the app
- `in-app-review` - Members of your team can log in to edit strings and review them in context inside the app
- `context-capture` - You are able to capture the screenshots with strings attached and upload them to the server
- `disabled` (default) - The SDK doesn't affect the app whatsoever

> Since Context-Capture and In-App-Review modes are combined into one from version 2.4 **mode** option affects in which of those modes application would be started. Then you can switch to another mode.

### logLevel
Defines the min level of the log messages the SDK exposes to Logcat
- none
- verbose
- debug
- info
- warn
- error

## Usage

### Upload resource files with strings to the dashboard
Use
```
./gradlew app:uploadSourceStrings
```
to retrieve all the resource files (from `/res` folder) containing `string`, `string-array` and `plural` tags for all the build variants and flavors (except `debug` ones) and upload them to the dashboard.

Use it with `-Pauthorize` parameter to authorize the strings for the translation.
```
./gradlew app:uploadSourceStrings -Pauthorize
```
> Note that this task requires you to add `auth` block into your smartling configuration in `build.gradle`

### In the code
Use your resource strings as usual
- `context.getString(R.string.some_string)` in the code
- `@string/some_string` in the xml layout files

and they will be localized according to the SDK mode.

## Mode-specific usage
### OTA (Over-the-Air)

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

### Both Context-Capture (manual) and In-App-Review
Make either three-finger long touch or double-click at any point of the screen to open dashboard screen. In the dashboard screen for every mode you can do mode-specific actions and switch to another one.

#### Gestures for Context-Capture
Either one- or two-finger long touch causes capturing of the screenshot, attaching the strings to it and uploading the image along with strings' coordinates to the server.

#### Gestures for In-App-Review
Make one-finger long touch on any string on the screen to edit it.
Make two-finger long touch at any point of the screen to edit all the strings from that screen.

### Context-Capture (Automatic way)
It's possible to integrate calls of the capture methods into your UI tests. 
Make setup as you usually do for the UI tests along with Smartling SDK dependency (`context-automation` module)...

```groovy
dependencies {
   androidTestCompile 'com.android.support.test:runner:0.5'
   androidTestCompile 'com.android.support.test:rules:0.5'
   androidTestCompile 'com.smartling.android:context-automation:{latest_version}'
}
```
... and create a test case for activity

```java
@RunWith(AndroidJUnit4.class)
public class MainActivityCaptureTest extends ContextCaptureTestCase {

    @Rule
    public ActivityTestRule<MainActivity> rule = new ActivityTestRule<>(MainActivity.class, false, false);

    @Test
    public void takeScreenshot() throws Exception {
        rule.launchActivity(new Intent());
        ...some movements, button presses, clicks if needed
        SmartlingInstrumentation.grab(rule.getActivity());
    }
}
```
> It's needed to prevent activity launch so that strings were loaded before it's launched. That's why we create `ActivityTestRule` object with `false` parameters.

`ContextCaptureTestCase` does all the work to preload the strings.

That's it. After this test case is executed you will see the screenshot of `MainActivity` with the strings marked on it in the dashboard.

## Build variants
To avoid compiling **review** artifact (needed only for Context-Capture and In-App-Review) into release build you can apply different Smartling settings into each build variant.

See below the example of `build.gradle` file with few flavors and release and debug variants.

```groovy
buildscript {
    ext.smartlingVersion = '2.4'
    ext.smartlingRepoUrl = 'https://raw.githubusercontent.com/Smartling/android-repository/releases'
    repositories {
        jcenter()
        maven { url "${smartlingRepoUrl}" }
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.2'
        classpath "com.smartling.android:plugin:${smartlingVersion}"
    }
}
apply plugin: 'com.android.application'
apply plugin: 'com.smartling.android.plugin'

repositories {
    jcenter()
    maven { url "$smartlingRepoUrl" }
    ...
}
...

smartling {
    projectId = "..."
    description = "..."
    mode = "ota-serving"
    logLevel = "verbose"
    otaServing {
        projectSecret = "..."
    }
    auth {
        userIdentifier = "..."
        userSecret = "..."
    }
    buildVariants {
        reviewDebug {
            mode = "context-capture"
        }
        reviewRelease {
            mode = "context-capture"
        }
    }
}

android {
    ...
    signingConfigs {
        release {
            storeFile ...
            storePassword ...
            keyAlias ...
            keyPassword ...
        }
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
        }
    }
    productFlavors {
        prod {
        }
        review {
            applicationIdSuffix ".review"
        }
    }
    ...
}

dependencies {
    compile "com.smartling.android:sdk:$smartlingVersion"
    reviewCompile "com.smartling.android:review:$smartlingVersion"
    ...
}
...
```
Note the `buildVariants` closure inside `smartling` block. Also, note that you should use full build variant names inside it (like `reviewDebug`). If some build variant is missed inside `buildVariants` block it's okay and settings from `smartling` block will be applied. This configuration allows you to make 2 builds not changing `build.gradle` file - one for release to Play Store (`prodRelease`) and another one for the translators and reviewers (`reviewRelease`). Also, it allows you to have 2 applications on your device to debug - one in OTA mode (`prodDebug`) and another one in Review mode (`reviewDebug`).

> You can still not to use build variants. Just don't forget to change *mode* from `context-capture` or `in-app-review` before release.

# Release notes

### Version 2.5.1
-Fix issue https://github.com/Smartling/android-repository/issues/9 (NPE crash in Context-Capture mode)

### Version 2.5
- Significantly reduce the size that Smartling SDK adds to client application's apk file.
- Fix issue with `ViewPager` capturing (in Context-Capture mode)

### Version 2.4
- Context-Capture and In-App-Review modes are combined into one

### Version 2.3.3
- Fix issue https://github.com/Smartling/android-repository/issues/7 (black title color in Toolbar)

### Version 2.3.2
- Fix issue https://github.com/Smartling/android-repository/issues/5 (crash if WebView is presented in the layout).
- Create gradle task `uploadSourceStrings` in the plugin to upload string resource files to the dashboard.

### Version 2.3.1
- Fix issue https://github.com/Smartling/android-repository/issues/3 associated with work in Instant Run mode.

### Version 2.3.0
- Create `context-automation` module as separate Gradle module to be integrated as dependency for UI integration tests.
