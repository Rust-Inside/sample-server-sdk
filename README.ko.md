# Sample Rust Core · Kotlin/JVM SDK

[English](README.md) | [한국어](README.ko.md)

Kotlin/Spring 서버는 이 SDK로 `GET /v1/me`의 프로필 응답을 만듭니다. 서버가 요청을 인증하고 계정을 조회하면, Rust 라이브러리가 표시 이름을 정하고 응답을 JSON으로 직렬화합니다. Rust Inside가 SDK를 빌드해 이 저장소에 JAR로 배포합니다. [Rust 구현](https://github.com/Rust-Inside/sample-rust-core/blob/v0.1.0/src/api.rs)

JVM 17 이상이 필요합니다. JAR에는 Linux(glibc) x86-64·ARM64용 네이티브 라이브러리와 Apple Silicon Mac에서 개발할 때 쓸 라이브러리가 들어 있습니다.

## Gradle에 추가하기

Gradle이 GitHub Releases에서 JAR과 Ivy 메타데이터를 직접 받습니다. `settings.gradle.kts`의 `dependencyResolutionManagement.repositories`에 아래 Ivy 저장소를 추가하세요. 같은 블록에 `mavenCentral()`도 있어야 Kotlin, 코루틴, JNA 의존성을 받을 수 있습니다. [v0.1.0 릴리스](https://github.com/Rust-Inside/sample-server-sdk/releases/tag/v0.1.0)

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

서버 모듈의 `build.gradle.kts`에 SDK 의존성을 추가합니다.

```kotlin
dependencies {
    implementation("dev.rustinside.sample_rust_coresdk:sample-rust-core-jvm:0.1.0")
}
```

## 로그인한 사용자의 프로필 응답

Spring 컨트롤러에 `GET /v1/me`를 선언하고, Spring Security에서 이 경로에 인증을 요구하도록 설정하세요. 예제의 `AccountService.requireByUsername`은 앱에서 구현하는 계정 조회 메서드입니다. 컨트롤러는 인증된 계정의 데이터를 `meResponseJson()`에 넘겨 JSON 응답을 보냅니다.

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

현재 Rust 코어 소스는 요청 경로를 내부 상수로 관리합니다. 이미 배포된 `v0.1.0` JAR에는 이전 `mePath()` API가 포함돼 있습니다. [현재 Rust 소스](https://github.com/Rust-Inside/sample-rust-core/blob/main/src/api.rs)

닉네임이 비어 있으면 핸들이 표시 이름이 됩니다. 예를 들어 `meResponseJson("u42", "martin", "")`은 `{"id":"u42","handle":"martin","displayName":"@martin"}`을 반환합니다.

다른 계정 서버에 요청을 보내는 서비스라면 `AccountClient(apiOrigin, authorization)`을 만들고 코루틴에서 `account.getMe()`를 호출하세요.

## 라이선스

[MIT](LICENSE)
