# Xcode could not resolve type from dSYM if build folder is different

## Problem
The problem is that the debugger does not see the types from the context.
Debug panel does not display types, only names.
At the same time, breakpoint work, dSYM loaded (did it by hand - add-dsym ) and symbols exist.

```
(lldb) po self
error: <EXPR>:3:1: error: use of unresolved identifier 'self'
self
^~~~
(lldb) fr variable
self = <could not resolve type>
(lldb) image lookup -r -n DevPodViewController*.
10 matches found in /Users/user/DD/DebugProblemApp-guvizewcykoobqfyyzhzzcggtkeq/Build/Products/Debug-iphonesimulator/DebugProblemApp.app/Frameworks/DevPod.framework/DevPod:
        Address: DevPod[0x0000000000001820] (DevPod.__TEXT.__text + 0)
        Summary: DevPod`DevPod.DevPodViewController.viewDidLoad() -> () at DevPodViewController.swift:5        Address: DevPod[0x00000000000019a0] (DevPod.__TEXT.__text + 384)
        Summary: DevPod`type metadata accessor for DevPod.DevPodViewController at <compiler-generated>        Address: DevPod[0x00000000000019f0] (DevPod.__TEXT.__text + 464)
        Summary: DevPod`@objc DevPod.DevPodViewController.viewDidLoad() -> () at <compiler-generated>        Address: DevPod[0x0000000000001a30] (DevPod.__TEXT.__text + 528)
        Summary: DevPod`DevPod.DevPodViewController.__allocating_init(nibName: Swift.Optional<Swift.String>, bundle: Swift.Optional<__C.NSBundle>) -> DevPod.DevPodViewController at DevPodViewController.swift:3        Address: DevPod[0x0000000000001a80] (DevPod.__TEXT.__text + 608)
        Summary: DevPod`DevPod.DevPodViewController.init(nibName: Swift.Optional<Swift.String>, bundle: Swift.Optional<__C.NSBundle>) -> DevPod.DevPodViewController at DevPodViewController.swift:3        Address: DevPod[0x0000000000001c10] (DevPod.__TEXT.__text + 1008)
        Summary: DevPod`@objc DevPod.DevPodViewController.init(nibName: Swift.Optional<Swift.String>, bundle: Swift.Optional<__C.NSBundle>) -> DevPod.DevPodViewController at <compiler-generated>        Address: DevPod[0x0000000000001cc0] (DevPod.__TEXT.__text + 1184)
        Summary: DevPod`DevPod.DevPodViewController.__allocating_init(coder: __C.NSCoder) -> Swift.Optional<DevPod.DevPodViewController> at DevPodViewController.swift:3        Address: DevPod[0x0000000000001d00] (DevPod.__TEXT.__text + 1248)
        Summary: DevPod`DevPod.DevPodViewController.init(coder: __C.NSCoder) -> Swift.Optional<DevPod.DevPodViewController> at DevPodViewController.swift:3        Address: DevPod[0x0000000000001dd0] (DevPod.__TEXT.__text + 1456)
        Summary: DevPod`@objc DevPod.DevPodViewController.init(coder: __C.NSCoder) -> Swift.Optional<DevPod.DevPodViewController> at <compiler-generated>        Address: DevPod[0x0000000000001e10] (DevPod.__TEXT.__text + 1520)
        Summary: DevPod`DevPod.DevPodViewController.__deallocating_deinit at DevPodViewController.swift:3
```

And I can even create this type
```
(lldb) po DevPodViewController()
<DevPod.DevPodViewController: 0x7fbdb5d1b690>
(lldb) po DevPodViewController.self
DevPod.DevPodViewController
```

## How came across a problem
I am trying to implement solutions to reuse prebuilt dynamic frameworks.
I came across a problem with debug and learned how to consistently reproduce it using a simpler example.

## More about the demo project:
I have a project with iOS application target. 
The target has a dependency on the dynamic framework in which the controller is located. 
This is a framework I want to debug. There is a configuration file - Config.xcconfig. 
This file says that you need to link with the framework and where to look for this framework.
```
FRAMEWORK_SEARCH_PATHS = $(inherited) "${CONFIGURATION_BUILD_DIR}/DevPod"
OTHER_LDFLAGS = $(inherited) -framework "DevPod"
OTHER_CFLAGS = $(inherited) -iquote "${CONFIGURATION_BUILD_DIR}/DevPod/DevPod.framework/Headers"
CONFIGURATION_BUILD_DIR = ${BUILD_DIR}/$(CONFIGURATION)$(EFFECTIVE_PLATFORM_NAME)
```

There is also a shell build step inside project file that copies the framework and dSYM from the path I specified to the folder where it should be, according to the configuration file. (For linking)
And one more build step that move framework from the build folder inside the application. ( inside .app for load )
I also have another target in the same project. It is needed to build the framework.
The main target does not depend on the framework in the project settings only through the configuration file. ( It is important )

## Steps to reproduce
1. Open framework project - DebugProblemApp.xcodeproj
2. Choose a scheme to build the framework - DevPod
3. Built it for iOS simulator
2. Then copy the results framework and dSYM from Dereived Data in the DevPodFramework folder.
4. Switch to main scheme - DebugProblemApp
5. Add breakpoint to DevPodViewController file - DevPod/DevPodViewController.swift line 7
6. Run
7. Check that everything works - the variable (self) is displayed // or it may not work already
8. Delete everything from Derived Data folder
9. Run again
10. Check that variable information is no longer displayed

Result:
Debug panel does not display types, only names.

Additional Information:
In this case, the path to the source files (the files which the framework originated from) was not changed (equal).
If the framework was from an Objective C files, debug would work. (Verified)
