# MapReduce in Beam (Python) 2.5

In this lab, you will identify Map and Reduce operations, execute the pipeline, use command line parameters.

## Download Code Repository
**Note:** Compute VM instance need to be created.

Download a code repository for use in this lab. In the **training-vm** SSH terminal enter the following:
```bash
git clone https://github.com/GoogleCloudPlatform/training-data-analyst
```

## Identify Map and Reduce operations
1. Return to the **training-vm** SSH terminal and navigate to the directory `/training-data-analyst/courses/data_analysis/lab2/python` and view the file `is_popular.py` with 
Nano. Do not make any changes to the code. Press Ctrl+X to exit Nano.
```bash
cd ~/training-data-analyst/courses/data_analysis/lab2/python
nano is_popular.py
```

## Execute the pipeline
1. In the training-vm SSH terminal, run the pipeline locally:
```bash
python3 ./is_popular.py
```

2. Identify the output file. It should be output<suffix> and could be a sharded file.
```bash
ls -al /tmp
```

3. Examine the output file, replacing '-*' with the appropriate suffix.
```bash
cat /tmp/output-*
```


## Use command line parameters
1. In the training-vm SSH terminal, change the output prefix from the default value:
```bash
python3 ./is_popular.py --output_prefix=/tmp/myoutput
```
2.What will be the name of the new file that is written out?

3. Note that we now have a new file in the **/tmp** directory:
```bash
ls -lrt /tmp/myoutput*
```
