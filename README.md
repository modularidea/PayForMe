# PayForMe (category fork)

Unofficial fork of [PayForMe](https://github.com/InteractionEngineer/PayForMe), an iOS client for
Cospend on Nextcloud & iHateMoney. For the app itself, releases, bug reports and contributing,
please go to the official repository:

**https://github.com/InteractionEngineer/PayForMe**

## What this fork adds

* Bill categories and payment modes on iHateMoney servers that support them (servers that report
  `categoryid` in the project info; stock iHateMoney is unaffected)
* Cospend's built-in categories in the category picker for such servers
* Optional, off by default: suggest a category from the bill title (Projects tab → Categories)

The changes are proposed upstream in
[#105](https://github.com/InteractionEngineer/PayForMe/pull/105).

## Build

Open `PayForMe.xcodeproj` in Xcode, or run the tests from the command line:

```bash
xcodebuild -project PayForMe.xcodeproj -scheme PayForMe \
  -destination 'platform=iOS Simulator,name=iPhone 17' -only-testing:PayForMeTests test
```

Using SwiftUI and Combine.
