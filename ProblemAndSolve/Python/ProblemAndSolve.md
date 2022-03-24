## [Python] Problem & Solve

> ### [SOLVED] SyntaxError: Non-ASCII character '\xeb' in file
> **Problem** :: * SyntaxError: Non-ASCII character '\xeb' in file  
> **Solution** :: open the file, add `# -*- coding: utf-8 -*-` on the first line, save then. Try to execute again.

> ### [SOLVED] Python Syntax error (replace single quote to double quote)
> ![img.png](../../Assets/images/img.png)
> **Problem** :: The quote for replace is missing. `polygon_obj = eval(polygon_str.replace("'", ""))`  
> **Solution** :: Wrap the double quote with single quote to replace from single quote to double quote.   
> So change the statement to : `polygon_obj = eval(polygon_str.replace("'", '"'))`.
