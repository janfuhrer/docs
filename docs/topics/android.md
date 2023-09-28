tags: #android

# Android Programming

links: [[200 Programming Languages MOC|Programming Languages]] - [[000 Index|Index]]

---
## Utilities
- Android Studio

```bash
brew install jadx
brew install apktool
```

## Emulator

```bash
# list emulators (created with Android Studio)
emulator -list-avds

# start emulator
$(which emulator) @Pixel_6_API_33

# start shell
adb shell
```

## Package

```bash
# get files of apk
unzip ${NAME}.apk

# dexdump to see all classes
dexdump classes.dex

# get files with jadx
jadx-gui ${NAME}.akp

# get cert infos
openssl pkcs7 -in ${NAME}.RSA -inform DER -print

# install package
adb install ${NAME}.apk

# decompile apk with apktool
apktool d ${NAME}.apk

# recompile apk (path to directory)
apktool b ${NAME}

# sign package
apksigner sign -ks ${KEYSTORE} ${NAME}/dist/${NAME}.apk

# install apk
adb install ${NAME}/dist/${NAME}.apk
```

---
links: [[200 Programming Languages MOC|Programming Languages]] - [[000 Index|Index]]