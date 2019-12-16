---
title: Anaconda Environments 
teaching: 15
exercises: 15
questions:
- "How do I install new python libraries?"
- "How are Anaconda Environments useful to my research?"
objectives:
- "Create an Anaconda Environment"
- "Add packages to that environment"
- "Share environments"
- "Change environments"
keypoints:
- "Anaconda Environments"
---

Anaconda allows you to create environments that use different 
versions of Python and/or packages installed in them. This allows you
to create a different environment for every project, switch between 
environments and share environments. Anaconda environments are independent 
of python used by the operating system so updates or changes to the system 
python or packages will not impact your environment.  Anaconda environments
retain package version information which aids in research repeatability.
This is a brief overview of how to use Anaconda environments, for a more 
complete tutorial please visit Conda's 
[Managing Environments](https://conda.io/docs/user-guide/tasks/manage-environments.html)
user guide.

## Creating an Environment

For this module we will be working in the terminal.  Create a new environment 
with `conda create`.  By default it will set the environment up
with the latest version of Python and only the necessary packages.

~~~
$ conda create --name testEnv
~~~
{: .bash}
You may be prompted to install new packages.  Hit Y [Enter] to proceed

*Note: at this time you can also specify which version of python you would like to use ex:*

~~~
$ conda create --name testEnv2 python=2.7
~~~
{: .bash}

Now activate your environment with:

~~~
$ conda activate testEnv
(testEnv) $
~~~
{: .bash}

Notice the command prompt has changed slightly to include the name of the active environment in parentheses

## Installing Packages

Now install a new package to your environment:

~~~
(testEnv) $ conda install scipy
Fetching package metadata ...........
Solving package specifications: .

Package plan for installation in environment /home/user/.conda/envs/testEnv:

The following NEW packages will be INSTALLED:

    libgfortran-ng: 7.2.0-h9f7466a_2    
    scipy:          1.0.0-py36hbf646e7_0

Proceed ([y]/n)? y

scipy-1.0.0-py 100% |###################| Time: 0:00:04   4.45 MB/s

~~~
{: .bash}

Let's view the packages currently installed in our environment:

~~~
(testEnv) $ conda list
# packages in environment at /home/user/.conda/envs/testEnv:
#
asn1crypto                0.23.0           py36h4639342_0  
ca-certificates           2017.08.26           h1d4fec5_0  
certifi                   2017.11.5        py36hf29ccca_0  
cffi                      1.11.2           py36h2825082_0  
chardet                   3.0.4            py36h0f667ec_1  
cryptography              2.1.4            py36hd09be54_0  
idna                      2.6              py36h82fb2a8_1  
intel-openmp              2018.0.0             hc7b2577_8  
libedit                   3.1                  heed3624_0  
libffi                    3.2.1                hd88cf55_4  
libgcc-ng                 7.2.0                h7cc24e2_2  
libgfortran-ng            7.2.0                h9f7466a_2  
libstdcxx-ng              7.2.0                h7a57d05_2  
mkl                       2018.0.1             h19d6760_4  
ncurses                   6.0                  h9df7e31_2  
numpy                     1.13.3           py36ha12f23b_0  
openssl                   1.0.2n               hb7f436b_0  
pandas                    0.22.0           py36hf484d3e_0  
pandas-datareader         0.5.0                    py36_0  
pip                       9.0.1            py36h6c6f9ce_4  
pycparser                 2.18             py36hf9f622e_1  
pyopenssl                 17.5.0           py36h20ba746_0  
pysocks                   1.6.7            py36hd97a5b1_1  
python                    3.6.4                hc3d631a_0  
python-dateutil           2.6.1            py36h88d3b88_1  
pytz                      2017.3           py36h63b9c63_0  
readline                  7.0                  ha6073c6_4  
requests                  2.18.4           py36he2e5f8d_1  
requests-file             1.4.1                    py36_0  
requests-ftp              0.3.1                    py36_0  
scipy                     1.0.0            py36hbf646e7_0  
setuptools                36.5.0           py36he42e2e1_0  
six                       1.11.0           py36h372c433_1  
sqlite                    3.20.1               hb898158_2  
tk                        8.6.7                hc745277_3  
urllib3                   1.22             py36hbe7ace6_0  
wheel                     0.30.0           py36hfd4bba0_1  
xz                        5.2.3                h55aa19d_2  
zlib                      1.2.11               ha838bed_2  

~~~
{: .bash}

Notice scipy is installed and for future reference the python version is 3.6.4

## Sharing Environments

Now let's try sharing our environment.  Environments are shared using yaml files
which, among other things, include the name of the environment and the environments
dependencies.  Export your environment with: 

~~~
(testEnv) $ conda env export > testEnv.yml
~~~
{: .bash}

> ## Exporting Environments in PowerShell
> 
> In windows powershell I found using `>` to save my output to file saved the data in an odd format.  Instead I had to use `conda env export | Out-File -FilePath .\environment.yml -Encoding ASCII` to force an encoding that would work
>
{: .callout}

You could then share this yaml file with others so they could load the same python 
environment you have been working in.  To simplify sharing I have provided a yaml
file to share with you: [python102.yml](../files/python102-linux.yml) (linux) or [python102.yml](../files/python102-windows.yml) (windows) Right click and select 
"save link as" to download and save the file to your current working directory.

Exit your current environment with:

~~~
$ conda deactivate
~~~
{: .bash}

Notice the command prompt has changed back and no longer shows the environment name.

To load and activate the environment I have shared with you:

~~~
$ conda env create -f python102.yml 
Using Anaconda API: https://api.anaconda.org
Fetching package metadata ...........
Solving package specifications: .
#
# To activate this environment, use:
# > conda activate python102
#
# To deactivate an active environment, use:
# > conda deactivate
#

$ conda activate python102
(python102) $
~~~
{: .bash}

## View Environments

If you have created a new environment from a file and are unsure of the name you 
can look at a list of all of your environments with:

~~~
(python102) $ conda info --envs
# conda environments:
#
python102             *  /home/user/.conda/envs/python102
testEnv                  /home/user/.conda/envs/testEnv
testEnv2                 /home/user/.conda/envs/testEnv2
root                     /software/anaconda/3
~~~
{: .bash}

*Note: the asterisks indicates the active environment.*



























