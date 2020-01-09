# Error Message Formatter

[TOC]

## Introduction

To make a light-weight, simple error message formatter that visualize the error message in a nice and tidy manner.  The major requirement for this is **regular expression** and parsing technique.  

The scale for this project is light-weighted, single-file and does not require much external libraries.  

## Library

1. Phrase I
   - Display: Bootstrap
   - Display: Font Awesome
   - Function: mixed
   - Image: Filesaver
   - Image: dom-to-image
   - Target: 
     - Prototype
2. Phrase II
   - Display: Bootstrap
   - Display: Font Awesome
   - Function: jQuery
   - Image: Filesaver
   - Image: dom-to-image
   - Target:
     - Tidy up the structure
     - Convert as much DOM-related code to jQuery
3. Phrase III
   - Display: Bootstrap
   - Display: Font Awesome
   - Function: Vue
   - Image: Filesaver
   - Image: dom-to-image
   - Target: 
     - use Vue ability to minimize code, such as placeholder for `v-if`,  stack trace table with `v-for`

## Design

### Functions

- Format error messages
  - Error Message: format in large font
  - Stack trace: format in table
- Trim file name
- Display file
  - Hide lines with filtered files in stack trace
  - Only show lines with target files in stack trace
  - Italic lines with filter files in stack trace
  - Bold lines with target files in stack trace
- Download Result
  - Download formatted error message as an image
- Theme / Styling
  - Light Layout
  - Dark Layout

### Input Variables

- Mode
  - Display All
  - Filter File
  - Display Only Target File
- Directories (for Trim File Name)
- Filter Files (for Display File)
- Target Files (for Display File)
- Theme

### Runtime Variables and Functions

#### Formatter Module

The main working module of the file.  

| Function Name            | Input                 | Output          | Description                                                |
| ------------------------ | --------------------- | --------------- | ---------------------------------------------------------- |
| Extractor                | none                  | JSON            | extract setting from DOM tree                              |
| Line Breaker             | string, JSON          | array of string | Break string into lines                                    |
| Error Message Identifier | array of string, JSON | string          | extract the main error message                             |
| Stack Trace Formatter    | array of string, JSON | array of arrays | Further break string into columns                          |
| Stack Trace Visualizer   | array of arrays, JSON | array of arrays | Filter out unwanted lines & mark display options like bold |
| Visualizer               | array of arrays, JSON | none            | Modify the DOM tree                                        |
| Container                | none                  | none            | Orchestra all the functions                                |

The pseudo-code for the formatter module is used as below: 

```javascript
function container() {
    var settings = extractor();
    var raw_error_str = $("textarea[name='input_raw_error_str']").val();
    if (raw_error_str.trim() == '') {
      alert('Error: Empty Error Message');
      return;
    }
    var lines = line_breaker(raw_error_str,settings);
    if (lines.length<=0) {
      alert('Error: Invalid Language Options');
      return
    }
    var main_error_message = error_message_identifier(lines, settings);    
    var stack_trace_lines = stack_trace_formatter(lines, settings);
    var stack_trace_tables = stack_trace_visualizer(stack_trace_lines, settings);    
    visualizer(stack_trace_tables, main_error_message, settings);
  }
```

#### Input Module

Module to assist display and input.  

| Function Name | Input        | Output | Description                                                  |
| ------------- | ------------ | ------ | ------------------------------------------------------------ |
| addRow        | `td` element | none   | add 1 row below the nearest row, used by row.                |
| deleteRow     | `td` element | none   | remove the nearest row, used by row.                         |
| appendRow     | string       | none   | add 1 row below the table, used by the header of table.      |
| clearTable    | string       | none   | remove all row(s) below the table, used by the header of table. |

The pseudo-code for the input table is as shown below: 

```html
<div class='row'>
    <div class="col-8">
        <h3>Title</h3>
    </div>
    <div class="col-4">
        <button type="button" name="button" class="btn btn-default" onclick="input_appendRow('table_id')"><i class="fa fa-plus" aria-hidden="true"></i></button>
        <button type="button" name="button" class="btn btn-default" onclick="input_clearTable('table_id')"><i class="fa fa-eraser"></i></button>
    </div>
</div>
<table class="table table-hover" id="table_id">                    
    <tbody>
        <tr>
            <td>
                <input type="text" style="min-width: 100%">
            </td>
            <td>
                <button type="button" name="button" class="btn btn-default" onclick="input_addRow(this)"><i class="fa fa-plus" aria-hidden="true"></i></button>
                <button type="button" name="button" class="btn btn-default" onclick="input_deleteRow(this)"><i class="fa fa-trash" aria-hidden="true"></i></button>
            </td>
        </tr>
    </tbody>
</table>
```

#### Display Module

Module for styling.  

| Function Name    | Input | Output | Description                             |
| ---------------- | ----- | ------ | --------------------------------------- |
| display_theme    | none  | none   | change the theme of formatter           |
| display_no_row() | none  | none   | display placeholder when no stack trace |

