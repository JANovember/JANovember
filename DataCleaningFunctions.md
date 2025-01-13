# Usual Imports
### Necessary libraries
```
import csv
import pandas as pd
import numpy as np
import regex as re
import difflib
import logging
```
### Set up logging (for a Jupyter notebook, we will print logs directly to output)
```
def setup_logging():
    logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')
```
### Read a CSV file and apply the result to a dataframe named 'df'
```
def read_csv(file_path, sep=';', header=0, engine=None):
    try:
        df = pd.read_csv(file_path, sep=sep, header=header, engine=engine)
        return df
    except Exception as e:
        print(f'Error reading csv: {e}')
        return None
```
