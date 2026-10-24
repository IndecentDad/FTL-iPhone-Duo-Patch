FTL iPhone Duo patcher
======================

Turns a decrypted FTL iOS .ipa into one that:
  * runs fullscreen on iPhone Duo
  * saves to the Files app (On My iPhone > FTL) so saves can be copied in/out
  * installs on iPhone and iPad

HOW TO USE - WINDOWS
  Drag your FTL .ipa onto patch_ftl.bat.
  A new file named <original name>_duo.ipa appears next to it.
  Install that with Sideloadly as usual.

  Command line:  powershell -ExecutionPolicy Bypass -File patch_ftl.ps1 -Ipa FTL.ipa [-Out out.ipa]

HOW TO USE - macOS
  Open Terminal and run:
      bash /path/to/patcher/patch_ftl.command /path/to/FTL.ipa
  (Tip: type "bash ", drag patch_ftl.command into the window, type a space,
   drag the .ipa in, press Return.)

  To make patch_ftl.command double-clickable, run once:
      chmod +x /path/to/patcher/patch_ftl.command
  Then double-click it and drag the .ipa into the window when asked. If macOS
  blocks it as from an unidentified developer, right-click > Open, or allow it in
  System Settings > Privacy & Security.

  Uses only tools built into macOS (bash, unzip, zip, plutil, perl).

BOTH
  Keep the patcher files together in one folder:
    patch_ftl.bat, patch_ftl.ps1              (Windows)
    patch_ftl.command, patch_ftl.pl           (macOS)
    FTLSceneShim.dylib                        (both)
  Your original .ipa is never modified.
  Running the patcher on an already-patched ipa is safe; it skips what's done.
  Both versions produce identical output.

WHAT IT CHANGES
  1. Game binary
     - keeps only the 64-bit (arm64) code; the 32-bit slice is unused and encrypted
     - stamps it as built with the iOS 27.1 SDK. iPhone Duo letterboxes apps
       built with Xcode 26 or earlier, so this is what unlocks fullscreen
     - adds a load command for FTLSceneShim.dylib
     - changes one instruction so saves go to Documents instead of
       Library/Application Support
  2. Frameworks/FTLSceneShim.dylib is added. Apps built with the iOS 27 SDK must
     use the UIScene lifecycle; FTL predates it. The shim provides the scene
     delegate, starts FTL once the scene exists, puts FTL's window on that scene
     and forwards pause/resume. It writes Documents/sceneshim.log on each launch
     (visible in the Files app) for troubleshooting.
  3. Info.plist: SDK name/version, device family iPhone+iPad, arm64 capability,
     file sharing on, scene manifest pointing at FTLSceneDelegate.
  4. SC_Info (App Store DRM leftovers) is removed.

REQUIREMENTS
  * A decrypted ipa. The patcher stops if the 64-bit code is still encrypted.
  * Made for FTL 1.6.21. Other builds may work if the save-folder instruction
    pattern is found exactly once; otherwise the patcher stops without writing
    anything.

REBUILDING THE SHIM
  Source and build script: ..\shim\  (FTLSceneShim.c, build.sh)
  Needs only LLVM for Windows (clang + ld64.lld); no Apple SDK.
  Copy the rebuilt FTLSceneShim.dylib into this folder.
