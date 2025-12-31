---
title: 'How to Fix the "Server May Not Support Tls Protocol Version" in gradle project'
date: 2024-01-25T10:23:00+08:00
draft: false
tags: [android, gradle, tls]
---

## Problem

I have a flutter project managed by a gradle, when I try to run the project in android phone, I got the following errors:

```
Execution failed for task ':location:compileDebugLibraryResources'.
> A failure occurred while executing com.android.build.gradle.tasks.CompileLibraryResourcesTask$CompileLibraryResourcesAction
   > Could not isolate value com.android.build.gradle.tasks.CompileLibraryResourcesTask$CompileLibraryResourcesParams_Decorated@6644f036 of type CompileLibraryResourcesTask.CompileLibraryResourcesParams
      > Could not resolve all files for configuration ':location:detachedConfiguration2'.
         > Failed to transform aapt2-7.3.0-8691043-osx.jar (com.android.tools.build:aapt2:7.3.0-8691043) to match attributes {artifactType=_internal-android-aapt2-binary, org.gradle.libraryelements=jar, org.gradle.status=release, org.gradle.usage=java-runtime}.
            > Could not download aapt2-7.3.0-8691043-osx.jar (com.android.tools.build:aapt2:7.3.0-8691043)
               > Could not get resource 'https://dl.google.com/dl/android/maven2/com/android/tools/build/aapt2/7.3.0-8691043/aapt2-7.3.0-8691043-osx.jar'.
                  > Could not GET 'https://dl.google.com/dl/android/maven2/com/android/tools/build/aapt2/7.3.0-8691043/aapt2-7.3.0-8691043-osx.jar'.
                     > The server may not support the client's requested TLS protocol versions: (TLSv1.2, TLSv1.3). You may need to configure the client to allow other protocols to be used. See: https://docs.gradle.org/7.5/userguide/build_environment.html#gradle_system_properties
                        > Remote host terminated the handshake
```

## Solution

After investigate in the official gradle doc, I found the solution below:

update `gradle.properties` and add the following statement:

```properties
https.protocols=TLSv1.1,TLSv1.2,TLSv1.3
```

## References:

* [Build environment](https://docs.gradle.org/current/userguide/build_environment.html)