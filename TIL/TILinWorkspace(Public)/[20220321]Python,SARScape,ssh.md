## Python, SARscape runner, ssh
`2022.03.21. - `

---

## Task
* [x] fix the encoding to UTF-8
* [x] fix `% Attempt to call undefined procedure: 'PSINSAR_PROCESS'.`
* [x] Analyze the script, success to execute
* [x] Success to execute zip_shp.py
* [ ] Fix HttpStatus 451
* [ ] Success to upload result file and generate to geoserver layer

---

## Details

[ Run Python script ]
> python SCRIPT-NAME.py

[ Syntax ]

**== commands ==**

> ### Execute python file in cmd
> `python SCRIPT-NAME.py`
>
> ### Edit file with vim editor
> `vim SCRIPT-NAME.py`
>
> ### Save changes and quit vim editor
> press ESC, then type to `:wq!`
>
> ### Don't save changes and quit in vim
> press ESC, then type to `:q!`
>
> ### Install requirements
> pip3 install -r requirements.txt
>
> ### Execute bash shell script in cmd
> sh ./SCRIPT-FILENAME.sh
> 
> ### Create file with cmd
> cat FILENAME.EXT
>
> ### Set line number on vim editor
> set nu
>
> ### Undo the type
> `u`

**== Python ==**
>
> ### Print something
> print("something")
>
> ### Exit program with cmd
> exit()

---

## Issues & Problem solving
### SyntaxError: Non-ASCII character '\xeb' in file
> **Problem** :: * SyntaxError: Non-ASCII character '\xeb' in file  
> **Solution** :: open the file, add `# -*- coding: utf-8 -*-` on the first line, save then. Try to execute again.

### Python Syntax error (replace single quote to double quote)
![img.png](../../Assets/images/img.png)
> **Problem** :: The quote for replace is missing. `polygon_obj = eval(polygon_str.replace("'", ""))`  
> **Solution** :: Wrap the double quote with single quote to replace from single quote to double quote.   
> So change the statement to : `polygon_obj = eval(polygon_str.replace("'", '"'))`.


### % Attempt to call undefined procedure: 'PSINSAR_PROCESS'.
![img_1.png](../../Assets/images/img_1.png)
> **Problem** :: % Attempt to call undefined procedure: 'PSINSAR_PROCESS'.  
> **Solution** :: When execute idl .pro file with shell script, the extension '.pro' is already included so if try to execute with the command that like `idl -e FILENAME.pro` would syntax error occurred.
> So omit to the extension. Change from `shell_command = ["idl", "-e" "psinsar_process_test.sav.", "-args" ...]` to `shell_command = ["idl", "-e", "psinsar_senitnel1_test", "-args" ... ]`

### Comparison enum values in Python3
> **Problem**: Comparison enum values has been failed.    
> **Solution**: Explanation follows:
> ```python
> from enum import Enum, IntEnum
> ...
> class ENUM(IntEnum):
>     ENUM-NAME = SOME-VALUE
> ...
> def send_to_server_result(self, test_file_path, INT-ARG):
>     if(INT-ARG != ENUM.ENUM-NAME):
>         DO-SOMETHING
>         return
> ```

---

## Remark

---

## Reference
[Omit to Extension when execute .pro file with shell script](https://www.l3harrisgeospatial.com/Support/Forums/aft/7058)  
[Python : I want to replace single quotes with double quotes in a list](https://stackoverflow.com/questions/42183479/i-want-to-replace-single-quotes-with-double-quotes-in-a-list)  
[Python : The Enum types in Python](https://brownbears.tistory.com/531)
---
