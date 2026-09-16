name: Build Sathi AI APK

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Extract project ZIP
        run: |
          unzip -q *.zip -d extracted
          find extracted -maxdepth 3 -type f | head -50

      - name: Find Android project
        run: |
          PROJECT=$(find extracted -name settings.gradle.kts -o -name settings.gradle | head -1)
          echo "Found: $PROJECT"
          echo "PROJECT_DIR=$(dirname "$PROJECT")" >> "$GITHUB_ENV"

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - name: Setup Gradle
        uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: '8.7'

      - name: Setup Android SDK
        uses: android-actions/setup-android@v3

      - name: Build APK
        working-directory: ${{ env.PROJECT_DIR }}
        run: |
          gradle assembleDebug --stacktrace

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: Sathi-AI-APK
          path: ${{ env.PROJECT_DIR }}/app/build/outputs/apk/debug/*.apk
