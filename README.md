# BlendVision Playback Link SDK

The PlaybackLink SDK is an assist for dealing with BlendVision Backend API

## Requirements

- **IDE**: Android Studio 3.0 or later
- **minSdkVersion**: 21
- **targetSdkVersion**: 34
- **Kotlin Version**: 1.9.x or later

## Integration

### In your settings.gradle file, `dependencyResolutionManagement` sections:
[Gets username and password](https://github.com/BlendVision/Android-Playback-Link-SDK/wiki/Android%E2%80%90Playback%E2%80%90Link-pull-credentials)
```groovy=
dependencyResolutionManagement {
  repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
  repositories {
    google()
    mavenCentral()
    //add below
    maven {
      url = uri("https://maven.pkg.github.com/blendvision/Android-Playback-Link-SDK")
      credentials {
        username = //TODO
        password = //TODO
      }
    }

  }
}
```

### Add the dependencies for the Player SDK to your module's app-level Gradle file, normally app/build.gradle:

```groovy=
dependencies {
    implementation 'com.blendvision.playback.link:bvplaybacklink:$latest_version'
}
```

> **Note**: After making changes, don't forget to sync your Gradle files to ensure that the project
> compiles successfully.

## Interface
```kotlin=

val errorEvent: Flow<BVPlaybackLinkError>

/**
 * Update the playback token
 */
fun updatePlaybackToken(token: String)

/**
 * Get the resource info for analysis
 *
 * @return The resource info for analysis, or null if error occurred
 */
suspend fun getResourceInfo(): ResourceInfo?

/**
 * Start a playback session with updated playback token
 */
fun startSession()

/**
 * End a playback session with updated playback token
 */
fun endSession()

/**
 * Release this instance
 */
fun release()

```

## Usage
Positions 1 to 4 shown below are where the interface may be called.
```kotlin=
// 1
val playbackLink = BVPlaybackLink.Builder().build()
playbackLink.updatePlaybackToken(token)
val resourceInfo = playbackLink.getResourceInfo()


val player = Player.Builder(
        PlayerConfig(
            licensekey = "XXXXXX",
            serviceConfig = V2
        )
    )
    .setAnalyticsConfig(
        AnalyticsConfig(
            // 2
            resourceId = resourceInfo.id,
            resourceType = resourceInfo.type
        )
    )

    
// load content to playback
player.load(MediaConfig)
playbackLink.startSession()             // 3

// exit playback OR error occurs
player.stop()
player.release()

playbackLink.endSession()               // 4
playbackLink.release()

```
