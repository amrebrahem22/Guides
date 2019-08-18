#### After Insatalling Sublime
*go to `tools` in the navbar and choose `build` then choose `python` and you will see in the terminal some wired text but that's to run some code*
### install Package Control
*go to `tools` and `command palette` then type `install package control`*</br>
*and after it installed go to `tools` and `command palette` then type `install` and you will find in the search `package-control: install package` then select it to install packages*</br>
*then type `predawn` to install it*</br>
*then open the package control install package and install `material theme`*
# Sublime Settings
### Preferences sublime settings
*then i will copy this code in sublime*
``` json
{
    "bold_folder_labels": true,
    "caret_extra_width": 1,
    "caret_style": "phase",
    "close_windows_when_empty": false,
    "color_scheme": "Packages/Predawn/predawn.tmTheme",
    "copy_with_empty_selection": false,
    "drag_text": false,
    "draw_minimap_border": true,
    "draw_white_space": "none",
    "enable_tab_scrolling": false,
    "ensure_newline_at_eof_on_save": true,
    "file_exclude_patterns":
    [
        "*.pyc",
        "*.pyo",
        "*.exe",
        "*.dll",
        "*.obj",
        "*.o",
        "*.a",
        "*.lib",
        "*.so",
        "*.dylib",
        "*.ncb",
        "*.sdf",
        "*.suo",
        "*.pdb",
        "*.idb",
        ".DS_Store",
        "*.class",
        "*.psd",
        "*.sublime-workspace"
    ],
    "font_face": "Source Code Pro",
    "font_options":
    [
        "no_round"
    ],
    "font_size": 20,
    "highlight_line": true,
    "highlight_modified_tabs": true,
    "ignored_packages":
    [
        "ActionScript",
        "AppleScript",
        "ASP",
        "D",
        "Diff",
        "Erlang",
        "Graphviz",
        "Groovy",
        "HTML-CSS-JS Prettify",
        "Lisp",
        "Lua",
        "Objective-C",
        "OCaml",
        "Rails",
        "Ruby",
        "Vintage"
    ],
    "installed_packages":[
        "Anaconda",
        "BracketHighlighter",
        "Material Theme",
        "Predawn",
        "SideBarEnhancements"
    ],
    "line_padding_bottom": 1,
    "line_padding_top": 1,
    "match_brackets_content": false,
    "match_selection": false,
    "match_tags": false,
    "material_theme_accent_graphite": true,
    "material_theme_compact_sidebar": true,
    "mini_diff": false,
    "open_files_in_new_window": false,
    "overlay_scroll_bars": "enabled",
    "preview_on_click": false,
    "scroll_past_end": true,
    "scroll_speed": 5.0,
    "show_definitions": false,
    "show_encoding": true,
    "show_errors_inline": false,
    "show_full_path": false,
    "sidebar_default": true,
    "swallow_startup_errors": true,
    "theme": "Material-Theme-Darker.sublime-theme",
    "translate_tabs_to_spaces": true,
    "trim_trailing_white_space_on_save": true,
    "use_simple_full_screen": true,
    "word_wrap": false
}
```
*then go to `Preferences > settings` in the tool bar, then i removed everything and paste my code and save then restart it* </br>
*then i want to install another package then go to `command palette` and type `install` then select `package-control: install package` and search in `blackedHighlighter` and this package is useful because it will show you the opening and closing brackets* </br>
*now i will install `sidebarEnhancements` and this package will give you a lot of options to use when right click on the file in sidebar*</br>
*now i will install `anaconda` then go to `Preferences > package settings > anaconda > settings-user` and paste this code*
``` json
{
    "auto_formatting": true,
    "autoformat_ignore":
    [
    ],
    "pep8_ignore":
    [
        "E501"
    ],
    "anaconda_linter_underlines": false,
    "anaconda_linter_mark_style": "none",
    "display_signatures": false,
    "disable_anaconda_completion": true,
    "python_interpreter": "/usr/local/bin/python3"
}
```
*and save then restart* </br>
*now it's time to create our build tool now go to `tools > build system > new build system` and paste this code and save it `python3.5.sublime-build` that's for python 3*
``` json
{
    "cmd": ["/usr/local/bin/python3", "-u", "$file"],
    "file_regex": "^[ ]*File \"(...*?)\", line ([0-9]*)",
    "quiet": true
}
```
*and for paython2 you will name it `python2.6.sublime-build`*
``` json
{
    "cmd": ["/usr/bin/python2.7", "-u", "$file"],
    "file_regex": "^[ ]*File \"(...*?)\", line ([0-9]*)",
    "quiet": true
}
```
*now when you run the file it will print the python code as in python shell inside sublime*
