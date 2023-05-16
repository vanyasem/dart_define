# dart_define

![[CI]](https://github.com/Mankeli-Software/dart_define/actions/workflows/flutter_ci.yaml/badge.svg)
[![style: mankeli analysis](https://img.shields.io/badge/style-mankeli__analysis-blue)](https://pub.dev/packages/mankeli_analysis)

[!["Buy Me A Coffee"](https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png)](https://www.buymeacoffee.com/mankeli)

## Motivation 🔥

`dart_define` is an all-in-one tool to handle secrets and launch configurations both locally and in CI/CD pipelines super easily. Say goodbye to `.env` files or any other
tedious and error-prone ways. With this package you can access your secrets in dart code as well as platform specific configurations.

Flutter supports adding compile-time variables with `--dart-define` option, and using them in both dart code and native configurations. However, with many variables,
the commands become quite long quite fast. A workaround for that would be to create a custom launch shell scripts, or storing variables in a JSON file and using it with `--dart-define-from-file` option.

Hovewer, that raises a ton of other issues within CI/CD pipelines. You'd have to gitignore the launch config files, so you don't leak any secrets unwantedly. Then you'd have to create some custom script to initialize and fill those files with proper values in your CI/CD environments. Even after working your way around those issues, you'd have to be very careful to always remember update those pipelines when new values are added (missing values are not recognized at compile-time, leading to possible run-time errors in production).

With `dart_define`, you'll get easy ways of handling secrets and configurations in both local development environment as well as CI/CD pipelines. You can specify which variables are required, so the CLI tool will throw an exception if those are missing. This way missing values will result in failed pipeline run, not run-time error in production.

## Quick Start 🚀

1. Install the CLI tool, OR add it into dependencies

    ```sh
    # activate the CLI tool
    dart pub global activate dart_define
    ```

    OR

    ```sh
    # add the tool to dependencies
    flutter pub add --dev dart_define
    ```

2. Define your variables in `pubspec.yaml`

    ```yaml
    dart_define:

        # OPTIONAL: Whether to generate the dart boilerplate or not. 
        # Defaults to true
        dart: true 

        # OPTIONAL: Whether to generate the json boilerplate or not. 
        # Defaults to true
        json: true 

        # REQUIRED: The variables to add to the generated config files
        variables: 

              # REQUIRED: The name of the variable
            - name: STRING_VALUE 

              # REQUIRED: The description of the variable
              description: An example String value 

              # REQUIRED: If `required` is false, this field is mandatory. 
              # Otherwise, it is ignored.
              default: hello world! 

              # OPTIONAL: If this is set to true, the CLI tool will 
              # throw an exception if the variable is not provided 
              # as an argument. This is handy when using the tool
              # in CI/CD pipelines.
              # Defaults to true.
              required: false 
            - name: INT_VALUE
              description: An example int value
              default: 3
              required: false
            - name: BOOL_VALUE
              description: An example bool value
              required: true
    ```

3. Generate boilerplate

    ```sh
    # If you activated the CLI tool
    dart_define generate
    ```

    OR

    ```sh
    # If you added the tool to dependencies
    dart run dart_define generate
    ```

   *NOTE: You can override values and variables from pubspec.yaml config with CLI arguments*

    <!-- markdownlint-disable -->
    <img src="doc/asset/usage.png" width="50%" title="usage" alt="usage">
    <!-- markdownlint-enable -->

4. Use your variables in dart code

   This tool generates dart boilerplate, so you can access the values very easily.

   ```dart
    /// This is the code this tool creates
    class DartDefine {
        /// An example String value
        static const stringValue = String.fromEnvironment('STRING_VALUE');

        /// An example int value
        static const intValue = int.fromEnvironment('INT_VALUE');

        /// An example bool value
        static const boolValue = bool.fromEnvironment('BOOL_VALUE');
    }

    /// With this you can access the variables like so:
    String testValue = DartDefine.stringValue;
   ```

5. Access the values in platforms native configurations (see [this article][1])

    Within android build.gradle

    ```gradle
    // In app/build.gradle
    defaultConfig {
        applicationId STRING_VALUE

        // This allows us to access the value in eg. AndroidManifest.xml
        resValue "string", "STRING_VALUE", STRING_VALUE
    }
    ```

    Within AndroidManifest.xml

    ```xml
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="@string/STRING_VALUE">
    </manifest>
    ```

    Within iOS config files

    ```xcconfig
    $(STRING_VALUE)
    ```

6. Run the app from the configuration file

   ```sh
   flutter run --dart-define-from-file=dart_define.json
   ```

7. Add your configurations to .gitignore (highly recommended if configs contain secrets)

    ```gitignore
    dart_define.json
    ```

8. Configure your CI/CD pipeline

    Add custom shell script or run command before build
      <!-- markdownlint-disable -->
    ```sh
    # Notice how this takes the secrets from the CI/CD environment secrets and populates the dart_define.json based on those
    dart run dart_define generate --BOOL_VALUE=${BOOL_VALUE} --STRING_VALUE=${STRING_VALUE} --INT_VALUE=${INT_VALUE}
    flutter build apk --dart-define-from-file=dart_define.json
    ```
      <!-- markdownlint-enable -->

[1]: https://itnext.io/flutter-3-7-and-a-new-way-of-defining-compile-time-variables-f63db8a4f6e2