#### Utility Module

Module for basic components

| Function Name | Input                  | Output         | Description                                              |
| ------------- | ---------------------- | -------------- | -------------------------------------------------------- |
| util_exist    | string, list of string | Boolean        | the string includes any of the strings in list           |
| util_trim     | string, list of string | string         | trim the string if it matches any of the strings in list |
| util_extract  | string, list of string | list of string | return the extracted strings from given pattern          |

#### Others

| Function Name | Input | Output | Description                          |
| ------------- | ----- | ------ | ------------------------------------ |
| screen_cap    | none  | none   | Change the background and text color |

## Example

### Python

Error Message:

```
    Traceback (most recent call last):
      File "<doctest...>", line 10, in <module>
        lumberjack()
      File "<doctest...>", line 4, in lumberjack
        bright_side_of_death()
    IndexError: tuple index out of range
```

Expected Output:

**IndexError: tuple index out of range**

| #    | File         | Line | Function   | Code                   |
| ---- | ------------ | ---- | ---------- | ---------------------- |
| 0    | <doctest...> | 10   | <module>   | lumberjack()           |
| 1    | <doctest...> | 4    | lumberjack | bright_side_of_death() |

### PHP

Error Message: 

```
Fatal error: Uncaught exception 'PDOException' with message SQLSTATE[42000] [1049] Unknown database ‘pdol" in c:\www\hosts\localhost\common.inc.php:46 Stack trace: #0 c:\www\hosts\localhost\common.inc.php(46): PDO->__construct('mysqt:host=loca...', ‘root’, ‘root’, Array) #1 c:\www\hosts\localhost\books.php(10): include(‘c:\www\hosts\lo...') #2 {main} thrown in c:\www\hosts\localhost\common.inc.php on line 46
```

Expected Output: 

**Fatal error: Uncaught exception 'PDOException' with message SQLSTATE[42000] [1049] Unknown database 'pdol"**

| #    | File                                  | Line | Function | Code                                                         |
| ---- | ------------------------------------- | ---- | -------- | ------------------------------------------------------------ |
| 0    | c:\www\hosts\localhost\common.inc.php | 46   |          | PDO->__construct('mysqt:host=loca...', ‘root’, ‘root’, Array) |
| 1    | c:\www\hosts\localhost\books.php      | 10   |          | include(‘c:\www\hosts\lo...')                                |
| 2    | c:\www\hosts\localhost\common.inc.php | 46   | {main}   |                                                              |

If we input dir= 'c:\www\hosts\localhost\\': 

**Fatal error: Uncaught exception 'PDOException' with message SQLSTATE[42000] [1049] Unknown database 'pdol"**

| #    | File           | Line | Function | Code                                                         |
| ---- | -------------- | ---- | -------- | ------------------------------------------------------------ |
| 0    | common.inc.php | 46   |          | PDO->__construct('mysqt:host=loca...', ‘root’, ‘root’, Array) |
| 1    | books.php      | 10   |          | include(‘c:\www\hosts\lo...')                                |
| 2    | common.inc.php | 46   | {main}   |                                                              |

Further input filter_files= 'books.php' as it is not the main working file and choose filter files as format mode:  

**Fatal error: Uncaught exception 'PDOException' with message SQLSTATE[42000] [1049] Unknown database 'pdol"**

| #    | File           | Line | Function | Code                                                         |
| ---- | -------------- | ---- | -------- | ------------------------------------------------------------ |
| 0    | common.inc.php | 46   |          | PDO->__construct('mysqt:host=loca...', ‘root’, ‘root’, Array) |
| 1    | common.inc.php | 46   | {main}   |                                                              |

## Security

Since the browser is purely driven by JavaScript, no sensitive message are sent to any server, thus ensuring no leaking out development progress or stealing code.  

## Problems

### Exceptional Case

There are 3 known exceptional case that the formatter cannot work on: 

1. Cannot Handle String issue

```
  File "<stdin>", line 1
    while True print('Hello world')
                   ^
SyntaxError: invalid syntax
```

The above error message has a line with only `^` symbol, which breaks the formatting

2. Cannot accept irregular formatting

```
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ZeroDivisionError: division by zero
```

The above error message does not have any code in the 3rd line.  

They are pretty the same: violating an expected formatting.  

#### Image Download Size

Theoretically, there is no limit on the length of error message.  However, the image download size may exceed the limit of Filesaver.js, or the `textarea` maximum input length.  

### Browser Compatibility

The formatter should work properly, unless the external libraries cannot work

| Browser | Version |
| ------- | ------- |
| Chrome  | 49      |
| Firefox | 45      |

The major reason for many browser not working such as IE and Safari and Opera is due to the limit of dom-to-image.js.  It is welcome to use html2canvas in the function `screencap()`, but the image quality and font styling is much worse.  

### Device

The formatter is not displaying well in mobile device, but still working well except the image downloading image.  

## Credits

Vincent Chu Chi Hang, Dec 2019 - Jan 2020, Copyright Reserved