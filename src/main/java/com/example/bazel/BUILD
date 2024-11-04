load("@rules_android//android:rules.bzl", "android_library")

package(
    default_visibility = ["//src:__subpackages__"],
)

android_library(
    name = "greeter_activity",
    srcs = [
        "Greeter.java",
        "MainActivity.java",
    ],
    manifest = "AndroidManifest.xml",
    resource_files = glob(["res/**"]),
    deps = [
        "@my_maven//:androidx_appcompat_appcompat",
    ]
)