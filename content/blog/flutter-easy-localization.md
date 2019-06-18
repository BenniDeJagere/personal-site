---
title: "Easy localization in Flutter wih continuous integration"
date: 2019-06-17T22:08:59+02:00
draft: true
---

Did you know [Flutter](https://flutter.dev/) has built-in support for l10n, also known as localization? I've been searching for an easy way to localize my app, but most of the blog posts I've found rephrased [the official documentation on internationalization](https://flutter.dev/docs/development/accessibility-and-localization/internationalization).

During my job as a Xamarin developer, I regularly use [Loco](https://localise.biz/), a software translation management platform. Turns out this also makes localizing Flutter apps a piece of cake.

In this tutorial I'll walk you through setting up l10n for your app and configuring a system that both developers and translators can use to localize your app with ease.

## Add dependencies

To get started with l10n, add the `flutter_localizations` package to your `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations: # add the package
    sdk: flutter # make it target the right SDK
  ...
```

Flutter comes bundled with the `en_US` locale and this package adds support for more than 50 languages.

We will be using Dart's official [Intl](https://pub.dev/packages/intl) package to make localizing strings a breeze, so add this package to your development dependencies.

```yaml
dev_dependencies:
  ...
  intl_translation: ^0.17.5
```

Run `flutter packages get` to make sure the dependencies are installed.

## Define a class for localized resources

Just like in the official documentation, we'll add a class that contains the logic to load the translations.

```Dart
class AppLocalizations {
  static Future<AppLocalizations> load(Locale locale) {
    final String name = locale.countryCode.isEmpty ? locale.languageCode : locale.toString();
    final String localeName = Intl.canonicalizedLocale(name);
    return initializeMessages(localeName).then((_) {
      Intl.defaultLocale = localeName;
      return AppLocalizations();
    });
  }

  static AppLocalizations of(BuildContext context) {
    return Localizations.of<AppLocalizations>(context, AppLocalizations);
  }
}
```

This class loads the current locale from the device. It tries to find a locale with a matching country code, otherwise it falls back on the language.

The static `of` method allows us to instantiate the class based on a context from anywhere inside a `build` method.

Don't worry about the missing `initializeMessages` method for now as we will automatically generate this.

## Integrate with Flutter's built-in l10n support

Both `MaterialApp` and `CupertinoApp` have arguments that take in a `LocalizationsDelegate`. This delegate is the glue between our `AppLocalizations` and the app.

```Dart
class AppLocalizationsDelegate extends LocalizationsDelegate<AppLocalizations> {
  const AppLocalizationsDelegate();

  @override
  bool isSupported(Locale locale) {
    return ['en', 'nl', 'fr'].contains(locale.languageCode);
  }

  @override
  Future<AppLocalizations> load(Locale locale) {
    return AppLocalizations.load(locale);
  }

  @override
  bool shouldReload(LocalizationsDelegate<AppLocalizations> old) {
    return false;
  }
}
```

The `load` simply returns our `AppLocalizations` class as it will contain all the localized resources.

In the `isSupported` method you can define which locales your app should support (English, Dutch and French in this example).

The `shouldReload` method always returns `false` in this case. It defines if all the app's widgets should be reloaded when the `load` method is completed.

Next, add the `LocalizationsDelegate` to the app class as arguments and define the locales that our app supports:

```Dart
return MaterialApp(
    localizationsDelegates: [
    AppLocalizationsDelegate(),
    GlobalMaterialLocalizations.delegate,
    GlobalWidgetsLocalizations.delegate
    ],
    supportedLocales: [Locale('en'), Locale('nl'), Locale('fr')],
    ...
);
```

The order of the `supportedLocales` is important as this is also the order in which a fallback will occur when the device is set to an unsupported locale.

## Localizing strings

Now that we have the boilerplate code out of the way, it's time to localize a few strings.

Resources should be added at the end of the `AppLocalizations` class as properties with a getter.

```Dart
  String get appTitle => Intl.message('My localized app', name: 'appTitle');
  String get welcomeText => Intl.message('Hello world!', name: 'welcomeText');
```

The package also has support for formatted text with arguments, plurals, genders, date and number formats etc. It's very well documented on the [Intl package readme](https://pub.dev/packages/intl).

## Using localized resources inside a widget

The app title can be translated inside `MaterialApp` or `CupertinoApp` like so:

```Dart
class MyLocalizedApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      localizationsDelegates: [
        AppLocalizationsDelegate(),
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate
      ],
      supportedLocales: [Locale("en"), Locale("nl"), Locale("fr")],
      onGenerateTitle: (BuildContext context) =>
          AppLocalizations.of(context).appTitle,
      ...
    );
  }
}
```

This is also how you can retrieve a translated resource from within a widget:

```Dart
@override
  Widget build(BuildContext context) {
    final AppLocalizations loc = AppLocalizations.of(context);

    return Scaffold(
        appBar: AppBar(
          title: Text(loc.welcomeText),
        ),
        ...
        );
  }
}
```

## Generating ARB-files

ARB stands for [Application Resource Bundle](https://github.com/google/app-resource-bundle) and is a file format made by Google that contains localized resources for software.

The format is based on JSON and uses ICU-like syntax. ICU, [International Components for Unicode](http://site.icu-project.org/), is a widely-used internationalization library.

The `Intl` package can automatically generate these files which can then be used by a translation management tool.

To generate ARB files, run the following command:

```sh
mkdir l10n
flutter pub pub run intl_translation:extract_to_arb --output-dir=l10n lib/localization.dart
```

`lib/localization.dart` is the path where my localized strings live (the `AppLocalizations` class) and `l10n` is a folder where I want the ARB files to be generated. Make sure to create this folder using the `mkdir` commando first so that the folder exists. The command will fail otherwise.

This generates a file called `intl_messages.arb` inside the `l10n` folder. Take a look at the file and you'll notice that it contains resources you mentioned in `AppLocalizations`.

## Importing in Loco

If you haven't done so already, create an account and a new project on [Loco](https://localise.biz/). In the example above I've used English as the base language for my app, so make sure that the base language of the Loco project matches with the one used above.

![Loco](/img/post/2019/06/loco-setup.png)

By clicking on the wrench icon on right side of the page, you can open the _Developer Tools_ section where you can create a _Full access API key_.

With this key, we can now upload our ARB file to Loco like so:

```sh
curl -f -s --data-binary '@l10n/intl_messages.arb' 'https://localise.biz/api/import/arb?async=true&index=id&locale=en&key=YOURAPIKEY'
```

Replace `YOURAPIKEY` with your API key and set the locale to your default locale if it's not `en`.

You can use this command and the previous one in a build script to automatically import any new translations during continuous integration. My build script is included at the end of this tutorial. There are a few [other settings](https://localise.biz/api/docs/import/import) you can include to further streamline this process.

Now you can use Loco to translate your app to all the locales you want. The advantage is that Loco is very accessible, also to non-developers. If you're doing a project for someone else, you can easily give them access to your Loco project and let them add the correct translations.

## Exporting from Loco and generating Dart files

Our solution starts to take shape, but we're not there yet. We still need to get our translated resources back into our app and our code doesn't compile yet.

When you're finished with l10n, you can export the translated resources from Loco using the following commands:

```sh
curl -s -o 'translated.zip' 'https://localise.biz/api/export/archive/arb.zip?key=YOURAPIKEY'
unzip -qq 'translated.zip' -d 'l10n'
```

This downloads the translated resources as a zip file containg ARB files and then unzips it to a folder called `l10n`. Both the import and the export functions are also available on Loco's website.

Now we can run a final command to generate the Dart code that is necessary to use our new translations.

```sh
flutter pub pub run intl_translation:generate_from_arb --output-dir=lib/l10n --no-use-deferred-loading lib/localization.dart l10n/*/l10n/intl_messages_*.arb
```

This matches the official documentation again and generates a `messages_CODE.dart` file for each locale that you support and a `messages_all.dart` that links them all together.

We can now resolve the remaining compile error in `AppLocalizations` by adding the right imports and our app is localized.

## Integrating l10n in your workflow

As far as I know, Loco is the only translation platform, besides Google Translator Toolkit, that supports working with ARB files. The steps outlined in this tutorial require some muscle memory on how to add translations, but it becomes easier as soon as you integrate this into a continuous integration workflow.

This is the script I run before every build:

```sh
#!/bin/sh -x

flutter packages get
mkdir l10n-input
flutter pub pub run intl_translation:extract_to_arb --output-dir=l10n-input lib/localization.dart
curl -f -s --data-binary '@l10n-input/intl_messages.arb' 'https://localise.biz/api/import/arb?async=true&index=id&locale=en&key=APIKEY'
curl -s -o 'translated.zip' 'https://localise.biz/api/export/archive/arb.zip?key=APIKEY'
unzip -qq 'translated.zip' -d 'l10n-translated'
flutter pub pub run intl_translation:generate_from_arb --output-dir=lib/l10n --no-use-deferred-loading lib/localization.dart l10n-translated/*/l10n/intl_messages_*.arb
rm translated.zip
rm -rf l10n-translated
rm -rf l10n-input
```

It uploads any new strings that were added during my last commits and downloads the latest version of the translations so that they are included in my build.

You can also run this locally to update your local copy of the translations during development.

## Questions?

Let me know if you have any questions, I'd be glad to help you out!