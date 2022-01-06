# Jav Config overview

## Introduction
The goal of this document is to give an overview of the usage of the `jav_config.ws` file and it's format. I hope that once you read this you will have a better understanding of how the game client is initialized and configured, as well as a better understanding of the AppletViewer parsing code.

> Note: Eventually this will be combined with other documentation and hosted all together. Once that happens this repo will be updated with a link to that.

### Disclaimer
All information used to construct this document came from reverse engineering the Jagex Applet Viewer delivered within the download section of runescape.com as it appeared on July 18th 2010. This __Does not__ necessarily align with the more recent parsing logic implemented in newer versions of RuneScape or OldSchool Runescape.

### Quick overview of applet viewer invocation
Starting in 2010 Jagex supplied users the option to download a executable which would launch Runescape from their desktop (instead of as an `<Applet />` loading in the browser). In order to support this the following components are required:

- Launcher
- AppletViewer
- Loader
- Client

#### Launcher
The launcher is a very simple `C++` program whose sole purpose is to collect JVM options and initialize the JVM using the bundled JRE. It is passed a single argument `gameName` which is passed to the `AppletViewer` as well as being used to determine the directory of the `.prm` file containing JVM options.

```bash
# Default location of the launcher on Windows
%LOCALAPPDATA%\jagexlauncher\bin\JagexLauncher.exe

# Default location of the <gameName>.prm file
%LOCALAPPDATA%\jagexlauncher\<gameName>\<gameName>.prm
```

Within the `.prm` file there are a number of rudementary JVM options, but also a Java definitions which specify the location of the `jav_config` configuration file:

- `-Dcom.jagex.config`
- `-Dcom.jagex.configFile`

The applet viewer requires one of these to be set (if both are set `-Dcom.jagex.config` takes precedence). If neither are set the applet viewer will exit immediately with an error.

#### Applet Viewer
Once passed the configuration file location, it is the Applet Viewer's job to fetch the contents of the file and parse it.

If passed a configuraton file path using `-Dcom.jagex.configFile` it is directly loaded from the filesystem. No special handling is nessasary.

---

If passed a URL the client first tries to resolve any variables which might be present in the URL. Variables can be present in the URL using the following format:

`https://example.com/lang/$(name:defaultValue)`

The applet viewer only supports a single variable `Language` being declared in the URL:

`http://www.runescape.com/k=3/l=$(Language:0)/jav_config.ws`

The AppletViewer has a initial setup routine which attempts to utilize `Locale.getDefault();` to resolve the users locale. And map that to a integer language ID.
- `0 == English`
- `1 == German`
- `2 == French`
- `3 == Brazilian Portuguese`

By using the above table you can see that if the applet viewer is unable to resolve the users locale it will fallback to using english.

## Format of `jav_config`

### Example
The format of the `jav_config` is quite similar to the [toml configuration file format](https://toml.io/).

It allows for:
- Comments
- Top level configuration values
- Scoped configuration values (using server blocks)
- Locale strings for internationalizing the string displayed to the user
- Parameters for the downstream applications to query
- String keys
- String values

An example configuration file can be seen below:

```
// This is the top level configuration block. If the selected server doesn't specify a config item explicitly
// lookup will fallback to searching this block.
title=RuneScape
serverlist=http://example.com/serverlist.ws

# Locale strings
# Format: msg=<key>=<value>
msg=quit=Quit

# Parameters
# Format: param=<key>=<value>
param=modewhat=0

# Server blocks allow you to specify specific server configurations which override the top level configuration.
[server1]
# `servername` is used to populate the dropdown within the applet viewer
servername=ServerOne
title=My Server One

[server2]
title=My server Two
servername=ServerTwo
```

### Comments
`jav_config` supports two different styles of single line comments. Multiline comments are not supported.

`# Lines starting with "#" are ignored`

`// Lines starting with "//" are also ignored`

### Base configuration options
Configuration options which are not prepended by either `msg=` or `param=` are, unless explicitly passed, used exclusively to configure the applet viewer.

| Configuration Option | Description |
| --- | --- |
| `title` | Specifies the title of the frame |
| `viewerversion` | Specifies the required applet viewer version. |
| `servername` | Name of the server configuration. |
| `serverlist` | A URL specifying which server configs are enabled. |
| `cachesubdir` | The subdirectory where cache and other files will be placed. |
| `codebase` | The root URL where assets are fetched. |
| `browsercontrol_win_x86_jar` | x86 `browsercontrol.dll` jar filename. |
| `browsercontrol_win_amd64_jar` | amd64 `browsercontrol.dll` jar filename. |
| `loader_jar` | Filename for the loader jar. |
| `advert_height` | Height of the advertisement container. |
| `window_preferredwidth` | Preferred width of the frame. |
| `window_preferredheight` | Preferred height of the frame. |
| `applet_minwidth` | Minimum width of the frame. |
| `applet_minheight` | Minimum height of the frame. |
| `applet_maxwidth` | Maximum width of the frame. |
| `applet_maxheight` | Maximim height of the frame. |
| `adverturl` | URL where advertisment content will be loaded. |

### Server Configuration blocks
In order to support multiple server configurations without having to update `-Dcom.jagex.config` or `-Dcom.jagex.configFile` you can specify a "server block". This is done by adding a block to the configuration file using the following format

```
[serverid]
servername=Human readable server name
# ... Everything after here will override the top level configuration
```
By adding two or more server blocks you will be presented with an additional link within the applet viewer, which allows switching between the servers:
![Switch server option](https://user-images.githubusercontent.com/2308484/148379231-32230139-a95b-4280-b019-c27b450a3799.png)

The content for this link is loaded from `msg=switchserver`

And opon clicking this link you will be presented with all the enabled server configurations. Listed by their `servername` configuration option:

![Switch server dialog](https://user-images.githubusercontent.com/2308484/148379611-22dc2abf-a774-410d-8db5-4c439fe928f2.png)

Once you switch to another server the applet viewer will reload with the new configuration.

### Serverlist file

If you happen to have a massive amount of server configurations and wish to only enable a subset of them to appear within the server list you can utilize the `serverlist` configuration option to specify a URL which contains a list of serverid's and their enabled status.

For example, given the following configuration:

```
serverlist=http://example.com/serverlist.ws
# ...Remaining configuration items

[server1]
servername=My cool server
# ...Other configuration overides

[server2]
servername=My other cool server
# ...Other configuration overides
```

And the following `serverlist.ws` file hosted on `http://example.com/serverlist.ws`

```
server1,false
server2,true
```

You will see the following dialog when attempting to switch servers

![Server list filtered switch server dialog](https://user-images.githubusercontent.com/2308484/148380487-596623e6-3ca1-4c77-8e8a-c0c7dace7c2c.png)

### Locale strings

The `jav_config` configuration file provides the ability to implement rudimentary localization with the usage of "locale strings". These are configuration items with the following format:

`msg=<key>=<value>`

And are used for all user facing strings within the applet viewer

### Parameters
> Note: I have not yet read through all the downstream applications. The information below is from reading through the applet viewer and loader only.

In order to emulate the `parameter` functionality of `<Applet />`'s the `jav_config` configuration file allows for specifying "parameters" using the following format:

`param=<key>=<value>`

These configuration items can be accessed by the downstream `<Applet />`'s by utilizing the `getParameter()` method.
