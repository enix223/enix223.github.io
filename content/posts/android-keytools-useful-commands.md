---
title: 'Android Keytools Useful Commands'
date: 2024-03-02T09:55:35+08:00
draft: true
---

## Create keystore

```shell
keytool -genkeypair -v -keystore <keystore-name>.keystore -storepass <store_pass> -keypass <key_pass> -keyalg RSA -keysize 2048 -validity 36500 -alias <alias>
```

* Use the same password for both the Key and Keystore.

## Show keystore

```shell
keytool -list -v -keystore packages/app_mobile/android/app/google.keystore    
```

## Print cert for given apk/aab

```shell
keytool -printcert -jarfile YourApp.apk
```

```shell
keytool -printcert -jarfile YourApp.aab
```
