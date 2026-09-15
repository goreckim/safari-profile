# safari-profile

Open a URL in a **specific Safari profile** from the command line.

```sh
safari-profile Work https://github.com/example/example-repo/pull/123
```

Safari has no way to do this on its own: `open -a Safari <url>` lands in whichever window
happens to be frontmost, so a URL that needs the GitHub login living in your `Work`
profile may well open in `Personal`.

## Behaviour

| situation | what happens |
| --- | --- |
| no window belongs to `<profile>` | a new window is opened in that profile, with `<url>` |
| a tab with `<url>` is already open in one of that profile's windows | that window is raised and that tab becomes current |
| otherwise | a new tab with `<url>` is opened in the profile's frontmost window |

In every case the window is de-minimized, raised, made frontmost, and the relevant tab
becomes the current one.

## Usage

```
safari-profile <profile> <url>              open <url> in <profile>
safari-profile --dry-run <profile> <url>    report the action, change nothing
safari-profile --list                       list the available Safari profiles
safari-profile --help
```

Exit codes: `0` ok, `1` usage error, `2` unknown profile, `3` Safari/AppleScript failure.

### URL matching

URLs are compared after normalization:

* the `#fragment` is dropped,
* the scheme and host are case-folded,
* a default port is dropped (`:80` for `http`, `:443` for `https`),
* one trailing `/` is removed.

Path and query are otherwise significant (including their case). So
`https://example.com/a`, `https://EXAMPLE.com/a/` and `https://example.com/a#top` all match
the same tab, while `https://example.com/a?b=1` does not.

## Requirements and how it works

Only tools that ship with macOS: `bash`, `osascript`, `awk`. Tested on macOS 26.6 with
Safari 26.6.

Safari's AppleScript dictionary has no notion of profiles, so the script uses two facts:

1. **Identifying a window's profile.** When more than one profile exists, Safari prefixes
   each window title with the profile name and an em dash (U+2014), so a window of profile
   `Work` showing example.com is titled `Work [U+2014] Example Domain`. That prefix is
   readable through Safari's own dictionary, so finding windows and tabs needs no special
   permission.
2. **Creating a window in a profile.** This is only reachable through the menu bar:
   **File > New Window > `New <Profile> Window`**, driven with System Events.

### Accessibility permission

The menu-bar part (`--list`, and opening a window for a profile that has none) requires
**Accessibility permission for the application that runs the script**: your terminal, or
whatever launches it. Grant it in *System Settings > Privacy & Security > Accessibility*.
Reusing an existing window or tab needs no permission.

### Limitations

* Requires **more than one Safari profile**: with a single profile Safari adds no title
  prefix and offers no *New Window* submenu.
* Relies on the **English** menu titles `File` and `New Window`.
* A window whose page title is empty appears as just `Work`; that case is handled, but a
  page whose own title happens to start with the profile name plus an em dash would be
  misattributed.
* Private-browsing windows are not profile windows and are never matched.
