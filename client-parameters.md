# Client Parameters

## Introduction
The hopes of this document will be to cover every parameter accessed by both the loader and client.

## `modewhat` : `0`

### Applet Viewer
Within the `AppletViewer` the `modewhat` parameter is only used to determine the cache location.

The cache folder name is constructing using the following logic:

`.jagex_cache_<32 + modewhat>`

### Loader
The loader uses this parameter for the same purpose as the Applet Viewer

### Client

## `unsignedurl` : `null`

### Loader

If this parameter is set the loader will attempt to execute `SecurityManager.checkExec('echo');`

If the loader is unsigned the above execution will fail and the applet context will be instructed to show the URL pointed to by the `unsignedurl` parameter.

### Client

**TODO**

## `crashurl` : `null`

### Loader

If this parameter is set the loader will utilize this pointed to URL for showing error messages.

**When the parameter is set the error URL will be in the following format:**

`<crashurl>&s=<errorid>`

**When the parameter is NOT set the error URL will be in the following format:**

`<Applet.getCodeBase()>/error_loader_<errorid>.ws`

### Client

**TODO**

## `cachesubdirid` : `0`

### Loader

This parameter determines the subdirectory used when reading and writing the cache.

_Note: There are a number of possible cache locations, but for this document the only the subdirectory is relevant_

The following table shows the mapping:

| `cachesubdirid` | Cache subdirectory folder name |
| --- | --- |
| 0 | "runescape" |
| 1 | "stellardawn" |

**An example cache directory when `cachesubdirid` == `0`**

`C:\.jagex_cache_32\runescape\`

**An example cache directory when `cachesubdirid` == `1`**

`C:\.jagex_cache_32\stellardawn\`

### Client

**TODO**

## `colorid` : `0`

### Loader

This parameter determines the colorscheme used when painting the progress bar. The colors correspond to the different game values seen above,
and the table below shows which colors are active depending on which color id is set:

| `lang` ID | Game name | Progress bar bg hex | Progress bar font hex |
| --- | --- | --- | --- |
| 0 | "runescape" | `#8C1111` | `#FFFFFF` |
| 0 | "stellardawn" | `#FFFFFF` | `#FFFFFF` |

## `lang` : `0`

### Loader

This parameter corresponds to the language ID and is used when localizing strings.

In the case of the loader, the `lang` parameter only supports the following languages for displaying loading information:

| `lang` ID | Language name |
| --- | --- |
| 0 | English |
| 1 | German |
| 2 | French |

**_Note:_** In the 592 loader, only the above languages are supported, but support for up to `lang=4` is handled.

- In the case that `lang > 2 && lang < 5` English will be used as a fallback.
- Cases where `lang > 5` are invalid and will result in accessing outside array bounds.

## `suppress_sha` : `null`

### Loader

By the name and the usage in the loader it can be assumed that this flag turns off `SHA-1` validation of files loaded from the cache. But actually this
parameter doesn't do anything. It can be assumed that in the production build the logic for skipping `SHA-1` validation has been stripped out.

**_Note:_** The only time this parameter would be applicable would be when loading the following files from the **cache**. If they cannot be found in the cache
they are fetched from the server, and `SHA-1` validation is explicity forced.

- `jaggl` libraries
- `jagmisc` libraries
- `sw3d` libraries
