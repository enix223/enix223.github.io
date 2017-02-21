---
layout: post
categories: blog
published: true
title: '[Xcode] Create New configuration and scheme with Cocoapod library'
tags: ''
---
## _**Scenario**_

  

I need to create a new app and distribute the app through TestFlight for
external testing. But I don’t want it to use the production release setting,
since we need our user use some preload setting provided by us (e.g., login
user name, and preload data for that user). So I have create some MACRO in the
source code, like that:

    /// FMDB
    #define HB_FMDB_PATH            @“XXXXXXX.db"
    #ifdef TEST
        #define HB_FMDB_INIT_SQL        @"init_test.sql"
    #else
        #define HB_FMDB_INIT_SQL        @"init.sql"
    #endif

We want to compile the source base on the MACRO. And distribute the archive
base on the MACRO too. So we have to leverage the **Configuration** and
**Scheme** in Xcode.

  

## _**Steps**_

  

If you need to create a new configuration like DEBUG or RELEASE, let’s say
TEST, which is special for Testing purpose.

  

You can follow the instructions below to create it:

  

1) Navigate to Project setting

![d5a20c9df3c1021c7432cd8812975efa](/media/d5a20c9df3c1021c7432cd8812975efa.png)

  

2) Open _**Info**_ tab and **_Configuration_** section, and click **+** to
duplicate a new configuration from Debug, and named it to Test![4159369b34f1bd
916c744ba4dbf50c29](/media/4159369b34f1bd916c744ba4dbf50c29.png)

  

3) Navigate back to _**Target -&gt; Build Settings**_, then you can find out
that there will be a **_Test_** configuration available together with DEBUG
&amp; RELEASE occurrence.

  

![91b65b85bf9981e151a348e8c7779b75](/media/91b65b85bf9981e151a348e8c7779b75.png)

  

Now, let’s get one more step, and we need to create a new Scheme for testing
in Testflight, and we want the archive action using the new _**Test**_
configuration, and compile with Preprocessor macro: **TEST=1**

  

4) Create a **_New scheme_**, and give it a name, such as _’XXX_Testflight’,_
and then click **_Edit Scheme..._**

![8bbbd3b34f3e98e7ddf64ce400abcb38](/media/8bbbd3b34f3e98e7ddf64ce400abcb38.png)

  

5) Click **Run** tab, and choose ’**Test**’ in the _**Build configuration**_
dropdown control.

![ab50d47c4c9a3c10bf0eef00cf4006af](/media/ab50d47c4c9a3c10bf0eef00cf4006af.png)

  

6) Click Archive tab, and choose Archive, then select ’Test’ in the Build
configuration dropdown control:

![d54e4d7e09311f038e4ccbd18a360331](/media/d54e4d7e09311f038e4ccbd18a360331.png)

  

7) After finished, close the scheme setting. Let’s build and see the result,
but you may encountered the following error when you using cocoapod library at
the same time.

  

![6348b68313eb8f19eb8337f587f2766c](/media/6348b68313eb8f19eb8337f587f2766c.png)

  

The above message is saying that, the linker command could not find the
AFNetworking library.

  

So let’s have a look at the linker library directory, and see if we can find
the libAFNetworking.a file. As we expect, we can’t find any library in
/Users/enix/Library/Developer/Xcode/DerivedData/HeartBeat-
ggdjgcuqgoxglwetfgphqumpwhqh/Build/Products/Test-iphoneos directory, so what’s
going wrong?

  

Let’s have a look at the build log above, and see what’s happening.

  

Arh.. we get some clues about what’s going on now. That’s because the
AFNetworking library is build and output to Release-iphoneos directory. And
our linker would like to search the library in Test-iphoneos directory, which
is not the same as the library build output place. And then it will fail for
**_Library not found for -lAFNetworking_**.

![63e9c7ac51d906b141193018283e555b](/media/63e9c7ac51d906b141193018283e555b.png)

  

That’s fine, we have found out the reason for the compile issue. So how can we
fix it?

  

That’s because, we just create a configuration in our project setting, and we
did not create a Test configuration under Pod project setting.

  

Now let’s open the Pod project setting as follow, and Create a new
configuration under _**Configuration**_ Section:

![477415672e195328c41a1f99f28f65bf](/media/477415672e195328c41a1f99f28f65bf.png)

  

And be aware of that,  you need to set the name as you used in your project
setting configuration, for example, we use ‘**Test'**, in project setting, so
we need to use ‘**Test'** again for the new Pod configuration.

  

After done, you may got the screen like that:

![08f64b4cb2647fe8944f61f8f82ae3e6](/media/08f64b4cb2647fe8944f61f8f82ae3e6.png)

  

Now, let’s compile again, and see what’s happening:

![aed4108a69701c384ba2e8276d9cd57c](/media/aed4108a69701c384ba2e8276d9cd57c.png)

  

Yowww, build success.

  

## _**Summary**_

  

  1. If you need to use a new configuration, navigate to Project setting -&gt; Info -&gt; Configurations to create a new one
  2. If you also use cocoapod in your project, you need to create a new configuration under Pod project as well
  3. Create a new scheme to manage the Build/Run/Archive with your new configuration.

  

Happy coding… :)
