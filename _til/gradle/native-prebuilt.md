---
title: "Prebuilt Native Libraries"
tags: [gradle, native, build, prebuilt libraries]
category: gradle
---

You can specify a prebuilt library:

```groovy
model {
    // ...

    repositories {
        libs(PrebuiltLibraries) {
            boost {
                headers.srcDir "${outputDir}/include"
            }
            boost_atomic {
                binaries.withType(StaticLibraryBinary) {
                    def libName = targetPlatform.operatingSystem.windows ? "boost_atomic.lib" : "libboost_atomic.a"
                    staticLibraryFile = "${outputDir}/lib/${libName}"
                }
            }
        }
    }

    components {
        main(NativeExecutableSpec) {
            cpp.lib library: 'boost', linkage: 'api'
            cpp.lib library: 'boost_atomic', linkage: 'static'
        }
    }
}
```

There are three linkage types supported for a library dependency in an executable or library component:

* `shared` means to link the shared library
* `static` means to link the static library
* `api` means to use the library as a header-only library

In the above, `boost` is used as a header-only library, then we specifically link with `boost_atomic`, etc. Unfortunately, for something like boost, we need to add each of the boost_xxx libraries as separate.

I'm still looking into how we can simplify this. It should in theory be possible to do something like (the following is completely untested):

```groovy
libs(PrebuiltLibraries) {
    all {
        // COMPLETELY UNTESTED AS OF YET
        binaries.withType(StaticLibraryBinary) { bin ->
            def libName = targetPlatform.operatingSystem.windows ? "${bin.name}.lib" : "lib${bin.name}.a"
            staticLibraryFile = "${outputDir}/lib/${libName}"
        }
    }

    boost_atomic
    boost_foo
    boost_bar
}
```

I'm still trying to grok the model stuff, to be honest.
