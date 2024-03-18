
# Misc: made-sense
solvers: [L3d](https://github.com/imL3d)  
writeup-writer: [L3d](https://github.com/imL3d)  
___
**Author:** doubledelete  
**Description:**
> i couldn't log in to my server so my friend kindly spun up a server to let me test makefiles. at least, they thought i couldn't log in :P  

**files (copy):** [app.py](files/app.py)  

In this challenge we receive a site (and it's code), that allows us to write and execute a GNU Make recipe, with some restrictions.  
We need to bypass those restrictions and get the flag.  
Sssentially, a Make jail.  

## Solution

*This Challenge is the first challenge out of a series of 4 challenges.*  


### Preview  
Before starting this challenge, lets examine the source code given.  
The web application we got receives the Make files `target`, and it's `content`. It checks each part against some regex pattern, and if it passes it builds and execute the following Makefile:
```make
SHELL := /bin/bash
.PHONY: TARGET

TARGET: flag.txt
    CONTENT
```  
Lets break this Makefile down:

`.PHONY: TARGET` - this part make sure each time the `make` command is being run, the recipe is executed. You can read more about in the [official documentation](https://www.gnu.org/software/make/manual/html_node/Phony-Targets.html) (its really good).  
`TARGET: flag.txt` - this defines a [recipe](https://www.gnu.org/software/make/manual/html_node/Recipes.html) - the target name is the input given by the user, and the dependecy is `flag.txt` - the flag!  
`CONTENT` - this is the recipe's content given by the user, that is then being run by in the shell. What shell? the shell defined in the first line of this Makefile, in this case `bash`.

### Solution
In that case this is very easy, this challenge seems very easy! why not just print the `flag.txt`.  
But, upon trying to send `cat flag.txt` as the content we don't get the flag. 
this is becasue the content is being checked to not contain the string `"flag"`:
```python
if re.search(r'flag', code):
    return 'no'
```  
We can easily bypass this with a simple string concatination: `cat "fla""g.txt"`.  
Bingo! we get the flag: 
`wctf{m4k1ng_vuln3r4b1l1t135}`