# Latest version

3.0.0

> Starting from version 3.0.0 you can apply the Smartling SDK to library projects as well as application projects.
> Starting from version 2.4 you can use functionality of Context-Capture and In-App-Review modes in the same build.
See below installation and usage instructions for that combined **Review** mode.

## Installation

You have to add some configuration to your `build.gradle` files.

### Add Smartling Plugin Dependency

Add Smartling's Android repository to your main `build.gradle` file.

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

Add these repository URLSs into your `allprojects` `repositories` block.

```groovy
allprojects {
  repositories {
     ...
     maven { url 'https://raw.githubusercontent.com/Smartling/android-repository/releases'}
     maven { url "https://www.jitpack.io" }
  }
}
```

### Apply the Plugin to Your Application

Apply Smartling's Android plugin after the Android plugin:

```groovy
apply plugin: 'com.android.application'
apply plugin: 'com.smartling.android.application'
```

### Apply the Plugin to Other Modules (optional)

If you have a multi-module build, you can apply Smartling's library plugin to your Android
library modules.

Apply Smartling's Android plugin after the Android plugin:

```groovy
apply plugin: 'com.android.library'
apply plugin: 'com.smartling.android.library'
```

#### Configure the plugin

In your main `build.gradle` file, configure the Smartling plugin.

```groovy
apply plugin: 'com.smartling.android.config'

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

For *Over-the-Air* mode the instructions above are enough.

## Configuration Options

### projectId

The id of the project in Smartling dashboard

### projectSecret

Project AES key for the OTA updates.
> `otaServing` block is required only for `ota-serving` mode

### userIdentifier & userSecret

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

### In the code

Use your resource strings as usual:

- `context.getString(R.string.some_string)`
- `@string/some_string` in the xml layout files

They will be localized according to the SDK mode.

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

Create a test case for activity:

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

## Conditional Compilation

To avoid compiling **review** artifact (needed only for Context-Capture and In-App-Review) into release build you can apply different Smartling modes during configuration.

The SDK mode can be configured by a build variable to enable or disable it for a given build run:

```groovy
ext {
   SMARTLING_SDK_MODE = "in-app-review"
}

smartling {
    projectId = "deaf9496f"
    mode = SMARTLING_SDK_MODE
}
```

You can control this the same you update any build property at build time.

## Release notes

## Version 3.0.0

- Add support for multi module projects
- Fix issue with conditional compilation
- Upgrade outdated dependencies

### Version 2.5.2

- Fix issue with Android Gradle plugin 3.0.0 https://github.com/Smartling/android-repository/issues/8

### Version 2.5.1

- Fix issue https://github.com/Smartling/android-repository/issues/9 (NPE crash in Context-Capture mode)

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
