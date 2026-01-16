# KMP Publish iOS App to App Store

This GitHub Action automates the process of building and distributing an iOS application to the App
Store using Fastlane. It supports secure code signing via Fastlane Match, handles build number
incrementing, metadata management, and uploads the app to the App Store via App Store Connect API
keys.

## Configuration

iOS configuration inputs are optional (except `metadata_path`). Configuration is read from
`fastlane-config/project_config.rb` in the project repository when parameters are not provided.

**Supported configuration methods:**

- Explicit parameters (backward compatible with previous versions)
- Automatic extraction from `project_config.rb`

See usage examples below for implementation details.

---

Start from local first. If it works locally, it will definitely work on CI.

<hr data-start="378" data-end="381" class="">

## 1. Local Setup (Required Once)

1. **Install Ruby**  
   [https://www.ruby-lang.org/en/documentation/installation/](https://www.ruby-lang.org/en/documentation/installation/)

2. **Install Fastlane**  
   [https://docs.fastlane.tools/getting-started/ios/setup/](https://docs.fastlane.tools/getting-started/ios/setup/)

```
ruby -v
fastlane -v
```

<hr data-start="378" data-end="381" class="">

## 2. Apple Developer Setup

- Go to [Apple Developer Console](https://developer.apple.com/account).

- Make sure you have at least one registered device:  
  https://developer.apple.com/account/resources/devices/list

<hr data-start="378" data-end="381" class="">

## 3. Set Up Fastlane in Your KMP iOS Project

In your terminal: (inside your root project)

```
fastlane init
```

When prompted:

1. Choose option:
   `3. Automate App Store distribution`

2. Apple ID Email:
   Type your Apple Developer email (e.g., yourname@icloud.com).

3. App Identifier:
   If it doesn't exist, you'll see:

```
Checking if the app 'org.your.bundle.id' exists in your Apple Developer Portal..
It looks like the app 'org.your.bundle.id' isn't available on the Apple Developer Portal
Do you want fastlane to create the App ID for you on the Apple Developer Portal? (y/n)
```

Type y
Then enter App Name (can be anything, just make sure it's unique)

4. Then:

```
Checking if the app 'org.your.bundle.id' exists on App Store Connect..
It looks like the app isn't available on App Store Connect
Do you want fastlane to create the App on App Store Connect for you? (y/n)
```

You can type y, but this step sometimes fails.

If it fails, go manually:  https://appstoreconnect.apple.com/apps → + New App → Fill required fields
Then return to the terminal and re-run the setup.

5. Manage App Metadata: Fastlane will ask:

```
Would you like fastlane to manage your app's metadata? (y/n)
```

Type y. This will create a metadata folder with .txt files for app details (e.g., name, description,
keywords) and download existing screenshots into a screenshots folder.

<hr data-start="378" data-end="381" class="">

## 4. Generated Files

After init, you’ll see:

```
rootProject/
└── fastlane/
    ├── metadata/
    ├── screenshots/
    ├── Appfile
    ├── Deliverfile
    └── Fastfile
```

- Appfile: Stores basic metadata like your Apple ID and app identifier.
- Deliverfile: Configures metadata and screenshot delivery to the App Store.
- Fastfile: Defines the automation workflow for App Store distribution.
- metadata/: Contains .txt files for app metadata (e.g., name, description).
- screenshots/: Contains downloaded screenshots from App Store Connect.

<hr data-start="378" data-end="381" class="">

## 5. App and Screenshot Setup

**App Publishing Details**

For detailed guidance on preparing your app for the App Store, including metadata, app details, and
submission requirements, refer to:
https://help.apple.com/app-store-connect/en.lproj/static.html#dev354659071

This includes:

- Filling out app metadata (name, description, keywords, etc.) in the metadata/ folder.
- Providing app information like privacy policy, support URL, and categories.
- Preparing your app for review (e.g., demo account details, review notes).

**Screenshot Specifications**

Screenshots must meet Apple's requirements for size and quantity. For detailed specifications, refer
to:
https://developer.apple.com/help/app-store-connect/reference/screenshot-specifications

**Requirements:**

- Quantity: At least 1 screenshot per device type, with a maximum of 10 per device type.
- Devices to Support: Create screenshots for iPhone, iPad, and Apple Watch.
- Dimensions:
    - iPhone 6.5-inch display: 1242 × 2688 pixels (or 2688 × 1242 for landscape).
    - iPhone 6.7-inch display: 1290 × 2796 pixels (or 2796 × 1290 for landscape).
    - iPad Pro (12.9-inch, 3rd Gen): 2048 × 2732 pixels (or 2732 × 2048 for landscape).
    - Apple Watch Ultra: 410 × 502 pixels.

**Steps to Create Screenshots:**

1. Capture screenshots for each device type using a simulator or real device.
2. Ensure the screenshots meet the exact dimensions listed above.
3. Name the screenshots following this convention (good practice):
   `[index]_APP_[DEVICE_TYPE]_[DISPLAY/GEN]_[RESOLUTION]_[INDEX].png`.
    - Example: `0_APP_IPHONE_65_0.png`, `1_APP_IPAD_PRO_3GEN_129_1.png`.
4. Place the screenshots in a localized folder inside `screenshots/`, such as `screenshots/en-GB/`.

Example Screenshot Structure:
For `en-GB` localization, your `screenshots/en-GB/` folder might look like this (with at least 1
screenshot per device type, up to 10):

```
screenshots/
└── en-GB/
├── 0_APP_IPHONE_65_0.png    # iPhone 6.5-inch, 1242 × 2688
├── 0_APP_IPHONE_67_0.png    # iPhone 6.7-inch, 1290 × 2796
├── 0_APP_IPAD_PRO_3GEN_129_0.png  # iPad Pro 12.9-inch, 2048 × 2732
├── 0_APP_WATCH_ULTRA_0.png  # Apple Watch Ultra, 410 × 502
├── 1_APP_IPHONE_65_1.png
├── 1_APP_IPHONE_67_1.png
├── 1_APP_IPAD_PRO_3GEN_129_1.png
├── 1_APP_WATCH_ULTRA_1.png
├── 2_APP_IPHONE_65_2.png
├── 2_APP_IPHONE_67_2.png
├── 2_APP_IPAD_PRO_3GEN_129_2.png
├── 2_APP_WATCH_ULTRA_2.png
```

## 6. Configure Code Signing with fastlane match

Now let’s set up automatic provisioning and code signing with Fastlane Match.

Step-by-step:

1. Run Match Init

```
fastlane match init
```

2. Choose Storage Type

It will ask:
<blockquote data-start="293" data-end="376">
<p data-start="295" data-end="376" class=""><strong>fastlane match</strong> supports multiple storage options. Please select the one you want to use:.</p>
</blockquote>

```
1. git
2. google_cloud
3. s3
4. gitlab_secure_files
```

Type 1 and press enter.

We’ll store certificates and provisioning profiles in a private Git repository.

3. Create a Private Git Repo

- Go to GitHub and create a new private repository (e.g., `your-app-match-certificates`).

- Copy the HTTPS URL (e.g., `https://github.com/your-org/your-app-match-certificates.git`).

- Paste it in the terminal when prompted:

<blockquote data-start="293" data-end="376">
<p data-start="295" data-end="376" class="">URL of the Git Repo:<br>
https://github.com/your-org/your-app-match-certificates.git.</p>
</blockquote>

4. Edit Matchfile

Open the newly created Matchfile in your iosApp/fastlane/ folder. You’ll see:

```
type("development")
```

Replace "development" with "appstore" since we’re deploying to TestFlight:

```
type("appstore")
```

<hr data-start="378" data-end="381" class="">

## 7. Generate Certificates & Profiles

Now we’ll actually create and fetch the necessary provisioning profile and certificate.

```
fastlane match appstore
```

It will ask:

```
Enter the passphrase that should be used to encrypt/decrypt your certificates
```

Create a secure passphrase (save it somewhere!). This will encrypt your certificates in the Git
repo.

You’ll later add this to your GitHub repo as a secret:

```
MATCH_PASSWORD=your_passphrase
```

<hr data-start="378" data-end="381" class="">

## 8. Verify on Apple Developer Portal

After the above, Fastlane will automatically:

- Create a Distribution Certificate
  at [Certificates List](https://developer.apple.com/account/resources/certificates/list)

- Create a Provisioning Profile
  at [Profiles List](https://developer.apple.com/account/resources/profiles/list)

<hr data-start="378" data-end="381" class="">

## 9. Xcode Configuration

Now link the generated provisioning profile to your project manually:

1. Open iosApp.xcodeproj in Xcode.

2. Click your project in the navigator (top left).

3. Under Targets, click your app.

4. Go to the Signing & Capabilities tab.

5. Uncheck “Automatically manage signing”

6. From Provisioning Profile, select the one that was created via fastlane match.

<hr data-start="378" data-end="381" class="">

## 10. Create App Store Connect API Key

1. Go to [Users and Access](https://appstoreconnect.apple.com/access/api) → Keys

2. Click + to generate a new key

3. You'll get:

- Key ID

- Issuer ID

- Private key (.p8 file)

Save them somewhere secure.

<hr data-start="378" data-end="381" class="">

## 11. Add the Private Key to Project

Place your .p8 file inside:

```
secrets/Api_key.p8
```

Add secrets/Api_key.p8 to .gitignore (probably it will directly be added in `.gitignore`)

<hr data-start="378" data-end="381" class="">

## 12. Step-by-Step: SSH Setup for Fastlane Match

1. Generate SSH Key Locally<br>
   In your terminal, run:

```yaml
ssh-keygen -t ed25519 -C "your_email@example.com"
```

It will ask for a file path. Press enter a custom path like:

```yaml
~/.ssh/match_ci_key
```

You can skip setting a passphrase when prompted (just hit enter twice).

This generates two files:

- `~/.ssh/match_ci_key` → Private key

- `~/.ssh/match_ci_key.pub` → Public key

2. Add the Private Key to the SSH Agent (optional but helpful)
   This step ensures the key is used during local development.

```yaml
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/match_ci_key
```

3. Add the Public Key to Your Certificates Repo (GitHub)

- Go to your certificates repo on GitHub (e.g., openMF/ios-provisioning-profile).
- Go to Settings → Deploy Keys.
- Click “Add deploy key”.
- Set title as `CI Match SSH Key` and paste the content of:

```yaml
cat ~/.ssh/match_ci_key.pub
```

- Check **Allow write access**.
- Click Add key.

4. Convert the Private Key to Base64
   This is how we pass it to GitHub Actions securely.

```yaml
base64 -i ~/.ssh/match_ci_key | pbcopy
```

This command copies the base64-encoded private key to your clipboard (macOS).

5. Save the Private Key as a GitHub Secret

- Go to the repo with your Fastfile (the project repo, not the certs repo).
- Navigate to Settings → Secrets and variables → Actions → New repository secret.
- Add the following:
    - Name: MATCH_SSH_PRIVATE_KEY
    - Value: Paste the base64-encoded private key

<hr data-start="378" data-end="381" class="">

## 13. Fastfile

```yaml
default_platform(:ios)

  platform :ios do
  desc "Upload iOS Application to AppStore"
  lane :release do |options|
  # Flexible parameter resolution: CLI option → ENV fallback
  key_id = options[:appstore_key_id] || ENV["APPSTORE_KEY_ID"]
  issuer_id = options[:appstore_issuer_id] || ENV["APPSTORE_ISSUER_ID"]
  key_filepath = options[:key_filepath] || ENV["KEY_FILEPATH"] || "./secrets/Api_key.p8"
  match_type = options[:match_type] || ENV["MATCH_TYPE"]
  app_identifier = options[:app_identifier] || ENV["APP_IDENTIFIER"]
  git_url = options[:git_url] || ENV["GIT_URL"]
  git_branch = options[:git_branch] || ENV["GIT_BRANCH"]
  match_password = options[:match_password] || ENV["MATCH_PASSWORD"]
  git_private_key = options[:git_private_key] || ENV["GIT_PRIVATE_KEY"] || "./secrets/match_ci_key"
  metadata_path = options[:metadata_path] || "./fastlane/metadata"

  # Prepares the Fastlane environment for use in CI systems
  setup_ci(
provider: "circleci"
  )

  # Create an App Store Connect API key using either passed options or environment variables
  api_key = app_store_connect_api_key(
key_id: key_id,
issuer_id: issuer_id,
key_filepath: key_filepath
  )

  # Downloads code signing credentials using the API key
  match(
type: match_type,
app_identifier: app_identifier,,
readonly: true,
git_url: git_url,
git_branch: git_branch,
api_key: api_key,
git_private_key: git_private_key
  )

  # Retrieves the most recent TestFlight build number
  latest = latest_testflight_build_number(
app_identifier: app_identifier,
api_key: api_key
  )

  # Increment build number locally
  increment_build_number(
build_number: latest + 1,
xcodeproj: "iosApp/iosApp.xcodeproj"
  )

  # Build ios app
  build_ios_app(
scheme: "iosApp",
project: "iosApp/iosApp.xcodeproj",
output_name: "DeployIosApp.ipa",
output_directory: "iosApp/build",
  )

  # Upload metadata and screenshots
  deliver(
metadata_path: metadata_path,
submit_for_review: false, # Set to true if you want to auto-submit for review
automatic_release: faslse, # Set to true if you want to auto-release once it approved
api_key: api_key,
skip_app_version_update: true,
force: true, # Skips HTML report verification
precheck_include_in_app_purchases: false
  )
  end
  end
```

<hr data-start="378" data-end="381" class="">

## 14. Test Locally

```
bundle exec fastlane ios release
```

<hr data-start="378" data-end="381" class="">

## deploy-ios.yml (Caller)

### Configuration from project_config.rb

```yaml
name: Deploy iOS to App Store

on:
  workflow_dispatch:

jobs:
  deploy:
    runs-on: macos-latest

    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Deploy to App Store
        uses: openMF/mifos-x-actionhub-publish-ios-on-appstore@v2.0.0
        with:
          metadata_path: './fastlane/metadata'
          # Configuration read from fastlane-config/project_config.rb
          # Only secrets needed:
          appstore_key_id: ${{ secrets.APPSTORE_KEY_ID }}
          appstore_issuer_id: ${{ secrets.APPSTORE_ISSUER_ID }}
          appstore_auth_key: ${{ secrets.APPSTORE_AUTH_KEY }}
          match_password: ${{ secrets.MATCH_PASSWORD }}
          match_ssh_private_key: ${{ secrets.MATCH_SSH_PRIVATE_KEY }}
```

### Explicit Parameter Configuration

```yaml
name: Deploy iOS to App Store

on:
  workflow_dispatch:

jobs:
  deploy:
    runs-on: macos-latest

    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Deploy to App Store
        uses: openMF/mifos-x-actionhub-publish-ios-on-appstore@v2.0.0
        with:
          # Explicit configuration:
          app_identifier: 'org.your.bundle.id'
          git_url: 'git@github.com:your-org/your-app-certificates.git'
          git_branch: 'master'
          match_type: 'appstore'
          provisioning_profile_name: 'match AppStore org.your.bundle.id'
          metadata_path: './fastlane/metadata'
          # Secrets:
          appstore_key_id: ${{ secrets.APPSTORE_KEY_ID }}
          appstore_issuer_id: ${{ secrets.APPSTORE_ISSUER_ID }}
          appstore_auth_key: ${{ secrets.APPSTORE_AUTH_KEY }}
          match_password: ${{ secrets.MATCH_PASSWORD }}
          match_ssh_private_key: ${{ secrets.MATCH_SSH_PRIVATE_KEY }}
```

<hr data-start="378" data-end="381" class="">

## GitHub Secrets Required

 Secret                | Description                       
-----------------------|-----------------------------------
 APPSTORE_KEY_ID       | From App Store Connect API        
 APPSTORE_ISSUER_ID    | From App Store Connect API        
 APPSTORE_PRIVATE_KEY  | Paste full .p8 content here       
 MATCH_PASSWORD        | Password for encrypted match repo 
 MATCH_SSH_PRIVATE_KEY | SSH private key for match repo    
