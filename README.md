# BxFits

Generating and analysing custom MC fits

Part 0: Set up fitter

Part 1: Set up this machinery

Part 2: Generate submissions

Part 3: Collect fit results

Part 4: Create comparison tables


## Part 0: Set up fitter (for JURECA)

Two setup scripts:

```/p/project/cjikp20/jikp2008/setup_fitter_p*.sh```

Usage:

1. Create a folder in which you want to install the fitter, e.g.

```console
mkdir 2020-03-11-fitter-v63
```

2. Copy the two scripts there

```console
cp setup_fitter_p*.sh 2020-03-11-fitter-v631/.
```

3. Run the first script and state fitter version

```console
./setup_fitter_p1.sh 6.3.1
```

(it will ask you for the git credentials as part of the process and
 then keep going)

4. After the script is done running, run the second part

```console
./setup_fitter_p2.sh
```

5. Done


## Part 1: Set up this machinery

1. In your ```bx-GooStats``` folder, get the package


```console
git clone https://github.com/sagitta42/BxFits.git
```

NOTE: ```git clone``` does not work in develgpus mode for some reason. After the fitter installation, you will be in develgpus mode, so before proceeding to Part 1, do ```exit``` to exit to the normal mode (then go back to the ```bx-GooStats/``` folder).

2. Set up the basics

```console
./BxFits/setitup.sh
```

## Part 2: Generate submissions

1. Create a configuration file e.g. ```config_file.txt``` with lines

```bash
inputs=Phase3
Emin=140,150
Rdmin=484,500
var=npmts_dt1,npmts_dt2
nbatch=10
outfolder=my_systematics
...
```
Example files included in this repo: ```total_systematics_ana_hm.txt, systematics_ana_hm_mi.sh```

2. To see all possible inputs, do

```console
python massive.py
```

3. To create combinations do

```console
python massive.py config_file.txt
```

It will create

- folder ```fitoptions/``` with generated cfg files 

- folder ```species_list``` with generated icc files

- folder ```outfolder/``` (the name you give with the ```outfolder``` variable in ```config_file.txt```) with ```fit_*.sh``` files corresponding to your settings, and ```sbatch_*.sh``` files that launch the fit files. The future log files will be saved in this folder

- file ```outfolder_submission.sh``` that launches all the jobs

4. To submit the jobs do

```console
./outfolder_submission.sh
```

Make sure to do it when you are NOT in GPU mode.

## Part 3: Collect fit results

Usage

```console
python collect_species.py foldername
```

Will create a file ```foldername_species.out``` with a table of results collected from the log files in the given folder

On top of the file (line 8), one can select the species one wants to collect

```python
COLUMNS = ['nu(Be7)', 'nu(pep)', 'Bi210', 'C11', 'Kr85', 'Po210',\
    'Ext_Bi214', 'Ext_K40', 'Ext_Tl208', 'Po210shift', 'C11shift', 'chi2/ndof',\
    'C11_2', 'Po210_2']
```

If you don't need all of them, you can select only specific ones. 

At line 23, you can define what will be your "special column". The value of this column is not read from the log file, but from the name of the log file. On line 69 is the command that does this reading. For example,

```python
special_col = 'Period'
# data_Phase3_EO_22C11_TAUP_MI_Nov.log
spec = filename.split('_')[1] # Phase3
```

means that the name of the column will be 'Period', and the way the value for that column is read is "split the filename by underscore and take the second part". See example output in the bottom in the Examples section.

Using this table it's very easy to make plots with python

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv('examples/fits2012-2019_species.out', sep = ' ')
df.plot(x = 'Period', y = 'Bi210', yerr = 'Bi210Error')
plt.savefig('examples/bi210rate.png')
```

(```plot_example.py```)





## Part 4: Create comparison tables

Use

```console
python table_comparison_v1.py foldername
```

It will create a file ```foldername_comparison.csv``` that you can open with Spreadsheet. The results from each log file in the folder will be put side by side for comparison


If the species is not present in the log file (for example, here it's Po210 shift), a NaN value will be assigned. That means, in the table file it will be an empty space. If you open this file with a Spreadsheet program, it will be an empty cell. The default separator is space. If you want, you can change it to comma in the code.



## Examples to try out 

### Collet species

To create a table


```console
$ python collect_species.py examples/fits2012-2019
  Period  nu(Be7)  nu(pep)  Bi210   C11  Kr85   Po210  Ext_Bi214  ...  Kr85Error  Po210Error  Po210_2Error Po210shiftError  nu(Be7)Error  nu(pep)Error    ExpSub    ExpTag
