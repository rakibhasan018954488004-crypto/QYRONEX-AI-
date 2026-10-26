
name: QYRONEX AI APK

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '21'

      - name: Setup Android
        uses: android-actions/setup-android@v3

      - name: Extract ZIP
        run: |
          set -e
          mkdir extracted www
          ZIP=$(find . -maxdepth 1 -name '*.zip' -print -quit)
          test -n "$ZIP"
          unzip -q "$ZIP" -d extracted
          HTML=$(find extracted -type f -name 'index.html' -print -quit)
          test -n "$HTML"
          cp -a "$(dirname "$HTML")/." www/
          test -f www/index.html

      - name: Install Capacitor
        run: |
          npm init -y
          npm install @capacitor/core@7 @capacitor/cli@7 @capacitor/android@7

      - name: Configure Android
        run: |
          echo '{"appId":"com.noman.qyronexai","appName":"QYRONEX AI","webDir":"www"}' > capacitor.config.json
          npx cap add android
          npx cap sync android

      - name: Build APK
        run: |
          cd android
          chmod +x gradlew
          ./gradlew assembleDebug --no-daemon

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: QYRONEX-AI-APK
          path: android/app/build/outputs/apk/debug/*.apk
