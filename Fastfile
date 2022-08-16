# Customise this file, documentation can be found here:
# https://github.com/fastlane/fastlane/tree/master/fastlane/docs
# All available actions: https://docs.fastlane.tools/actions
# can also be listed using the `fastlane actions` command

# Change the syntax highlighting to Ruby
# All lines starting with a # are ignored when running `fastlane`

# If you want to automatically update fastlane if a new version is available:
# update_fastlane

# This is the minimum version number required.
# Update this, if you use features of a newer version
fastlane_version "2.62.0"

default_platform :ios

platform :ios do
  
  desc "Runs all the tests"
  lane :test do
    scan
  end
  
  lane :archive_adhoc do |options|
    begin
      # 設定Keychain
      if !options[:skip_setup_circle_ci]
        setup_circle_ci
      end
      xcode_select("/Applications/Xcode#{ENV['XCODE_VERSION'].nil? ? "" : "-" + ENV['XCODE_VERSION']}.app")
      # 將 Pipeline ID 設定到build number
      increment_build_number(build_number: ENV['CI_PIPELINE_IID'])
      # 同步憑證
      match(type: "adhoc", clone_branch_directly: true, readonly: true)

      project = Xcodeproj::Project.open(Dir["../*.xcodeproj"].first)
      # 將 AppStore 替換成 AdHoc
      project.build_configurations.each do |config|
        unless config.build_settings['PROVISIONING_PROFILE_SPECIFIER'].nil?
          specifier = config.build_settings['PROVISIONING_PROFILE_SPECIFIER']
          config.build_settings['PROVISIONING_PROFILE_SPECIFIER'] = specifier.sub 'AppStore', 'AdHoc'
        end
      end
      project.targets.each do |target|
        target.build_configurations.each do |config|
          unless config.build_settings['PROVISIONING_PROFILE_SPECIFIER'].nil?
            specifier = config.build_settings['PROVISIONING_PROFILE_SPECIFIER']
            config.build_settings['PROVISIONING_PROFILE_SPECIFIER'] = specifier.sub 'AppStore', 'AdHoc'
          end
        end
      end
      project.save

      gym(scheme: ENV['XCODE_SCHEME'],
        export_method: "ad-hoc",
        skip_profile_detection: true,
        export_options: {
          compileBitcode: false
        }
      )
    ensure
      unless Helper.ci?
        UI.message("Not running on CI, skipping delete_keychain")
        next
      end
      delete_keychain(name: ENV['MATCH_KEYCHAIN_NAME'])
    end
  end
  
  lane :archive_appstore do |options|
    begin
      if !options[:skip_setup_circle_ci]
        setup_circle_ci
      end
      xcode_select("/Applications/Xcode#{ENV['XCODE_VERSION'].nil? ? "" : "-" + ENV['XCODE_VERSION']}.app")
      if ENV['CI_COMMIT_TAG'].include? "+"
        increment_build_number(build_number: ENV['CI_COMMIT_TAG'].rpartition('+').last)
      else
        increment_build_number(build_number: ENV['CI_PIPELINE_IID'])
      end
      match(type: "appstore", clone_branch_directly: true, readonly: true)
      gym(scheme: ENV['XCODE_SCHEME'], skip_profile_detection: true) # Build your app - more options available
    ensure
      unless Helper.ci?
        UI.message("Not running on CI, skipping delete_keychain")
        next
      end
      delete_keychain(name: ENV['MATCH_KEYCHAIN_NAME'])
    end
  end
  
  desc "Submit a new Beta Build to Firebase"
  desc "This will also make sure the profile is up to date"
  lane :beta_firebase do |options|
    if ENV['CI_COMMIT_BEFORE_SHA'] == '0000000000000000000000000000000000000000'
      firebase_app_distribution(
        app: ENV['FIREBASE_APP'],
        firebase_cli_token: ENV['FIREBASE_CLI_TOKEN'],
        groups: ENV['FIREBASE_GROUPS']
      )
    else
      firebase_app_distribution(
        app: ENV['FIREBASE_APP'],
        firebase_cli_token: ENV['FIREBASE_CLI_TOKEN'],
        groups: ENV['FIREBASE_GROUPS'],
        release_notes: sh("git log --format='%h %s%n%b' --no-merges #{ENV['CI_COMMIT_BEFORE_SHA']}...@")
      )
    end
    unless ENV['GOOGLE_SERVICE_PLIST_PATH'].nil?
      upload_symbols_to_crashlytics(
        dsym_path: "#{ENV['XCODE_PRODUCT_NAME']}*.app.dSYM.zip",
        gsp_path: ENV["GOOGLE_SERVICE_PLIST_PATH"]
      )
    end
  end

  desc "Submit a new Beta Build to fir.im"
  desc "This will also make sure the profile is up to date"
  lane :beta_firim do |options|
    firim(firim_api_token: ENV['FIRIM_API_TOKEN'])
    unless ENV['GOOGLE_SERVICE_PLIST_PATH'].nil?
      upload_symbols_to_crashlytics(
        dsym_path: "#{ENV['XCODE_PRODUCT_NAME']}*.app.dSYM.zip",
        gsp_path: ENV["GOOGLE_SERVICE_PLIST_PATH"]
      )
    end
  end

  desc "Submit a new Beta Build to Pgyer"
  desc "This will also make sure the profile is up to date"
  lane :beta_pgyer do |options|
    pgyer(api_key: ENV['PGYER_API_KEY'], user_key: ENV['PGYER_USER_KEY'])
    unless ENV['GOOGLE_SERVICE_PLIST_PATH'].nil?
      upload_symbols_to_crashlytics(
        dsym_path: "#{ENV['XCODE_PRODUCT_NAME']}*.app.dSYM.zip",
        gsp_path: ENV["GOOGLE_SERVICE_PLIST_PATH"],
      )
    end
  end

  desc "Submit a new Beta Build to Apple TestFlight"
  desc "This will also make sure the profile is up to date"
  lane :beta_testflight do |options|
    begin
      if !options[:skip_setup_circle_ci]
        setup_circle_ci
      end
      xcode_select("/Applications/Xcode#{ENV['XCODE_VERSION'].nil? ? "" : "-" + ENV['XCODE_VERSION']}.app")
      pilot(skip_waiting_for_build_processing: true)
    ensure
      unless Helper.ci?
        UI.message("Not running on CI, skipping delete_keychain")
        next
      end
      delete_keychain(name: ENV['MATCH_KEYCHAIN_NAME'])
    end
  end

  desc "Generate devices.txt"
  lane :add_device do
    sh('echo "Device ID	Device Name" >> ../devices.txt')
    sh('echo "A123456789012345678901234567890123456789	NAME1" >> ../devices.txt')
    sh('echo "B123456789012345678901234567890123456789	NAME2" >> ../devices.txt')
  end

  lane :register_a_device do
    register_devices(
      devices_file: "./devices.txt"
    )
    refresh_profiles
    sh('rm -f ../devices.txt')
  end
  
  lane :register_a_device_without_apple_certs do
    register_devices(
      devices_file: "./devices.txt"
    )
    refresh_profiles_without_apple_certs
    sh('rm -f ../devices.txt')
  end

  # A helper lane for refreshing provisioning profiles.
  lane :refresh_profiles do
    match(
      type: "development",
      force: true,
      clone_branch_directly: true)
    match(
      type: "adhoc",
      force: true,
      clone_branch_directly: true)
  end
  
  # A helper lane for refreshing provisioning profiles.
  lane :refresh_profiles_without_apple_certs do
    match(
      type: "development",
      force: true,
      generate_apple_certs: false)
    match(
      type: "adhoc",
      force: true,
      generate_apple_certs: false)
  end
end


# More information about multiple platforms in fastlane: https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Platforms.md
# All available actions: https://docs.fastlane.tools/actions

# fastlane reports which actions are used. No personal data is recorded. 
# Learn more at https://github.com/fastlane/fastlane#metrics
