# Sample Rust Core · Kotlin/JVM SDK

[English](README.md) | [한국어](README.ko.md)

A Kotlin/Spring server uses this SDK to build the profile response for `GET /v1/me`. Your server authenticates the request and looks up the account. The Rust library chooses the display name and serializes the response. Rust Inside builds the SDK and publishes it here as a JAR. [Rust implementation](https://github.com/Rust-Inside/sample-rust-core/blob/main/src/api.rs)

The JAR requires JVM 17 or later. It includes native libraries for Linux with glibc on x86-64 and ARM64, and for development on Apple Silicon Macs.

## Install with Gradle

Gradle can download the JAR and its Ivy descriptor directly from GitHub Releases. Add the following Ivy repository to `dependencyResolutionManagement.repositories` in `settings.gradle.kts`. Include `mavenCentral()` in the same block for Kotlin, coroutines, and JNA. [Release v0.1.0](https://github.com/Rust-Inside/sample-server-sdk/releases/tag/v0.1.0)

```kotlin
ivy {
    url = uri("https://github.com/Rust-Inside/sample-server-sdk/releases/download")
    patternLayout {
        ivy("v[revision]/[module]-[revision].xml")
        artifact("v[revision]/[artifact]-[revision].[ext]")
    }
    metadataSources { ivyDescriptor() }
    content { includeModule("dev.rustinside.sample_rust_coresdk", "sample-rust-core-jvm") }
}
```

Add the SDK dependency to the server module's `build.gradle.kts`.

```kotlin
dependencies {
    implementation("dev.rustinside.sample_rust_coresdk:sample-rust-core-jvm:0.1.0")
}
```

## Return the signed-in user's profile

`mePath()` returns `/v1/me`. In the controller for this route, pass the authenticated account's ID, handle, and nickname to `meResponseJson`. Send its result as the JSON response body.

```kotlin
import dev.rustinside.sample_rust_coresdk.meResponseJson
import org.springframework.http.MediaType
import org.springframework.http.ResponseEntity

fun profileResponse(id: String, handle: String, nickname: String): ResponseEntity<String> =
    ResponseEntity.ok()
        .contentType(MediaType.APPLICATION_JSON)
        .body(meResponseJson(id, handle, nickname))
```

If the nickname is empty, the library uses the handle as the display name. For example, `meResponseJson("u42", "martin", "")` returns `{"id":"u42","handle":"martin","displayName":"@martin"}`.

If your service calls a separate account server, create an `AccountClient(apiOrigin, authorization)` and call `account.getMe()` from a coroutine.

## License

[MIT](LICENSE)
