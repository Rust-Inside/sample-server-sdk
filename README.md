# Sample Rust Core · Kotlin/JVM SDK

[English](README.md) | [한국어](README.ko.md)

A Kotlin/Spring server uses this SDK to build the profile response for `GET /v1/me`. Your server authenticates the request and looks up the account. The Rust library chooses the display name and serializes the response. Rust Inside builds the SDK and publishes it here as a JAR. [Rust implementation](https://github.com/Rust-Inside/sample-rust-core/blob/v0.1.0/src/api.rs)

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

Declare `GET /v1/me` in your Spring controller and configure Spring Security to require authentication for that route. In this example, your application provides `AccountService.requireByUsername` to look up the authenticated account. The controller sends the JSON returned by `meResponseJson()`.

```kotlin
import dev.rustinside.sample_rust_coresdk.meResponseJson
import java.security.Principal
import org.springframework.http.MediaType
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.RestController

@RestController
class ProfileController(private val accounts: AccountService) {
    @GetMapping("/v1/me", produces = [MediaType.APPLICATION_JSON_VALUE])
    fun getMe(principal: Principal): String {
        val account = accounts.requireByUsername(principal.name)
        return meResponseJson(account.id, account.handle, account.nickname)
    }
}
```

The current core source keeps the request path in a private Rust constant. The published `v0.1.0` JAR still contains the earlier `mePath()` API. [Current Rust source](https://github.com/Rust-Inside/sample-rust-core/blob/main/src/api.rs)

If the nickname is empty, the library uses the handle as the display name. For example, `meResponseJson("u42", "martin", "")` returns `{"id":"u42","handle":"martin","displayName":"@martin"}`.

If your service calls a separate account server, create an `AccountClient(apiOrigin, authorization)` and call `account.getMe()` from a coroutine.

## License

[MIT](LICENSE)
