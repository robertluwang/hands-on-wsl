# multi python on wsl

You can install multi version of Python on WSL, but normally latest one will override previous one when you type python/ipython/pip/easy_install cli tool.

Of course you still can run python2/pip2 or python3/pip3 to find correct python, but ipython and easy_install not that luck.

Another case is you installed multi python3, for example python3 from apt and anaconda, they are located different place.

This post is trying to find the way to run multi versions of python tool set properly.  

## python 

In my case, I installed python2 and python3 using apt on WSL, also installed anaconda3, below are locations:
```
(base) oldhorse $ which -a python
/home/oldhorse/anaconda3/bin/python  // python 3.7
/usr/bin/python  // python 2.7 
(base) oldhorse $ which -a python3
/home/oldhorse/anaconda3/bin/python3  // python 3.7
/usr/bin/python3  // python 3.6 
```
we can double verify the library path as below:

anaconda3 python 3.7 
```
(base) oldhorse $ /home/oldhorse/anaconda3/bin/python3 -c "import sys;print(sys.path)"
['', '/home/oldhorse/anaconda3/lib/python37.zip', '/home/oldhorse/anaconda3/lib/python3.7', '/home/oldhorse/anaconda3/lib/python3.7/lib-dynload', '/home/oldhorse/anaconda3/lib/python3.7/site-packages']
```
wsl python 3.6
```
(base) oldhorse $ /usr/bin/python3 -c "import sys;print(sys.path)"
['', '/usr/lib/python36.zip', '/usr/lib/python3.6', '/usr/lib/python3.6/lib-dynload', '/home/oldhorse/.local/lib/python3.6/site-packages', '/usr/local/lib/python3.6/dist-packages', '/usr/lib/python3/dist-packages']
```
wsl python 2.7
```
(base) oldhorse $ /usr/bin/python -c "import sys;print(sys.path)"
['', '/usr/lib/python2.7', '/usr/lib/python2.7/plat-x86_64-linux-gnu', '/usr/lib/python2.7/lib-tk', '/usr/lib/python2.7/lib-old', '/usr/lib/python2.7/lib-dynload', '/home/oldhorse/.local/lib/python2.7/site-packages', '/usr/local/lib/python2.7/dist-packages', '/usr/lib/python2.7/dist-packages', '/usr/lib/pymodules/python2.7']
```
## ipython
ipython is interactive python shell with lots of magic command, it is fact the module IPython, the ipython cli is wrapper script, depending on correct python binary.

anaconda3 ipython3
```
(base) oldhorse $ /home/oldhorse/anaconda3/bin/ipython3 -c "import sys;print(sys.path)"
['/home/oldhorse/anaconda3/bin', '/home/oldhorse/anaconda3/lib/python37.zip', '/home/oldhorse/anaconda3/lib/python3.7', '/home/oldhorse/anaconda3/lib/python3.7/lib-dynload', '', '/home/oldhorse/anaconda3/lib/python3.7/site-packages', '/home/oldhorse/anaconda3/lib/python3.7/site-packages/IPython/extensions', '/home/oldhorse/.ipython']
```
The wsl ipython2/ipython3 installed locally, make copy to /usr/bin.
```
(base) oldhorse $ sudo cp /home/oldhorse/.local/bin/ipython3 /usr/bin/ipython3
(base) oldhorse $ sudo cp /home/oldhorse/.local/bin/ipython2 /usr/bin/ipython2
```
can verify library path for each ipython, 
```
(base) oldhorse $ /usr/bin/ipython3 -c "import sys;print(sys.path)"
['/usr/bin', '/usr/lib/python36.zip', '/usr/lib/python3.6', '/usr/lib/python3.6/lib-dynload', '', '/home/oldhorse/.local/lib/python3.6/site-packages', '/usr/local/lib/python3.6/dist-packages', '/usr/lib/python3/dist-packages', '/home/oldhorse/.local/lib/python3.6/site-packages/IPython/extensions', '/home/oldhorse/.ipython']
(base) oldhorse $ /usr/bin/ipython2 -c "import sys;print(sys.path)"
['', '/usr/bin', '/usr/lib/python2.7', '/usr/lib/python2.7/plat-x86_64-linux-gnu', '/usr/lib/python2.7/lib-tk', '/usr/lib/python2.7/lib-old', '/usr/lib/python2.7/lib-dynload', '/home/oldhorse/.local/lib/python2.7/site-packages', '/usr/local/lib/python2.7/dist-packages', '/usr/lib/python2.7/dist-packages', '/usr/lib/pymodules/python2.7', '/home/oldhorse/.local/lib/python2.7/site-packages/IPython/extensions', '/home/oldhorse/.ipython']
```
## pip 
pip is python package management tool, it is fact the module pip for Wheel format, the pip cli is wrapper script, depending on correct python binary.

