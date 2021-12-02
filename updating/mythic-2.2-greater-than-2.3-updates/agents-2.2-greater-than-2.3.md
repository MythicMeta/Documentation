# Agents 2.2 -> 2.3

## Build Parameters

Build parameters are moving from a dictionary format to an array. There's now also the build parameter type of `BuildParameterType.Boolean`.

```
 build_parameters = [
        BuildParameter(
            name="mode",
            parameter_type=BuildParameterType.ChooseOne,
            description="Choose the build mode option. Select default for executables, "
                        "c-shared for a .dylib or .so file, "
                        "or c-archive for a .Zip containing C source code with an archive and header file",
            choices=["default", "c-archive", "c-shared"],
            default_value="default",
        ),
        BuildParameter(
            name="proxy_bypass",
            parameter_type=BuildParameterType.Boolean,
            default_value=False,
            description="Ignore HTTP proxy environment settings configured on the target host?",
        ),
    ]
```

There shouldn't have to be any other changes to this.

## Supported Operating Systems

In your payload definition file, you can specify the supported operating systems from a specific set of operating systems via `supported_os = [SupportedOS.Linux, SupportedOS.MacOS]`. To provide even more options going forward, if you want to specify an OS that doesn't have a pre-defined value (like `SupportedOS.Linux`), then you can supply your own via `SupportedOS("MyOS")`. At that point, `MyOS` would appear in the web interface as a build option.

## Browser Scripts

With the decommissioning of the old interface, we will also be decommissioning the old style of browser scripts. With that, there are no more `support_browser_scripts` in your payload definition file. That is strictly a relic of the old way of doing things.

In your individual commands though, there are still browser scripts. There needs to be an additional attribute `for_new_ui` added in to specify that a script is meant for the new UI vs the old UI.

```
browser_script = [
  BrowserScript(script_name="ls", author="@its_a_feature_"),
  BrowserScript(script_name="ls_new", author="@its_a_feature_", for_new_ui=True)
]
```

