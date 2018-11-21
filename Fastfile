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
  before_all do
    unless ENV['SLACK_URL'].nil? || ENV['SLACK_CHANNEL'].nil? || ENV['XCODE_SCHEME'].nil?
      slack(
        message: ENV['XCODE_SCHEME'] + " start fastlane.",
        channel: ENV['SLACK_CHANNEL'],
      )
    end
  end

  desc "Runs all the tests"
  lane :test do
    scan
  end
  
  private_lane :archive do
    match(type: "adhoc", clone_branch_directly: true)
    
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
  end

  desc "Submit a new Beta Build to Fabric"
  desc "This will also make sure the profile is up to date"
  lane :beta_fabric do |options|
    if !options[:skip_setup_circle_ci]
      setup_circle_ci
    end
    increment_build_number(build_number: ENV['CIRCLE_BUILD_NUM'])
    archive
    crashlytics(api_token: ENV['FABRIC_API_TOKEN'], build_secret: ENV['FABRIC_BUILD_SECRET'])
    upload_symbols_to_crashlytics(api_token: ENV['FABRIC_API_TOKEN'])
  end

  desc "Submit a new Beta Build to fir.im"
  desc "This will also make sure the profile is up to date"
  lane :beta_firim do |options|
    if !options[:skip_setup_circle_ci]
      setup_circle_ci
    end
    increment_build_number(build_number: ENV['CIRCLE_BUILD_NUM'])
    archive
    firim(firim_api_token: ENV['FIRIM_API_TOKEN'])
    unless ENV['FABRIC_API_TOKEN'].nil?
      upload_symbols_to_crashlytics(api_token: ENV['FABRIC_API_TOKEN'])
    end
  end

  desc "Submit a new Beta Build to Pgyer"
  desc "This will also make sure the profile is up to date"
  lane :beta_pgyer do |options|
    if !options[:skip_setup_circle_ci]
      setup_circle_ci
    end
    increment_build_number(build_number: ENV['CIRCLE_BUILD_NUM'])
    archive
    pgyer(api_key: ENV['PGYER_API_KEY'], user_key: ENV['PGYER_USER_KEY'])
    unless ENV['FABRIC_API_TOKEN'].nil?
      upload_symbols_to_crashlytics(api_token: ENV['FABRIC_API_TOKEN'])
    end
  end

  desc "Submit a new Beta Build to Apple TestFlight"
  desc "This will also make sure the profile is up to date"
  lane :beta_testflight do |options|
    if !options[:skip_setup_circle_ci]
      setup_circle_ci
    end
    increment_build_number(build_number: ENV['CIRCLE_BUILD_NUM'])
    match(type: "appstore", clone_branch_directly: true)
    gym(scheme: ENV['XCODE_SCHEME'], skip_profile_detection: true) # Build your app - more options available
    pilot(skip_waiting_for_build_processing: true)
  end

  desc "Deploy a new version to the App Store"
  lane :release do |options|
    if !options[:skip_setup_circle_ci]
      setup_circle_ci
    end
    increment_build_number(build_number: ENV['CIRCLE_BUILD_NUM'])
    match(type: "appstore", clone_branch_directly: true)
    # snapshot
    gym(scheme: ENV['XCODE_SCHEME'])# Build your app - more options available
    deliver(force: true,
            automatic_release: true,
            overwrite_screenshots: true,
            run_precheck_before_submit: false)
    # frameit
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

  # You can define as many lanes as you want

  after_all do |lane|
    # This block is called, only if the executed lane was successful
    unless ENV['SLACK_URL'].nil? || ENV['SLACK_CHANNEL'].nil? || ENV['XCODE_SCHEME'].nil?
      slack(
        message: ENV['XCODE_SCHEME'] + " successfully released!",
        channel: ENV['SLACK_CHANNEL'],
        success: true,
        payload: {
          "Build Date" => Time.new.to_s,
          "Built by" => ENV['USER'],
        },
        default_payloads: [:git_branch, :git_author],
        attachment_properties: {
          fields: [{
            title: "Build Version",
            value: get_version_number(target: ENV['XCODE_SCHEME']),
            short: true
          },{
            title: "Build Number",
            value: get_build_number,
            short: true
          }]
        }
      )
    end
  end

  error do |lane, exception|
    unless ENV['SLACK_URL'].nil? || ENV['SLACK_CHANNEL'].nil?
      slack(
        message: exception.message,
        channel: ENV['SLACK_CHANNEL'],
        success: false
      )
    end
  end
end


# More information about multiple platforms in fastlane: https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Platforms.md
# All available actions: https://docs.fastlane.tools/actions

# fastlane reports which actions are used. No personal data is recorded. 
# Learn more at https://github.com/fastlane/fastlane#metrics