anaconda3 pip3
```
(base) oldhorse $ /home/oldhorse/anaconda3/bin/pip --version
pip 19.2.3 from /home/oldhorse/anaconda3/lib/python3.7/site-packages/pip (python 3.7)
```
the wsl standard pip2/pip3 installed on locally, so I copy them to system bin folder, 
```
(base) oldhorse $ sudo cp /home/oldhorse/.local/bin/pip2 /usr/bin/pip2
(base) oldhorse $ sudo cp /home/oldhorse/.local/bin/pip3 /usr/bin/pip3
(base) oldhorse $ /usr/bin/pip3 --version
pip 19.3.1 from /home/oldhorse/.local/lib/python3.6/site-packages/pip (python 3.6)
(base) oldhorse $ /usr/bin/pip2 --version
pip 19.3.1 from /home/oldhorse/.local/lib/python2.7/site-packages/pip (python 2.7)
```
or directly run pip as python module,
```
(base) oldhorse $ /home/oldhorse/anaconda3/bin/python3 -m pip --version
pip 19.3.1 from /home/oldhorse/anaconda3/lib/python3.7/site-packages/pip (python 3.7)
(base) oldhorse $ /usr/bin/python3 -m pip --version
pip 19.3.1 from /home/oldhorse/.local/lib/python3.6/site-packages/pip (python 3.6)
(base) oldhorse $ /usr/bin/python2 -m pip --version
pip 19.3.1 from /home/oldhorse/.local/lib/python2.7/site-packages/pip (python 2.7)
```

## easy_install 

easy_install is handy installation tool, it is from module setuptools for Eggs format, check if module setuptools installed for each pip and easy_install wrapper script existing. If setuptools there but easy_install not, just manually create one, below easy_install3 somehow not there so just copy from easy_install, modify the correct python path in #! Shebang line.
```
(base) oldhorse $ /home/oldhorse/anaconda3/bin/easy_install --version
setuptools 41.4.0 from /home/oldhorse/anaconda3/lib/python3.7/site-packages (Python 3.7)

(base) oldhorse $ /usr/local/bin/easy_install3 --version
setuptools 41.0.1 from /home/oldhorse/.local/lib/python3.6/site-packages (Python 3.6)

(base) oldhorse $ /usr/local/bin/easy_install --version
setuptools 41.4.0 from /home/oldhorse/.local/lib/python2.7/site-packages (Python 2.7)
```

## Jupyter Notebook
Jupyter notebook very handy web based env for python is for live code, equations, visualizations.

The jupyter-kernel is key concept Jupyter notebook, a notebook kernel is an operating system process (in userland) that communicates through several ZeroMQ connections. It receives code snippets to execute, runs these code snippets, and returns the result and output of the execution.

install kernel using correct python,

wsl python2
```
(base) oldhorse $ /usr/bin/python2 -m ipykernel install --user
Installed kernelspec python2 in /home/oldhorse/.local/share/jupyter/kernels/python2
(base) oldhorse $ cat /home/oldhorse/.local/share/jupyter/kernels/python2/kernel.json
{
 "argv": [
  "/usr/bin/python2",
  "-m",
  "ipykernel_launcher",
  "-f",
  "{connection_file}"
 ],
 "display_name": "Python 2",
 "language": "python"
}
```
wsl python3
```
(base) oldhorse $ /usr/bin/python3 -m ipykernel install --user
Installed kernelspec python3 in /home/oldhorse/.local/share/jupyter/kernels/python3
(base) oldhorse $ cat /home/oldhorse/.local/share/jupyter/kernels/python3/kernel.json
{
 "argv": [
  "/usr/bin/python3",
  "-m",
  "ipykernel_launcher",
  "-f",
  "{connection_file}"
 ],
 "display_name": "Python 3",
 "language": "python"
}
```
when install conda python3 ipykernel , will overwrite the previous python3, so here is remedy, changed to different kernel name and display name, 
```
(base) oldhorse $ /home/oldhorse/anaconda3/bin/ipython3 -m ipykernel install --user --name conda-python3 --display-name "conda-python3"
Installed kernelspec conda-python3 in /home/oldhorse/.local/share/jupyter/kernels/conda-python3
(base) oldhorse $ cat /home/oldhorse/.local/share/jupyter/kernels/conda-python3/kernel.json
{
 "argv": [
  "/home/oldhorse/anaconda3/bin/python3",
  "-m",
  "ipykernel_launcher",
  "-f",
  "{connection_file}"
 ],
 "display_name": "conda-python3",
 "language": "python"
}
```
so far there are 3 ipykernel setting to 3 python versions, 
```
(base) oldhorse $ /usr/bin/python3 jupyter-kernelspec list|grep -i python
  conda-python3    /home/oldhorse/.local/share/jupyter/kernels/conda-python3
  python2          /home/oldhorse/.local/share/jupyter/kernels/python2
  python3          /home/oldhorse/.local/share/jupyter/kernels/python3
```
Here is sample launch,
```
(base) oldhorse $ python3 -m jupyter notebook
[I 17:27:15.813 NotebookApp] JupyterLab extension loaded from /home/oldhorse/anaconda3/lib/python3.7/site-packages/jupyterlab
[I 17:27:15.814 NotebookApp] JupyterLab application directory is /home/oldhorse/anaconda3/share/jupyter/lab
[I 17:27:17.012 NotebookApp] Serving notebooks from local directory: /home/oldhorse
[I 17:27:17.013 NotebookApp] The Jupyter Notebook is running at:
[I 17:27:17.013 NotebookApp] http://localhost:8888/?token=e663bd3ec02de1f9502e5a1aabfcbfdf84b2316ede3e475a
[I 17:27:17.014 NotebookApp]  or http://127.0.0.1:8888/?token=e663bd3ec02de1f9502e5a1aabfcbfdf84b2316ede3e475a
[I 17:27:17.014 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[C 17:27:22.273 NotebookApp]
```
