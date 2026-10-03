# AppFunctions API sample

A sample Android application demonstrating the usage of [AppFunctions](https://developer.android.com/ai/appfunctions)
to provide application functionality to system services and agents.

This sample is a chat application built with [Jetpack Compose](https://developer.android.com/jetpack/compose) and [Hilt](https://developer.android.com/training/dependency-injection/hilt-android).

## Features

- **AppFunctions Integration**: Provides core chat capabilities (sending messages, retrieving contacts, and making calls) via AppFunctions.
- **Modern Android Architecture**: Built using Kotlin, Compose, and Hilt.

## Getting Started

1. Clone the repository.
2. Open the project in Android Studio.
3. Build and run the app on an emulator or device.

## Development APK releases

The **ChatApp dev release** workflow builds the phone `:app` module on pushes to
`main` that change ChatApp or its release workflow. It can also be run manually
from GitHub Actions on `main`. It runs the phone debug unit tests, signs the dev APK,
and publishes a GitHub prerelease with:

- `chatapp-dev.apk`
- `chatapp-dev-cert.pem` and `chatapp-dev-cert.der`, the public signing certificate
- `signing-certificate.txt`, the verified APK signer fingerprints
- `SHA256SUMS`, checksums of the downloads

Install the APK directly after allowing installs from your browser or file manager,
or use `adb install -r chatapp-dev.apk`. You do **not** need to install the
certificate. Do not add it to Android's trusted CA store. Android verifies the
signature embedded in the APK itself.

The dev build is debuggable and uses package `com.example.chatapp.dev`, so it can
coexist with the original sample. It requires Android 7.0 / API 24 or newer;
AppFunctions availability depends on your Android version and system services.
Use `com.example.chatapp.dev` instead of `com.example.chatapp` in the adb examples
below when testing a dev build. The fully qualified function names stay unchanged.

### Signing setup for maintainers

The workflow needs two repository Actions secrets:

- `CHATAPP_DEV_KEYSTORE_BASE64`: a base64-encoded PKCS12 keystore with alias `chatapp-dev`.
- `CHATAPP_DEV_KEYSTORE_PASSWORD`: its store and key password.

Generate a dedicated development key once, upload it using `gh secret set`, and
keep a secure backup outside the repository. Reuse the same key for future builds.
Replacing it requires uninstalling the old app, which removes its local data.
Never commit or publish the keystore or its password. Only the public certificate
is included in releases. Signing secrets are not used in pull-request workflows.

For a local unsigned dev build, run `./gradlew :app:assembleDev` from `ChatApp`.
CI sets `-PdevVersionCode` to its increasing workflow run number so newer builds
can update existing installations. Reruns share the same version code.

## AppFunctions

This sample demonstrates how to define and provide the following AppFunctions in `AppFunctions.kt`:

- `searchContacts(query: String, filterType: String)`: Search for contacts or groups. `filterType` must be "INDIVIDUAL" or "GROUP".
- `send(name: String, endpointValue: String, messageBody: String, imageUri: Uri? = null)`: Sends a message to a recipient.
- `makeCall(contactName: String? = null, endpointValue: String? = null)`: Initiates a call to a recipient.

These functions are annotated with `@AppFunction`, allowing them to be discovered and executed by the system.

### AppFunctionSerializable

The sample demonstrates using `@AppFunctionSerializable` to provide custom data types to AppFunctions.
The following data classes are annotated with `@AppFunctionSerializable`:

- `ContactSearchResult`: Represents a contact search result.
- `Result`: The result of a message sending operation.
- `Recipient`: Represents a recipient of a message.
- `ChatGroup`: Represents a group of recipients.

### Verifying AppFunctions Behavior

You can verify the behavior of the provided AppFunctions using the `adb shell cmd app_function` tool.

For example, to execute the `send` function and simulate a system agent sending a message, run the following command in your terminal while the app is installed on an emulator or connected device:

```bash
adb shell "cmd app_function execute-app-function \
  --package com.example.chatapp \
  --function 'com.example.chatapp.appfunctions.AppFunctions#send' \
  --parameters '{\"name\": \"Alice\", \"endpointValue\": \"1\", \"messageBody\": \"Hello Alice!\"}'"
```

This will trigger the `send` AppFunction in the background, executing your sending logic natively.

To search for contacts:

```bash
adb shell "cmd app_function execute-app-function \
  --package com.example.chatapp \
  --function 'com.example.chatapp.appfunctions.AppFunctions#searchContacts' \
  --parameters '{\"query\": \"Alice\", \"filterType\": \"INDIVIDUAL\"}'"
```

## License

```
Copyright 2026 The Android Open Source Project

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
