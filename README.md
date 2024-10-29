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

```kotlin
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        //add below
        maven {
            url = uri("https://maven.pkg.github.com/blendvision/Android-Packages")
            credentials {
                username = //TODO
                password = //TODO
            }
        }

    }
}
```

### Add the dependencies for the Player SDK to your module's app-level Gradle file, normally app/build.gradle:

```kotlin
dependencies {
    implementation("com.blendvision.playback.link:bvplaybacklink:$latest_version")
}
```

> **Note**: After making changes, don't forget to sync your Gradle files to ensure that the project
> compiles successfully.

## BVPlaybackLink Usage

BVPlaybackLink is a helper tool for working with the BlendVision Backend API.

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

### Observes the playbackLink state event:

```kotlin
playbackLinker.stateEvent.collect { state ->
    when (state) {
        is BVPlaybackLinkState.GetResourceInfoSuccess -> {
            // You can get the resource info from the state.
            // For example:
            // val resourceInfo = state.resourceInfo
        }
        is BVPlaybackLinkState.StartPlaybackSessionSuccess -> {
            // You can get the playback session info from the state.
            // For example:
            // val playbackSessionInfo = state.playbackSessionInfo
        }
        is BVPlaybackLinkState.StartHeartbeatSuccess -> {
            // The state indicates that the heartbeat has started successfully.
        }
        is BVPlaybackLinkState.EndPlaybackSessionSuccess -> {
            // The state indicates that the playback session has ended successfully.
        }
        is BVPlaybackLinkState.Failure -> {
            // The state indicates that failure and you can get the bvPlaybackLinkError object.
            // for example:
            // val errorCode = state.bvPlaybackLinkError.code
            // val errorMessage = state.bvPlaybackLinkError.message
        }
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
    licensekey = "[YOUR_LICENSE_KEY]",
    serviceConfig = PlayerConfig.ServiceConfig(licenseVersion = PlayerConfig.ServiceConfig.LicenseVersion.V2)
)

val player = UniPlayer.Builder(context = context,playerConfig = playerConfig).setAnalyticsConfig(analyticsConfig=analyticsConfig).build()

```

### To start a playback session, which sends a heartbeat to increase concurrent viewers:

```kotlin
playbackLink.startSession()
player.start()
```

### To end a playback session, which stops the heartbeat to decrease concurrent viewers:

```kotlin
playbackLink.endSession()
player.stop()
```

### Remember to release the instance when it's no longer needed:

```kotlin
playbackLink.release()
player.release()
```

> For a full code example, please refer to the [Sample app demo](https://github.com/BlendVision/Android-Playback-Link-Samples)

## BVPlayback Usage

BVPlayback is a helper tool that integrates `BVPlybackLink` and `BVPlayer` to simplify live streaming playback.

### Create a BVPlayback instance:

```kotlin

//First, implement the BVPlaybackStateListener interface. Each state will represent the current BVPlayback state.
private val bvPlaybackStateListener: BVPlaybackStateListener = object : BVPlaybackStateListener {
    override fun onStateChanged(bvPlaybackState: BVPlaybackState) {
        when (bvPlaybackState) {
            is BVPlaybackState.GetResourceInfoSuccess -> {
                // The state indicates that get resource info form BV Backend API successfully.
            }
            is BVPlaybackState.InitiatePlayerSuccess->{
                // This state indicates that the player has been successfully initialized, and you can now retrieve the player interface for use.
                // For example:
                // val player: UnityPlayer = bvPlaybackState.player
                // You can use the player interface to control the player.
                // Remember to set the player to the player view.
                // For example:

                /**
                playerView.setupControlPanel(
                autoKeepScreenOnEnabled = true,
                defaultPanelType = PanelType.EMBEDDED,
                disableControlPanel = null
                )

                playerView.configureSettingOption(
                SettingOptionConfig(forceHideAutoPlay = true)
                )

                playerView.setUnifiedPlayer(player)
                 */

            }
            is BVPlaybackState.GetPlaybackSessionInfoSuccess -> {
                // The state indicates that get playback session info form BV Backend API successfully.
            }
            is BVPlaybackState.GetDashUrlSuccess -> {
                // The state indicates that get dash url successfully.
            }
            is BVPlaybackState.PlayerLoadSuccess -> {
                // The state indicates that player load successfully.
            }
            is BVPlaybackState.PlayerStartSuccess -> {
                // The state indicates that player start successfully.
            }
            is BVPlaybackState.Failure-> {
                // The state indicates that failure and you can get the bvPlaybackError object.
                // for example:
                // val errorCode = bvPlaybackState.bvPlaybackError.code
                // val errorMessage = bvPlaybackState.bvPlaybackError.message
            }
        }
    }
}

// create a BVPlayback instance
// lifecycle is the lifecycle of the activity or fragment
// enableDebugMode is to enable debug mode, default is false
val bvPlayback = BVPlayback.Builder(context = context, lifecycle = lifecycle)
    .enableDebugMode(true)
    .create(bvPlaybackStateListener)

```

### load to play live streaming:

Provide the playback token and player config to the BVPlayback instance to play live streaming.

```kotlin
val playerConfig = PlayerConfig(
    licensekey = "[YOUR_LICENSE_KEY]",
    serviceConfig = PlayerConfig.ServiceConfig(licenseVersion = PlayerConfig.ServiceConfig.LicenseVersion.V2)
)

bvPlayback.load(
    playbackToken = "[YOUR_PLAYBACK_TOKEN]",
    playerConfig = playerConfig
)
```

> For a full code example, please refer to the [Sample app demo](https://github.com/BlendVision/Android-Playback-Link-Samples)

#### If you want to know more about the APIs, please refer to the [APIs docs](https://blendvision.github.io/Android-Playback-Link-SDK/)
