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

```groovy
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

```groovy
dependencies {
    implementation 'com.blendvision.playback.link:bvplaybacklink:$latest_version'
}
```

> **Note**: After making changes, don't forget to sync your Gradle files to ensure that the project
> compiles successfully.

## Usage

### Create a PlaybackLink instance:

```kotlin
// create a playback link instance
val playbackLink = BVPlaybackLink.Builder(context).build()

// optional: Enable debug mode
// default is false
val playbackLink = BVPlaybackLink.Builder(context).enableDebugMode(true).build()

```

### Update the playback token to the playbackLink:

```kotlin
playbackLink.updatePlaybackToken([YOUR_PLAYBACK_TOKEN])
```

### Observes the playbackLink error event:

```kotlin
playbackLink.errorEvent.collect { error ->
    val message = when (error) {
        is BVPlaybackLinkError.ClientError -> error.message
        is BVPlaybackLinkError.UnknownError -> error.message
        ...etc
    }
}
```

### Get the resource info for analysis:

You can get the resource info for analysis by calling the `getResourceInfo()` method.
`ResourceInfo` contains the `id` and `type` of the resource.

```kotlin
val resourceInfo = playbackLink.getResourceInfo()
```

### Use the `resourceInfo` to set the `AnalyticsConfig` in the `Player` instance:

```kotlin
// input the resource info to the analytics config
val analyticsConfig = AnalyticsConfig(
    resourceId = resourceInfo.id,
    resourceType = resourceInfo.type
)

val playerConfig = PlayerConfig(
    licensekey = "XXXXXX",
    serviceConfig = PlayerConfig.ServiceConfig(licenseVersion = PlayerConfig.ServiceConfig.LicenseVersion.V2)
)

val player = UniPlayer.Builder(context = context,playerConfig = playerConfig).setAnalyticsConfig(analyticsConfig=analyticsConfig).build()

```

### To start a playback session, which sends a heartbeat to increase concurrent viewers:

```kotlin
playbackLink.startSession()
```

### To end a playback session, which stops the heartbeat to decrease concurrent viewers:

```kotlin
playbackLink.endSession()
```

### Remember to release the instance when it's no longer needed:

```kotlin
playbackLink.release()
```

> For a full code example, please refer to the [Sample app demo](https://github.com/BlendVision/Android-Playback-Link-Sample)

## Additional BVPlaybackLink APIs

```kotlin
    /**
     * Get the current playback token
     */
    fun getCurrentPlaybackToken(): String

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