4   2012     49.2     3.37   23.4  1.78  11.1  717.50       1.61  ...        4.3        2.70           2.9           0.041           2.8          0.72  10931.80  10074.20
6   2013     50.4     4.93   14.6  1.51   9.3  160.10       2.67  ...        4.0        1.50           1.8           0.110           2.9          0.76  10737.80   6624.08
0   2014     49.4     2.82   14.0  1.25   7.7  105.10       3.38  ...        3.4        1.10           1.4           0.130           2.5          0.60  14021.80   8045.30
3   2015     52.4     3.33    9.5  1.59   8.3   80.30       1.86  ...        3.5        1.00           1.3           0.170           2.6          0.65  12898.70   8145.44
5   2016     45.3     3.14    8.9  1.53  17.6   55.00       3.15  ...        4.1        1.10           1.3           0.230           2.9          0.73  10837.60   6266.81
1   2017     52.7     3.92    5.7  1.26   8.4   54.00       4.69  ...        3.8        1.00           1.2           0.220           2.7          0.66  13139.10   8261.60
2   2018     49.5     3.62    9.9  1.51   7.7   49.03       5.17  ...        3.6        0.98           1.1           0.240           2.6          0.66  14190.00   9299.89
7   2019     47.0     4.02    7.8  1.02  15.7   49.20       5.40  ...        5.5        1.60           1.8           0.350           4.1          0.95   5851.96   3776.53

[8 rows x 30 columns]
--> examples/fits2012-2019_species.out
```

To make a plot

```console
python plot_example.py
```

Will create ```examples/bi210rate.png```


### Table comparison

Create a csv with comparison

```console
python table_comparison.py examples/Ext_K40free

~~~ fit-cno-mv-pdfs_TAUP2017-PeriodPhase2MI-nhits-emin140_CNO-penalty-met_hm.log

           fit-cno-mv-pdfs_TAUP2017-PeriodPhase2MI-nhits-emin140_CNO-penalty-met_hm         
                                                                               Mean    Error
Period                                                 Phase2                            NaN
nu(Be7)                                                  49.1                            1.2
nu(pep)                                                   1.4                            1.4
Bi210                                                     8.8                            8.3
C11                                                     1.796                          0.097
Kr85                                                      4.2                            2.2
Po210                                                  249.59                           0.78
Ext_Bi214                                                1.82                            0.3
Ext_K40                                                  0.94                           0.62
Ext_Tl208                                                3.41                           0.16
Po210shift                                                NaN                            NaN
C11shift                                                  NaN                            NaN
chi2/ndof                                              1.2449                            NaN
C11_2                                                   64.29                           0.59
Po210_2                                                 306.6                              1
C11avg                                                26.9442                        1.47608
Po210avg                                              272.531                        1.23105

~~~ fit-cno-mv-pdfs_new-PeriodPhase2MI-nhits-emin140_CNO-penalty-met_hm.log

...

~~~ fit-gpu-mv-pdfs_TAUP2017-PeriodPhase2MZ-nhits-emin92_CNO-pileup-penalty-met_hm.log

...

~~~ fit-cno-mv-pdfs_new-PeriodPhase3MI-nhits-emin140_CNO-penalty-met_hm.log

...

~~~ fit-cno-mv-pdfs_TAUP2017-PeriodPhase2MI-nhits-emin140_CNO-pileup-penalty-met_hm.log

...         

~~~ fit-gpu-mv-pdfs_TAUP2017-PeriodPhase2MI-nhits-emin92_CNO-pileup-penalty-met_hm.log

...

----------------
-> Ext_K40free_comparison.csv
```

Later you can open it with Spreadsheet and make a table like in the example ```examples/Ext_K40free_comparison.pdf```


### Extra info

CNAF paths

PDFs:
/storage/gpfs_data/borexino/wg/nusol/MCpdfs/v2.1.3

Fitter inputs:
/storage/gpfs_data/borexino/wg/nusol/fitter_input/v4.1.3


## How to add a fitoptions (cfg) parameter


1. Add the line at the END of ```BxFits/templates/fitoptions.cfg``` (if does not exist yet) e.g. ```multivariate_rdist_fit_min = 500```.

2. Come up with a name for the parameter that will be used in the config e.g. ```rdmin```.

#### In ```generator.py```

3. If there are only certain values it can take (e.g. only "MI" and "MZ" for ```tfc```), add it to the dictionary ```options```.

4. Otherwise, add it to the list ```user```.

5. If you would like it to have a default value (i.e. can omit it in the config), add it to the dictonary ```defaults``` (e.g. ```'rdmin' : 500```).

6. If you would like to give multiple values for this parameter, so that multiple cfgs are generated, add it to the list ```par_loop```.

#### In ```creator.py``` in the class ```Submission()```

7. In the constructor ```__init__()```, create a class variable and assign the value of the parameter e.g. ```self.rdmin = str(params['rdmin'])```. All parameters here should be strings.

8. Add it to the name of the resulting cfg file ```self.cfgname```. E.g. create a variable ```rdminname = '-rdmin' + self.rdmin```, and add it in ```self.cfgname```.

9. Same applies to the output log file name ```self.outname```.

10. In the method ```cfgfile()```, add a line modifying the template line. Line number starts counting from 0, so if you look at the line number opening the template with vim, subtract one. E.g. ```rdmin``` is in line 97, so the line to modify it is:

```python
cfglines[96] = 'multivariate_rdist_fit_min = ' + self.rdmin
```

I recommend adding the line modification in order of line numbers.        

11. Add it in your config e.g. ```rdmin=480``` or ```rdmin=480,482``` if you enabled multuple values.

12. Done

