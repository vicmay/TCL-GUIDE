# Tk Guide - Tcl's GUI Toolkit

## Table of Contents
- [Introduction to Tk](#introduction-to-tk)
- [Basic Widgets](#basic-widgets)
- [Window Management](#window-management)
- [Geometry Management](#geometry-management)
- [Event Handling](#event-handling)
- [Dialogs and Menus](#dialogs-and-menus)
- [Canvas Widget](#canvas-widget)
- [Text Widget](#text-widget)
- [Themed Widgets (ttk)](#themed-widgets-ttk)
- [Custom Widgets](#custom-widgets)
- [Best Practices](#best-practices)
- [Common Pitfalls](#common-pitfalls)
- [Resources](#resources)

## Introduction to Tk

Tk is Tcl's standard GUI toolkit, providing a powerful and easy-to-use interface for creating graphical applications. It's cross-platform, running on Windows, macOS, and Linux with a native look and feel.

```tcl
# Simple Tk Hello World
package require Tk

# Create main window
set w .
wm title $w "Hello Tk"

# Create a label
label $w.l -text "Hello, Tk!" -font {Arial 12 bold}
pack $w.l -padx 50 -pady 20

# Create a button
button $w.b -text "Click Me!" -command {tk_messageBox -message "Button clicked!"}
pack $w.b -pady 10
```

## Basic Widgets

### Label
```tcl
label .label -text "This is a label" -foreground blue -background white
pack .label
```

### Button
```tcl
button .btn -text "Click Me" -command {puts "Button clicked!"} -bg lightblue
pack .btn
```

### Entry
```tcl
entry .entry -width 30 -font {Arial 10}
pack .entry
```

### Checkbutton
```tcl
checkbutton .chk -text "Enable feature" -variable ::feature_enabled -onvalue 1 -offvalue 0
pack .chk
```

### Radiobutton
```tcl
set ::choice 1
radiobutton .rb1 -text "Option 1" -variable ::choice -value 1
radiobutton .rb2 -text "Option 2" -variable ::choice -value 2
pack .rb1 .rb2
```

## Window Management

### Creating and Configuring Windows
```tcl
# Create a new toplevel window
toplevel .new
wm title .new "New Window"
wm geometry .new 300x200+100+100

# Add widgets to the new window
label .new.l -text "This is a new window"
pack .new.l
```

### Modal Dialogs
```tcl
proc show_dialog {} {
    set dlg [toplevel .dialog]
    wm title $dlg "Dialog"
    wm transient $dlg .
    wm protocol $dlg WM_DELETE_WINDOW {}
    
    label $dlg.l -text "Are you sure?"
    button $dlg.yes -text "Yes" -command {set ::dialog_result 1}
    button $dlg.no -text "No" -command {set ::dialog_result 0}
    
    grid $dlg.l -columnspan 2 -pady 10
    grid $dlg.yes $dlg.no -padx 10 -pady 10
    
    # Center the dialog
    wm withdraw $dlg
    update idletasks
    set x [expr {[winfo screenwidth .]/2 - [winfo reqwidth $dlg]/2}]
    set y [expr {[winfo screenheight .]/2 - [winfo reqheight $dlg]/2}]
    wm geometry $dlg +$x+$y
    wm deiconify $dlg
    
    # Wait for user response
    vwait ::dialog_result
    destroy $dlg
    return $::dialog_result
}
```

## Geometry Management

### Pack Geometry Manager
```tcl
# Pack with options
pack [label .l1 -text "Top"] -side top -fill x -padx 5 -pady 5
pack [label .l2 -text "Left"] -side left -fill y -padx 5 -pady 5
pack [label .l3 -text "Right"] -side right -padx 5 -pady 5
pack [label .l4 -text "Bottom"] -side bottom -fill x -padx 5 -pady 5
```

### Grid Geometry Manager
```tcl
# Create a simple form using grid
label .name_label -text "Name:"
entry .name_entry -width 30
label .email_label -text "Email:"
entry .email_entry -width 30
button .submit -text "Submit"

# Place widgets in grid
grid .name_label .name_entry -sticky w -padx 5 -pady 5
grid .email_label .email_entry -sticky w -padx 5 -pady 5
grid .submit -columnspan 2 -pady 10
```

### Place Geometry Manager
```tcl
# Absolute positioning with place
label .placed -text "Positioned at 50,50" -bg lightblue
place .placed -x 50 -y 50 -width 150 -height 30
```

## Event Handling

### Binding to Events
```tcl
# Button click event
button .btn -text "Click Me"
pack .btn
bind .btn <Button-1> {puts "Button 1 clicked at %x,%y"}

# Key press event
bind all <Key> {puts "Key pressed: %K"}

# Mouse motion
bind . <Motion> {wm title . "Mouse at %x,%y"}
```

### Event Sequences
```tcl
# Double click
bind .btn <Double-1> {puts "Double clicked!"}

# Key combinations
bind . <Control-s> {save_file ; break}
bind . <Control-q> {exit}
```

## Dialogs and Menus

### Standard Dialogs
```tcl
# Message box
tk_messageBox -message "Operation completed" -type ok

# File dialog
set filename [tk_getOpenFile -title "Select a file" -filetypes {
    {"Text files" {.txt .text}}
    {"All files" *}
}]

# Color dialog
set color [tk_chooseColor -title "Choose a color"]
```

### Creating Menus
```tcl
# Create menu bar
menu .menubar
. configure -menu .menubar

# File menu
menu .menubar.file -tearoff 0
.menubar add cascade -label "File" -menu .menubar.file
.menubar.file add command -label "New" -command new_file
.menubar.file add command -label "Open..." -command open_file
.menubar.file add separator
.menubar.file add command -label "Exit" -command exit

# Edit menu
menu .menubar.edit -tearoff 0
.menubar add cascade -label "Edit" -menu .menubar.edit
.menubar.edit add command -label "Cut" -command {event generate . <<Cut>>}
.menubar.edit add command -label "Copy" -command {event generate . <<Copy>>}
.menubar.edit add command -label "Paste" -command {event generate . <<Paste>>}
```

## Canvas Widget

The canvas widget provides powerful drawing capabilities.

```tcl
# Create a canvas
canvas .c -width 400 -height 300 -background white
pack .c -fill both -expand true

# Draw shapes
.c create rectangle 10 10 100 100 -fill blue -outline black
.c create oval 150 50 250 150 -fill red -width 2
.c create line 10 200 390 200 -width 3 -arrow both
.c create text 200 250 -text "Canvas Example" -font {Arial 14 bold}

# Add interactivity
set item [.c create rectangle 10 150 50 200 -fill green]
.c bind $item <Button-1> {.c itemconfigure $item -fill [expr {[rand() < 0.5 ? "red" : "green"]}]
```

## Text Widget

The text widget is a powerful widget for displaying and editing text.

```tcl
# Create a text widget with scrollbar
frame .text_frame
set t [text .text_frame.text -wrap word -undo 1 -width 60 -height 20]
set sb [scrollbar .text_frame.sb -command "$t yview"]
$t configure -yscrollcommand "$sb set"
pack .text_frame.sb -side right -fill y
pack .text_frame.text -side left -fill both -expand true
pack .text_frame -fill both -expand true -padx 5 -pady 5

# Add some text with tags
$t insert end "Welcome to the Text Widget!\n\n"
$t insert end "This is a demonstration of the text widget's capabilities.\n"

# Configure tags
$t tag configure heading -font {Arial 14 bold} -foreground blue
$t tag add heading 1.0 1.23

# Add a button to insert text
button .insert -text "Insert Text" -command {
    $t insert end "More text inserted at [clock format [clock seconds]]\n"
    $t see end
}
pack .insert -pady 5
```

## Themed Widgets (ttk)

Tk 8.5+ includes the themed widget set (ttk) which provides native-looking widgets.

```tcl
package require Tk 8.5
package require ttk

# Use ttk widgets for a modern look
ttk::style theme use clam  ;# Other themes: alt, default, classic

ttk::frame .f -padding "10 10 10 10"
pack .f -fill both -expand true

ttk::label .f.l -text "Username:"
ttk::entry .f.e -width 20
ttk::button .f.b -text "Login" -command {puts "Login as [$f.e get]"}

grid .f.l -row 0 -column 0 -sticky w -pady 5
grid .f.e -row 0 -column 1 -sticky we -pady 5
grid .f.b -row 1 -column 0 -columnspan 2 -pady 10

grid columnconfigure .f 1 -weight 1
```

## Custom Widgets

Create your own composite widgets using namespaces and procedures.

```tcl
namespace eval mywidgets {
    namespace export labeled_entry
    
    proc labeled_entry {w args} {
        frame $w -class LabeledEntry
        
        # Parse options
        array set opts $args
        set label_text [lindex $opts(-text) 0]
        set entry_var [lindex $opts(-textvariable) 0]
        
        # Create subwidgets
        label $w.label -text $label_text
        entry $w.entry -textvariable $entry_var
        
        # Pack them
        pack $w.label -side left -padx 5
        pack $w.entry -side left -fill x -expand true
        
        # Return the entry widget path for further configuration
        return $w.entry
    }
}

# Use the custom widget
mywidgets::labeled_entry .name -text "Name:" -textvariable ::name
pack .name -fill x -padx 10 -pady 5
```

## Best Practices

1. **Use ttk widgets** for a native look and feel across platforms
2. **Namespace your code** to avoid naming conflicts
3. **Use grid or pack** consistently within a container
4. **Separate UI and logic** by using procedures for event handlers
5. **Use variables** for widget options that might change
6. **Enable undo** in text widgets
7. **Handle window close** events properly
8. **Use standard dialogs** for common operations
9. **Test on multiple platforms** if cross-platform compatibility is needed
10. **Use the event loop** effectively with `after` for background tasks

## Common Pitfalls

1. **Forgetting to pack/grid/place widgets** - Widgets won't appear without a geometry manager
2. **Global variables in callbacks** - Use namespace variables or `upvar`
3. **Blocking the event loop** - Move long-running tasks to a separate thread
4. **Memory leaks** - Use `destroy` for toplevel windows when done
5. **Race conditions** - Be careful with `vwait` and the event loop
6. **Platform differences** - Test on all target platforms
7. **Font issues** - Use system fonts or package them with your application
8. **HiDPI displays** - Test on high-DPI screens

## Resources

- [Tk Documentation](https://www.tcl.tk/man/tcl8.6/TkCmd/contents.htm)
- [Tk Tutorial](https://tkdocs.com/tutorial/)
- [Tkinter 8.5 Reference](https://docs.python.org/3/library/tk.html) (Python's Tk interface, but the concepts are the same)
- [TkDocs](https://tkdocs.com/) - Modern Tk documentation
- [Tcler's Wiki](https://wiki.tcl-lang.org/) - Community resources and examples
- [Tk Themed Widgets](https://www.tcl.tk/man/tcl8.6/TkCmd/ttk_widget.htm)
- [Tk Canvas](https://www.tcl.tk/man/tcl8.6/TkCmd/canvas.htm)
- [Tk Text Widget](https://www.tcl.tk/man/tcl8.6/TkCmd/text.htm)
