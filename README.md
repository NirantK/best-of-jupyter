# Making the Best of Jupyter
Making the best of Jupyter: Tips, Tricks, Best Practices with Sample Code for Productivity Boost
---

### Getting Started Right
- Start your Jupyter server with ```supervisor``` or ```tmux``` instead of direct ```ssh``` or ```bash```. This works out to be more stable Jupyter server which doesn't die unexpectedly. Consider writing the buffer logs to a file rather than stdout
    - This is specially useful when working inside _Docker_ using `docker attach` where you might not see a lot of logs
- Consider using a ssh client like [MobaXterm Personal Portable Edition](https://download.mobatek.net/10520180106182002/MobaXterm_Portable_v10.5.zip) with multiple tabbed ssh client options
- Refer our [How to Tunnel using SSH](https://github.com/NirantK/best-of-jupyter/blob/master/TUNNELING.md) (with illustrations) to tunnel to a remote Jupyter notebook

# Debugging 
- When you see an error, you can run [```%debug```](http://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-debug) in a new cell to activate IPython Debugger. Standard keyboard shortcuts such as ```c``` for continue, ```n``` for next, ```q``` for quit apply
- Use `from IPython.core.debugger import set_trace` to **I**Python debugger checkpoints, the same way you would for `pdb` in PyCharm


```python
from IPython.core.debugger import set_trace

def foobar(n):
    x = 1337
    y = x + n
    set_trace() #this one triggers the debugger
    return y

foobar(3)
```

Returns:

```python-traceback
> <ipython-input-9-04f82805e71f>(7)fobar()
      5     y = x + n
      6     set_trace() #this one triggers the debugger
----> 7     return y
      8 
      9 fobar(3)

ipdb> q
Exiting Debugger.
```

*Preference Note:* If I already have an exception, I prefer `%debug` because I can zero down to the exact line where code breaks compared to ``set_trace()`` where I have to traverse line by line

- When editing imported code, use [```%load_ext autoreload; %autoreload 2 ```](https://ipython.org/ipython-doc/3/config/extensions/autoreload.html). The autoreload utility reloads modules automatically before entering the execution of code typed at the IPython prompt.

This makes the following workflow possible:
```python
In [1]: %load_ext autoreload

In [2]: %autoreload 2  # set autoreload flag to 2. Why? This reloads modules every time before executing the typed Python code

In [3]: from foo import some_function

In [4]: some_function()
Out[4]: 42

In [5]: # open foo.py in an editor and change some_function to return 43

In [6]: some_function()
Out[6]: 43
```
- When using `print(out_var)` on a nested list or dictionary, consider doing `print(json.dumps(out_var, indent=2))` instead. It will _pretty print_ the output string. 

# Programming Sugar
- _Cool Feature_: Execute a shell command from inside your notebook. You can use this to check what files are in available in your working folder```!ls *.csv``` or even ```!pwd``` to check your current working directory
    - You can `cd {PATH}` where PATH is a Python variable, similarly you can do `PATH = !pwd` to use relative paths instead of absolute
    - Both `pwd` and `!pwd` work with mild preference for `!pwd` to signal other code readers that this is a shell command
- Running jupyter from an environment does NOT mean that the shell environment in `!` will have the same environment variables
    - Running `!pip install foo` (or `conda install bar`) will use the `pip` which is in the path for the `sh` shell which might be different from whatever `bash` shell environment you use
- If you want to install a package while inside Jupyter and `!pip install foo` doesn't seem to do it, try:

```python
import sys
!{sys.executable} -m pip install foo  # sys.executable points to the python that is running in your kernel 
```
- Shell commands are nice, but we *discourage* their use - makes it difficult to refactor to script later. For instance `cd ../../` in jupyter could be done using `os.setcwd()`as well

## Jupyter Kungfu

- If in a cell after writing a function you hit `shift + tab`, it will display function's docstring in a tooltip, and it has options to expand the tooltip or expand it at the bottom of the screen    
- Use `?func_name()` to view function, class docstrings etc. For example:

```python-traceback
?str.replace() 
```

Returns:
```python-traceback
Docstring:
S.replace(old, new[, count]) -> str

Return a copy of S with all occurrences of substring
old replaced by new.  If the optional argument count is
given, only the first count occurrences are replaced.
Type:      method_descriptor
```

- List all the variables/functions in a module: `module_name.*?`. For instance: `pd.*?`. Additionally, this works with prefixes: `pd.read_*?` and `pd.*_csv?` will also work
- Show the docstring+code for a function/class using : `pd.read_csv??`    
- Press ```h``` to view keyboard shortcuts

# Better Mindset
- **IMPORTANT**: Frequently rewrite each cell logic into functions. These functions can be moved to separate ```.py``` files on regular intervals. Your notebook run should be mainly function calls. 
    - This would prevent your notebook from becoming a giant pudding of several global variables
- If particular cells take too long to run, add ``%%time`` cell magic as a warning + runtime logger
- If you are on **Py3.6+**, please use f-strings! ```f"This is iteration: {iter_number}"```is much more readable than ```.format()``` syntax
- Any code that is used in more than 3 notebooks should be moved to .py files (such as utils.py) and imported such as ```from xxx_imports import *```
- Quite often, we frequently re-run same code cell. Instead, refactor that cell to a function and call that function repeatedly to prevent accidental edits
- Use ```Pathlib``` instead of ```os.path``` wherever possible for more readable code. Here is a [beginner friendly tutorial](https://medium.com/@ageitgey/python-3-quick-tip-the-easy-way-to-deal-with-file-paths-on-windows-mac-and-linux-11a072b58d5f). If you just want to review, refer the [crisp tutorial](https://jefftriplett.com/2017/pathlib-is-wonderful/) or [official docs](https://docs.python.org/3/library/pathlib.html)

# Plotting and Visualization
- Always have [```%matplotlib inline```](http://ipython.readthedocs.io/en/stable/interactive/plotting.html) to ensure that the plots are rendered inside the notebook
- Use separate plotting functions instead of repeating ``plt.plot`` code to avoid code bloating. Using ``subplots`` from Matplotlib OO API is usually neater than using more ``plt.plots``

```python
def show_img(im, figsize=None, ax=None, title=None):
    if not ax: fig,ax = plt.subplots(figsize=figsize)
    ax.imshow(im, cmap='gray')
    if title is not None: ax.set_title(title)
    ax.get_xaxis().set_visible(True)
    ax.get_yaxis().set_visible(True)
    return ax
    
def draw_rect(ax, bbox):
    x, y, w, h = bbox
    patch = ax.add_patch(patches.Rectangle((x, y), w,h, fill=False, edgecolor='red', lw=2))
```

``show_img`` is a reusable plotting function which can be easily extended to plot one off images as well properly use subplots.
In below example, I use a single figure and add new images as subplots using the neater axes.flat syntax:

```python
fig, axes = plt.subplots(1, 2, figsize=(6, 2))
ax = show_img(char_img, ax= axes.flat[0], title = 'char_img_line_cropping:\n'+str(char_img.shape))
ax = show_img(char_bg_mask, ax=axes.flat[1], title = 'Bkg_mask:\n'+str(char_bg_mask.shape))

#  If you are working on image segmentation task, you can easily add red rectangles per subplot:
draw_rect(ax, char_bounding_boxes)  # will add red bounding boxes for each character
```

### General Tips
- If your imports are failing, check your notebook kernel on the right top in gray
- Consider using ```conda``` for instead of ```pip virtualenv``` similar because that ensures package versions are consistent. `conda` is not a Python package manager. Check [conda (vs pip): Myths and Misconceptions](https://jakevdp.github.io/blog/2016/08/25/conda-myths-and-misconceptions/) from the creator of Pandas
- The cell type can be changed to markdown and plain text too
    - Some people convert code cells to markdown if you don't want to execute them but don't want to comment either
- Consider downloading a notebook as a Python file and then push to Gitlab for code review

### Please don't overdo Cell Magic
- Don't use alias and alias_magic unless extremely helpful. Aliases make your code difficult to read for other developers
- Don't leave ```%%timeit``` in your code. Why? Because it does 1,00,000 runs of the cell and then return average of best 3 runtimes. This is not always needed. Instead use ```%%time``` or add average times in inline comments

# With Inputs from 
- Abdul Qadir
- [Anuj Khare](https://www.linkedin.com/in/anuj-khare-72942498/)
- Harsh Gupta
