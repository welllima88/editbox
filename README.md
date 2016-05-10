# editbox
Editbox is a plugin for the [Volatility Framework](https://github.com/volatilityfoundation/volatility). It extracts the text from Windows Edit controls, that is, textboxes as generated by Windows Common Controls.

## How does it work?
Edit controls are created by a call to [CreateWindowEx](https://msdn.microsoft.com/en-us/library/windows/desktop/ms632680%28v=vs.85%29.aspx). As such, they are technically windows. Their class name looks something like this:
`6.0.7601.17514!Edit`

The part before the exclamation mark (`6.0.7601.17514`) is the version of the [Common Controls](http://msdn.microsoft.com/en-us/library/windows/desktop/bb775493%28v=vs.85%29.aspx) library being used. Owing to Windows Side-by-Side execution (WinSxS), there can be many different versions of the Common Controls library in use at one time.

Edit controls, like Buttons and ListBoxes, are obviously special kinds of windows. As such, they have extra data after the window definition which differentiates them from a generic window. It works like this:
* The window has a property called cbWndExtra. This "[s]pecifies the number of extra bytes to allocate following the window instance."
* Typically, these bytes are a memory address to a structure.
* The structure holds the extra bytes that distinguish a specific window, for example an Edit control, from a generic window.
* The editbox plugin understands what (some of) the extra bytes mean, so, by parsing the structure, is able to get information about the Edit control.

## What information can it get?
1. **nChars**: The number of characters in the Edit control.
2. **selStart**: Selection Start. If text within the control is selected, this is the 0-based index of the first selected character.
3. **selEnd**: Selection End. If text within the control is selected, this is the 0-based index of the last selected character.
4. **isPwdControl**: Whether the EditBox is a password control, that is, characters are masked by another character.
5. **undoPos**: If there is text in the undo buffer, it was present at this offset.
6. **undoLen**: If there is text in the undo buffer, this is how many characters.
7. **address-of undoBuf**: Virtual memory address of the text in the undo buffer.
8. **undoBuf**: Shows up to the first 50 characters of the undo buffer. (If there are more than 50, the last 3 are replaced with '...'.)
7. And of course, the actual text of the Edit control.

## How do I use it?
### Let Volatility know you're using an additional plugin.
Use the `--plugins` switch to specify the folder containing any additional plugins you wish Volatility to load:
```
$ python vol.py --plugins=/folder/to/editbox -f memory.dmp --profile=Win7SP1x64 editbox
```
### Switches
#### --dump-dir/-D
The text of an Edit control can be long. For example, the contents of a Notepad window. Using this switch, EditBox will dump the text to a file in the specified folder. The file will be named as per the MD5 of the text from the Edit control.
#### --pid/-p
EditBox will only work on processes with the specified Process ID.

## Sample Output
```
Wnd context          : 1\WinSta0\Default
Process ID           : 1748
imageFileName        : explorer.exe
atom_class           : 6.0.7601.17514!Edit
value-of WndExtra    : 0x5ad76d0 [0x250c6d0]
nChars               : 7
selStart             : 6
selEnd               : 7
isPwdControl         : Yes
undoPos              : 0
undoLen              : 6
address-of undoBuf   : 0xb1508
undoBuf              : cheeky
monkey!
```
## More information about the experimental options.
The experimental option will try and extract useful information from the following controls:
* ListBox

### ListBox
#### What information can it get?
1. **caretPos**: The 0-based offset of the item in the list which has focus.
2. **rowsVisible**: The number of rows the ListBox shows before the user must scroll.
3. **firstVisibleRow**: The 0-based offset of the first row which is visible.
4. **itemCount**: The number of items contained in the list.
5. **stringsStart**: The memory offset at which the strings making up the items in the list can be found.
6. **stringsLength**: The number of bytes which make up the strings. Each character requires 2 bytes, and each string is null-terminated.
7. **strings**: A comma-separated list of the strings from the ListBox.

#### Sample Output
```
*******************************************************
*** Experimental **************************************
*******************************************************
Wnd context          : 1\WinSta0\Default
pointer-to tagWND    : 0xfffff900c06615f0 [0x16a445f0]
pid                  : 2576
process              : ComCtl.exe
wow64                : No
atom_class           : 6.0.7601.17514!Listbox
address-of cbwndExtra: 0xfffff900c0668418 [0xaa92418]
value-of cbwndExtra  : 8 (0x8)
address-of WndExtra  : 0xfffff900c0668458 [0xaa92458]
value-of WndExtra    : 0x2fd410 [0xd963410]
firstVisibleRow      : 0 (0x0)
caretPos             : 0 (0x0)
rowsVisible          : 3 (0x3)
itemCount            : 3 (0x3)
stringsStart         : 0x2fd510 [0xd963510]
stringsLength        : 56 (0x38)
strings              : lbxa-zero, lbxa-one, lbxa-two

```
