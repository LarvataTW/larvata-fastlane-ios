# Larvata Fastlane iOS

larvata-fastlane-ios is a pipeline for iOS auto building / deploying on GitLab CI.

## Prerequisite

- On your build machine
  - Use GitLab CI as your CI system.
  - Add a shell-executor gitlab runner with 'macOS' tag
  - Install Xcode on that runner.
  - [Optional] Download other versions Xcode which you need, rename with Xcode-{version}.app format and put into /Applications
    e.g. If you need Xcode 14.1, the path is /Applications/Xcode-14.1.app
- On your local machine
  - Install [bundler](https://bundler.io/)

## Installation

Copy everything in ./example to your project, and modify Appfile and Matchfile.

## Usage

Environment Variables:

| Key                                          | Description                                  | Default    | Optional |
| -------------------------------------------- | -------------------------------------------- | ---------- | -------- |
| XCODE_SCHEME_NAME                            | your xcode scheme name                       | ""         | false    |
| XCODE_PROJECT_NAME                           | your product name                            | ""         | false    |
| MATCH_GC_KEYS                                | gc_keys.json get from GCP IAM                | ""         | false    |
| FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD |                                              | null       | false    |
| PILOT_APPLE_ID                               |                                              | null       | false    |
| XCODE_VERSION                                | which xcode version you use                  | null       | true     |
| FASTLANE_ARCHIVE_METHOD                      | ["adhoc", "appstore"]                        | "adhoc"    | true     |
| FASTLANE_DEPLOY_METHOD                       | ["firebase", "firim", "pgyer", "testflight"] | "firebase" | true     |
| GOOGLE_SERVICE_PLIST_PATH                    | Set if you want send dSYM to firebase        | null       | true     |
| DEPLOY_DISABLED                              |                                              | "false"    | true     |
| STAGING_ENABLED                              |                                              | "true"     | true     |
| FIREBASE_APP                                 | firebase app id                              | null       | true     |
| FIREBASE_CLI_TOKEN                           | firebase cli token                           | null       | true     |
| FIREBASE_GROUPS                              | firebase app distribution groups             | null       | true     |
| FIRIM_API_TOKEN                              | firim api token                              | null       | true     |
| PGYER_API_KEY                                | pgyer api key                                | null       | true     |

## Contributing

Pull requests are welcome. For major changes, please open an issue first
to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License

[MIT](https://choosealicense.com/licenses/mit/)
