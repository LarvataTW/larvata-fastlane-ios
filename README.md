add this to your fastlane/Fastfile

```
import_from_git(
  url: "https://github.com/LarvataTW/larvata-fastlane-ios.git", # The URL of the repository to import the Fastfile from.
  path: "Fastfile" # The path of the Fastfile in the repository. Defaults to fastlane/Fastfile.
)
```
