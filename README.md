# rules_android + maven Toolchain issu reproduction

This is to reproduce a problem occurring when using [rules_android](https://github.com/bazelbuild/rules_android) and a [maven repository via rules_jvm_external](https://github.com/bazel-contrib/rules_jvm_external).

The issue is that, when using certain `androidx` Maven repositories, attempting to use the repository as a dependency results in the following error:
```
$ bazel build //src/main/java/com/example/bazel:greeter_activity --toolchain_resolution_debug='@bazel_tools//tools/android:sdk_toolchain_type' 
WARNING: Build option --toolchain_resolution_debug has changed, discarding analysis cache (this can be expensive, see https://bazel.build/advanced/performance/iteration-speed).
INFO: ToolchainResolution: Performing resolution of @@bazel_tools//tools/android:sdk_toolchain_type for target platform @@platforms//host:host
      ToolchainResolution: No @@bazel_tools//tools/android:sdk_toolchain_type toolchain found for target platform @@platforms//host:host.
ERROR: .../external/rules_jvm_external~~maven~my_maven/BUILD:161:11: While resolving toolchains for target @@rules_jvm_external~~maven~my_maven//:androidx_appcompat_appcompat (1bfa2c2): No matching toolchains found for types @@bazel_tools//tools/android:sdk_toolchain_type.
```

Ongoing Slack Thread: https://bazelbuild.slack.com/archives/CD9FZL3KR/p1730468977831409

## To Reproduce

1. Check out this repository
2. Ensure your `ANDROID_HOME` environment variable is set appropriately.
3. Execute: `bazel build //src/main/java/com/example/bazel:greeter_activity`

### Ineffective changes
The following items have had no effect on fixing the build:
- Including the following in [.bazelrc](./.bazelrc):
  ```
  common --android_sdk=@androidsdk//:sdk
  ```
- Including the following in [MODULE.bazel](./MODULE.bazel):
  ```python
  android_sdk_repository_extension = use_extension("@rules_android//rules/android_sdk_repository:rule.bzl", "android_sdk_repository_extension")
  use_repo(android_sdk_repository_extension, "androidsdk")
  register_toolchains("@androidsdk//:sdk-toolchain", "@androidsdk//:all")
  ```

### Effective changes
Including the following line in [WORKSPACE.bzlmod](./WORKSPACE.bzlmod) made it work:
```python
android_sdk_repository(name = "androidsdk")
```

### Notes
- This has been attempted on an M1 Macbook Pro running Sonoma 14.7.
- The Android App was lifted from the [Bazel Android Tutorial example](https://github.com/bazelbuild/examples/tree/fdfabba6aa8065f5b1349931dc08678fdccdb678/android/tutorial).
