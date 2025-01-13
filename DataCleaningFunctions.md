### Usual libraries
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

### Standardizing column headers
```
def standardize_column_headers(df):
    df.columns = df.columns.str.lower().str.replace(' ', '').str.replace('_', '')
    print(f'Headers standardized to lowercase and removed whitespace/underscores.')
    return df
```

### Validating email addresses
```
def validate_emails(df, column):
    try:
        email_pattern = r'^[a-zA-Z0-9._%+-]+@([^\d@]+\.[a-zA-Z]{2,})$'
        valid_domains = ['gmail.com', 'hotmail.com', 'yahoo.com', 'outlook.com', 'aol.com', 'icloud.com','naver.com','naver.net','hanmail.net']
        
        df[column] = df[column].str.replace(' ', '', regex=False)

        def clean_and_match_email(email):
            if isinstance(email, str) and '@' in email:
                username, domain = email.split('@', 1)
                domain = re.sub(r'[^a-zA-Z0-9.-]', '', domain).strip().replace(' ', '').lower()
                corrected_domain = difflib.get_close_matches(domain, valid_domains, n=1, cutoff=0.8)
                if corrected_domain:
                    domain = corrected_domain[0]
                return username + '@' + domain
            return email
        
        df[column] = df[column].apply(clean_and_match_email)
        invalid_emails = df[~df[column].str.match(email_pattern, na=False)]

        # Replace invalid emails with an empty string
        df.loc[~df['email'].str.match(email_pattern, na=False), column] = ""
        print(f'Found {len(invalid_emails)} invalid email addresses.')
        return df
    except Exception as e:
        print(f'Error validating emails: {e}')
        return df
```
### Cleaning and standardizing names
```
def clean_and_standardize_names(df, column_first_name):
    try:

        def retain_language_characters(text):
            """
            Retains all language-specific characters and removes everything else.
            This includes Unicode letters from all scripts and spaces.
            """
            if isinstance(text, str):
                # Use regex to match Unicode letters (all scripts) and spaces
                language_pattern = re.compile(r'[^\p{L}\s]', flags=re.UNICODE)
                return language_pattern.sub('', text).strip()
            return text

        df[column_first_name] = df[column_first_name].apply(retain_language_characters)
        df[column_first_name] = df[column_first_name].str.replace(r'\d+', '', regex=True).str.strip()

        def is_invalid_first_name(name: str) -> bool:
            if pd.isna(name) or not isinstance(name, str): return True
            if any(char.isdigit() for char in name) or any(char in set('~!@#$%^&*()_+=[]}{|\\:;,.<>?') for char in name):
                return True
            return False

        invalid_first_names = df[df[column_first_name].apply(is_invalid_first_name)]
        
        def clean_first_name(name):
            return '' if is_invalid_first_name(name) else name.title().strip()

        df[column_first_name] = df[column_first_name].apply(clean_first_name)
        return df
    except Exception as e:
        print(f'Error standardizing names: {e}')
        return df
```

### Validating phone numbers
```
def validate_phone_numbers(df, column):
    try:
        df[column] = df[column].astype(str).str.replace(r'\D', '', regex=True)
        
        def is_invalid_number(number: str) -> bool:
            if not (6 <= len(number) <= 16) or bool(re.search(r'[^0-9+]', number)):
                return True
            return False

        invalid_numbers = df[df[column].apply(is_invalid_number)]
        df.loc[df[column].apply(is_invalid_number), column] = ''

        print(f'Found {len(invalid_numbers)} invalid phone numbers.')
        return df
    except Exception as e:
        print(f'Error validating phone numbers: {e}')
        return df
```

### Removing duplicates
```
def remove_duplicates(df):
    try:
        initial_count = len(df)
        df = df.drop_duplicates()
        final_count = len(df)
        print(f'Removed {initial_count - final_count} duplicate rows.')
        return df
    except Exception as e:
        print(f'Error removing duplicates: {e}')
        return df
```
