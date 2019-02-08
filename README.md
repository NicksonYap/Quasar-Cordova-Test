# Quasar Cordova Test

Install Quasar: https://quasar-framework.org/guide/index.html

Install Cordova: https://quasar-framework.org/guide/cordova-preparation.html

Setup:

```
git clone https://github.com/NicksonYap/Quasar-Cordova-Test.git

cd Quasar-Cordova-Test

npm install
```

Live Reload:

Ensure that firewall allows Node.js

```
quasar build -m cordova -T [android|ios]

quasar dev -m cordova -T [android|ios]
```

For iOS, it may be required to use Xcode to open the `.xcodeproj` file and assign a Signing Certificate before running on device

Make sure Xcode uses **Legacy Build System** under `File > Workspace Settings > Build System` and/or `File > Project Settings > Build System`

Run Compiled:

```
cd src-cordova

cordova run [ios|android]
```