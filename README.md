# Movies: Extract, Transform, Load
## Resources
The data in this challenge comes from three sources:

**Wikipedia Movie Data (wiki_movies):** https://2u-data-curriculum-team.s3.amazonaws.com/dataviz-online/module_8/wikipedia-movies.json

**Movies Metadata (kaggle_data):** https://www.kaggle.com/rounakbanik/the-movies-dataset?select=movies_metadata.csv

**MovieLens Ratings Data (ratings):** https://www.kaggle.com/rounakbanik/the-movies-dataset?select=ratings.csv

## Analysis
### Deliverable 1
#### This code uses a function to read in the three relevant data files and creates three data frames with them. 
```
#Import dependencies
import json
import pandas as pd
import numpy as np
import re
import psycopg2
import time
from config import db_pw
from sqlalchemy import create_engine
```
1. Create a function that takes in three arguments: Wikipedia data, Kaggle metadata, and MovieLens rating data (from Kaggle)
```
def movie_data_fn():
```

2. Read in the kaggle metadata and MovieLens ratings CSV files as Pandas DataFrames.
```
    kaggle_metadata = pd.read_csv(f'{file_dir}movies_metadata.csv', low_memory=False)
    ratings = pd.read_csv(f'{file_dir}ratings.csv', low_memory=False)
```
3. Open the Wikipedia data JSON file. 
```   
    with open(f'{file_dir}wikipedia-movies.json', mode='r') as file: 
        wiki_movies_raw = json.load(file) 
```
4. Read in the raw wiki movie data as a Pandas DataFrame.        
```
        wiki_movies_df = pd.DataFrame(wiki_movies_raw)
```
5. Return the three DataFrames      
```
    return wiki_movies_df, kaggle_metadata, ratings 
```

6. Use the variables provided to create a path to the data files 

```
file_dir = 'C://Users/melet/Desktop/Movies-ETL/Resources/'
#file_dir = 'C://Users/M037228/Desktop/um/Movies-ETL/Resources/'
# Wikipedia data
wiki_file = f'{file_dir}/wikipedia_movies.json'
# Kaggle metadata
kaggle_file = f'{file_dir}/movies_metadata.csv'
# MovieLens rating data.
ratings_file = f'{file_dir}/ratings.csv'
```

7. Set the three variables in Step 6 equal to the function created in Step 1.
```
wiki_file, kaggle_file, ratings_file = movie_data_fn()
```

8. Set the DataFrames from the return statement equal to the file names in Step 6. 

```
wiki_movies_df = wiki_file
kaggle_metadata = kaggle_file
ratings = ratings_file
```

9. Check the wiki_movies_df DataFrame.
```
wiki_movies_df.head()

```
10. Check the kaggle_metadata DataFrame.
```
kaggle_metadata.head()
```

11. Check the ratings DataFrame.
```
ratings.head()
```
#### Complete code: 
```
import json
import pandas as pd
import numpy as np
import re
import psycopg2
import time
from sqlalchemy import create_engine

def movie_data_fn():
    kaggle_metadata = pd.read_csv(f'{file_dir}movies_metadata.csv', low_memory=False)
    ratings = pd.read_csv(f'{file_dir}ratings.csv', low_memory=False)
    with open(f'{file_dir}wikipedia-movies.json', mode='r') as file: 
        wiki_movies_raw = json.load(file) 
        wiki_movies_df = pd.DataFrame(wiki_movies_raw)
    return wiki_movies_df, kaggle_metadata, ratings 

#file_dir = 'C://Users/melet/Desktop/Movies-ETL/Resources/'
file_dir = 'C://Users/M037228/Desktop/um/Movies-ETL/Resources/'

# Wikipedia data
wiki_file = f'{file_dir}/wikipedia_movies.json'
# Kaggle metadata
kaggle_file = f'{file_dir}/movies_metadata.csv'
# MovieLens rating data.
ratings_file = f'{file_dir}/ratings.csv'

wiki_file, kaggle_file, ratings_file = movie_data_fn()

wiki_movies_df = wiki_file
kaggle_metadata = kaggle_file
ratings = ratings_file

wiki_movies_df.head()

kaggle_metadata.head()

ratings.head()
```
### Deliverable 2
#### This code uses a function to extract and transform Wikipedia data.

1. Add the clean movie function that takes in the argument, "movie".
```
def clean_movie(movie):
    movie = dict(movie)
    alt_titles = {}
    
    for key in ['Also known as','Arabic','Cantonese','Chinese','French',
                'Hangul','Hebrew','Hepburn','Japanese','Literally',
                'Mandarin','McCune-Reischauer','Original title','Polish',
                'Revised Romanization','Romanized','Russian',
                'Simplified','Traditional','Yiddish']:
        if key in movie:
            alt_titles[key] = movie[key]     
            movie.pop(key)
    if len(alt_titles) > 0:
        movie['alt_titles'] = alt_titles
        
    def change_column_name(old_name, new_name):
        if old_name in movie:
            movie[new_name] = movie.pop(old_name)
    change_column_name('Adaptation by', 'Writer(s)')
    change_column_name('Country of origin', 'Country')
    change_column_name('Directed by', 'Director')
    change_column_name('Distributed by', 'Distributor')
    change_column_name('Edited by', 'Editor(s)')
    change_column_name('Length', 'Running time')
    change_column_name('Original release', 'Release date')
    change_column_name('Music by', 'Composer(s)')
    change_column_name('Produced by', 'Producer(s)')
    change_column_name('Producer', 'Producer(s)')
    change_column_name('Productioncompanies ', 'Production company(s)')
    change_column_name('Productioncompany ', 'Production company(s)')
    change_column_name('Released', 'Release Date')
    change_column_name('Release Date', 'Release date')
    change_column_name('Screen story by', 'Writer(s)')
    change_column_name('Screenplay by', 'Writer(s)')
    change_column_name('Story by', 'Writer(s)')
    change_column_name('Theme music composer', 'Composer(s)')
    change_column_name('Written by', 'Writer(s)')

    return movie
```
2. Add the function from deliverable 1 that takes in three arguments: Wikipedia data, Kaggle metadata, and MovieLens rating data (from Kaggle)
```
def movie_data_extract(wiki_file, kaggle_file, ratings_file):
    kaggle_metadata = pd.read_csv(f'{file_dir}movies_metadata.csv', low_memory=False)
    ratings = pd.read_csv(f'{file_dir}ratings.csv', low_memory=False)
    
    with open(f'{file_dir}wikipedia-movies.json', mode='r') as file:
        wiki_movies_raw = json.load(file)          
```    
3. Write a list comprehension to filter out TV shows.
```
        wiki_movies = [movie for movie in wiki_movies_raw
               if ('Director' in movie or 'Directed by' in movie) 
                   and 'imdb_link' in movie
                   and 'No. of episodes' not in movie]
    print(len(wiki_movies))
```
4. Write a list comprehension to iterate through the cleaned wiki movies list and call the clean_movie function on each movie.
```
    clean_movies = [clean_movie(movie) for movie in wiki_movies]
```
5. Read in the cleaned movies list from Step 4 as a DataFrame.
```
    wiki_movies_df = pd.DataFrame(clean_movies)
```
6. Write a try-except block to catch errors while extracting the IMDb ID using a regular expression string, dropping any imdb_id duplicates. If there is an error, capture and print the exception.
```
    try:
        wiki_movies_df['imdb_id'] = wiki_movies_df['imdb_link'].str.extract(r'(tt\d{7})')
        wiki_movies_df.drop_duplicates(subset='imdb_id', inplace=True)
    except:
        print(f'ERROR:{Exception} - imdb_id cannot be extracted from {imdb_link}')
```
7. Write a list comprehension to keep the columns that don't have null values from the wiki_movies_df DataFrame.  
```
    wiki_columns_to_keep = [column for column in wiki_movies_df.columns if wiki_movies_df[column].isnull().sum() < len(wiki_movies_df) * 0.9]
    wiki_movies_df = wiki_movies_df[wiki_columns_to_keep]
```  
8. Create a variable that will hold the non-null values from the “Box office” column.
```
    box_office = wiki_movies_df['Box office'].dropna()
```   
9. Convert the box office data created in Step 8 to string values using the lambda and join functions
```
    box_office.apply(lambda x: ' '.join(x) if type(x) == list else x)
```
10. Write a regular expression to match the six elements of "form_one" of the box office data.
```
    form_one = r'\$\s*\d+\.?\d*\s*[mb]illi?on'
```
11. Write a regular expression to match the three elements of "form_two" of the box office data.
```
    form_two = r'\$\s*\d{1,3}(?:[,\.]\d{3})+(?!\s[mb]illion)'
```
12. Add the parse_dollars function.
```
    def parse_dollars(s):
        if type(s) != str:
            return np.nan
        if re.match(r'\$\s*\d+\.?\d*\s*milli?on', s, flags=re.IGNORECASE):
            s = re.sub('\$|\s|[a-zA-Z]','', s)
            value = float(s) * 10**6
            return value
        elif re.match(r'\$\s*\d+\.?\d*\s*billi?on', s, flags=re.IGNORECASE):
            s = re.sub('\$|\s|[a-zA-Z]','', s)
            value = float(s) * 10**9
            return value
        elif re.match(r'\$\s*\d{1,3}(?:[,\.]\d{3})+(?!\s[mb]illion)', s, flags=re.IGNORECASE):
            s = re.sub('\$|,','', s)
            value = float(s)
            return value
        else:
            return np.nan
```        
13. Clean the box office column in the wiki_movies_df DataFrame.
```
    box_office = box_office.str.replace(r'\$.*[-—–](?![a-z])', '$', regex=True)
    wiki_movies_df['box_office'] = box_office.str.extract(f'({form_one}|{form_two})', flags=re.IGNORECASE)[0].apply(parse_dollars)
    wiki_movies_df.drop('Box office', axis=1, inplace=True)
```
14. Clean the budget column in the wiki_movies_df DataFrame.
```
    budget = wiki_movies_df['Budget'].dropna()
    budget = budget.map(lambda x: ' '.join(x) if type(x) == list else x)
    budget = budget.str.replace(r'\$.*[-—–](?![a-z])', '$', regex=True)
    budget = budget.str.replace(r'\[\d+\]\s*', '', regex=True)
    wiki_movies_df['budget'] = budget.str.extract(f'({form_one}|{form_two})', flags=re.IGNORECASE)[0].apply(parse_dollars)
```
15. Clean the release date column in the wiki_movies_df DataFrame.
```
    release_date = wiki_movies_df['Release date'].dropna().apply(lambda x: ' '.join(x) if type(x) == list else x)
    date_form_one = r'(?:January|February|March|April|May|June|July|August|September|October|November|December)\s[123]?\d,\s\d{4}'
    date_form_two = r'\d{4}.[01]\d.[0123]\d'
    date_form_three = r'(?:January|February|March|April|May|June|July|August|September|October|November|December)\s\d{4}'
    date_form_four = r'\d{4}'
    release_date.str.extract(f'({date_form_one}|{date_form_two}|{date_form_three}|{date_form_four})', flags=re.IGNORECASE)
    wiki_movies_df['release_date'] = pd.to_datetime(release_date.str.extract(f'({date_form_one}|{date_form_two}|{date_form_three}|{date_form_four})')[0], infer_datetime_format=True)
```
16. Clean the running time column in the wiki_movies_df DataFrame.
```
    running_time = wiki_movies_df['Running time'].dropna().apply(lambda x: ' '.join(x) if type(x) == list else x)
    running_time_extract = running_time.str.extract(r'(\d+)\s*ho?u?r?s?\s*(\d*)|(\d+)\s*m')
    running_time_extract = running_time_extract.apply(lambda col: pd.to_numeric(col, errors='coerce')).fillna(0)
    wiki_movies_df['running_time'] = running_time_extract.apply(lambda row: row[0]*60 + row[1] if row[2] == 0 else row[2], axis=1)
    
    return wiki_movies_df, kaggle_metadata, ratings 
```
17. Create the path to your file directory and variables for the three files.
```
file_dir = 'C://Users/M037228/Desktop/um/Movies-ETL/Resources/'
#file_dir = 'C://Users/melet/um/hw/Movies-ETL/Resources/'
wiki_file = f'{file_dir}/wikipedia_movies.json'
kaggle_file = f'{file_dir}/movies_metadata.csv'
ratings_file = f'{file_dir}/ratings.csv'
```
18 and 19. Set the three variables equal to the function created in D1, and set the wiki_movies_df equal to the wiki_file variable. 

```
wiki_movies_df, kaggle_metadata, ratings = movie_data_extract(wiki_file, kaggle_file, ratings_file)
```
20. Check that the wiki_movies_df DataFrame looks like this. 
```
wiki_movies_df.head()
```
21. Check that wiki_movies_df DataFrame columns are correct. 
```
wiki_movies_df.columns.to_list()

```
#### Complete Code
```
import json
import pandas as pd
import numpy as np
import re
from sqlalchemy import create_engine
import psycopg2
import time

def clean_movie(movie):
    movie = dict(movie)
    alt_titles = {}
    
    for key in ['Also known as','Arabic','Cantonese','Chinese','French',
                'Hangul','Hebrew','Hepburn','Japanese','Literally',
                'Mandarin','McCune-Reischauer','Original title','Polish',
                'Revised Romanization','Romanized','Russian',
                'Simplified','Traditional','Yiddish']:
        if key in movie:
            alt_titles[key] = movie[key]     
            movie.pop(key)
    if len(alt_titles) > 0:
        movie['alt_titles'] = alt_titles
        
    def change_column_name(old_name, new_name):
        if old_name in movie:
            movie[new_name] = movie.pop(old_name)
    change_column_name('Adaptation by', 'Writer(s)')
    change_column_name('Country of origin', 'Country')
    change_column_name('Directed by', 'Director')
    change_column_name('Distributed by', 'Distributor')
    change_column_name('Edited by', 'Editor(s)')
    change_column_name('Length', 'Running time')
    change_column_name('Original release', 'Release date')
    change_column_name('Music by', 'Composer(s)')
    change_column_name('Produced by', 'Producer(s)')
    change_column_name('Producer', 'Producer(s)')
    change_column_name('Productioncompanies ', 'Production company(s)')
    change_column_name('Productioncompany ', 'Production company(s)')
    change_column_name('Released', 'Release Date')
    change_column_name('Release Date', 'Release date')
    change_column_name('Screen story by', 'Writer(s)')
    change_column_name('Screenplay by', 'Writer(s)')
    change_column_name('Story by', 'Writer(s)')
    change_column_name('Theme music composer', 'Composer(s)')
    change_column_name('Written by', 'Writer(s)')

    return movie

def movie_data_extract(wiki_file, kaggle_file, ratings_file):
    kaggle_metadata = pd.read_csv(f'{file_dir}movies_metadata.csv', low_memory=False)
    ratings = pd.read_csv(f'{file_dir}ratings.csv', low_memory=False)
    
    with open(f'{file_dir}wikipedia-movies.json', mode='r') as file:
        wiki_movies_raw = json.load(file)          
        wiki_movies = [movie for movie in wiki_movies_raw
               if ('Director' in movie or 'Directed by' in movie) 
                   and 'imdb_link' in movie
                   and 'No. of episodes' not in movie]
    print(len(wiki_movies))

    clean_movies = [clean_movie(movie) for movie in wiki_movies]
    wiki_movies_df = pd.DataFrame(clean_movies)
    try:
        wiki_movies_df['imdb_id'] = wiki_movies_df['imdb_link'].str.extract(r'(tt\d{7})')
        wiki_movies_df.drop_duplicates(subset='imdb_id', inplace=True)
    except:
        print(f'ERROR:{Exception} - imdb_id cannot be extracted from {imdb_link}')

    wiki_columns_to_keep = [column for column in wiki_movies_df.columns if wiki_movies_df[column].isnull().sum() < len(wiki_movies_df) * 0.9]
    wiki_movies_df = wiki_movies_df[wiki_columns_to_keep]
    
    box_office = wiki_movies_df['Box office'].dropna()
    box_office.apply(lambda x: ' '.join(x) if type(x) == list else x)
    form_one = r'\$\s*\d+\.?\d*\s*[mb]illi?on'
    form_two = r'\$\s*\d{1,3}(?:[,\.]\d{3})+(?!\s[mb]illion)'

    def parse_dollars(s):
        if type(s) != str:
            return np.nan
        if re.match(r'\$\s*\d+\.?\d*\s*milli?on', s, flags=re.IGNORECASE):
            s = re.sub('\$|\s|[a-zA-Z]','', s)
            value = float(s) * 10**6
            return value
        elif re.match(r'\$\s*\d+\.?\d*\s*billi?on', s, flags=re.IGNORECASE):
            s = re.sub('\$|\s|[a-zA-Z]','', s)
            value = float(s) * 10**9
            return value
        elif re.match(r'\$\s*\d{1,3}(?:[,\.]\d{3})+(?!\s[mb]illion)', s, flags=re.IGNORECASE):
            s = re.sub('\$|,','', s)
            value = float(s)
            return value
        else:
            return np.nan
        
    box_office = box_office.str.replace(r'\$.*[-—–](?![a-z])', '$', regex=True)
    wiki_movies_df['box_office'] = box_office.str.extract(f'({form_one}|{form_two})', flags=re.IGNORECASE)[0].apply(parse_dollars)
    wiki_movies_df.drop('Box office', axis=1, inplace=True)

    budget = wiki_movies_df['Budget'].dropna()
    budget = budget.map(lambda x: ' '.join(x) if type(x) == list else x)
    budget = budget.str.replace(r'\$.*[-—–](?![a-z])', '$', regex=True)
    budget = budget.str.replace(r'\[\d+\]\s*', '', regex=True)
    wiki_movies_df['budget'] = budget.str.extract(f'({form_one}|{form_two})', flags=re.IGNORECASE)[0].apply(parse_dollars)

    release_date = wiki_movies_df['Release date'].dropna().apply(lambda x: ' '.join(x) if type(x) == list else x)
    date_form_one = r'(?:January|February|March|April|May|June|July|August|September|October|November|December)\s[123]?\d,\s\d{4}'
    date_form_two = r'\d{4}.[01]\d.[0123]\d'
    date_form_three = r'(?:January|February|March|April|May|June|July|August|September|October|November|December)\s\d{4}'
    date_form_four = r'\d{4}'
    release_date.str.extract(f'({date_form_one}|{date_form_two}|{date_form_three}|{date_form_four})', flags=re.IGNORECASE)
    wiki_movies_df['release_date'] = pd.to_datetime(release_date.str.extract(f'({date_form_one}|{date_form_two}|{date_form_three}|{date_form_four})')[0], infer_datetime_format=True)

    running_time = wiki_movies_df['Running time'].dropna().apply(lambda x: ' '.join(x) if type(x) == list else x)
    running_time_extract = running_time.str.extract(r'(\d+)\s*ho?u?r?s?\s*(\d*)|(\d+)\s*m')
    running_time_extract = running_time_extract.apply(lambda col: pd.to_numeric(col, errors='coerce')).fillna(0)
    wiki_movies_df['running_time'] = running_time_extract.apply(lambda row: row[0]*60 + row[1] if row[2] == 0 else row[2], axis=1)
    
    return wiki_movies_df, kaggle_metadata, ratings     

file_dir = 'C://Users/M037228/Desktop/um/Movies-ETL/Resources/'
wiki_file = f'{file_dir}/wikipedia_movies.json'
kaggle_file = f'{file_dir}/movies_metadata.csv'
ratings_file = f'{file_dir}/ratings.csv'    

wiki_movies_df, kaggle_metadata, ratings = movie_data_extract(wiki_file, kaggle_file, ratings_file)

wiki_movies_df.head()

wiki_movies_df.columns.to_list()
```
### Deliverable 3
#### This code uses a function to extract and transform Kaggle metadata and MovieLens ratings data, convert transformed data to dataframes, and merge dataframes.

The function above was edited to include the following:
```
# 1. Add function that reads in the three data files
def extract_transform_return(wiki_file, kaggle_file, ratings_file):
    start_time = time.time()
    kaggle_metadata = pd.read_csv(f'{file_dir}movies_metadata.csv', low_memory=False)
    ratings = pd.read_csv(f'{file_dir}ratings.csv', low_memory=False)
    
    # Extract data: wiki_movies
    with open(f'{file_dir}wikipedia-movies.json', mode='r') as file:
        wiki_movies_raw = json.load(file)            
        wiki_movies = [movie for movie in wiki_movies_raw
                       if 'No. of episodes' not in movie]
        clean_movies = [clean_movie(movie) for movie in wiki_movies]
        wiki_movies_df = pd.DataFrame(clean_movies)      
    try:
        wiki_movies_df['imdb_id'] = wiki_movies_df['imdb_link'].str.extract(r'(tt\d{7})')
        wiki_movies_df.drop_duplicates(subset='imdb_id', inplace=True)
    except:
        print(f'ERROR:{Exception} - imdb_id cannot be extracted from {imdb_link}')
    wiki_columns_to_keep = [column for column in wiki_movies_df.columns if wiki_movies_df[column].isnull().sum() < len(wiki_movies_df) * 0.9]
    wiki_movies_df = wiki_movies_df[wiki_columns_to_keep]

    #clean box office data
    box_office = wiki_movies_df['Box office'].dropna()
    box_office.apply(lambda x: ' '.join(x) if type(x) == list else x)
    form_one = r'\$\s*\d+\.?\d*\s*[mb]illi?on'
    form_two = r'\$\s*\d{1,3}(?:[,\.]\d{3})+(?!\s[mb]illion)'
    def parse_dollars(s):
        if type(s) != str:
            return np.nan
        if re.match(r'\$\s*\d+\.?\d*\s*milli?on', s, flags=re.IGNORECASE):
            s = re.sub('\$|\s|[a-zA-Z]','', s)
            value = float(s) * 10**6
            return value
        elif re.match(r'\$\s*\d+\.?\d*\s*billi?on', s, flags=re.IGNORECASE):
            s = re.sub('\$|\s|[a-zA-Z]','', s)
            value = float(s) * 10**9
            return value
        elif re.match(r'\$\s*\d{1,3}(?:[,\.]\d{3})+(?!\s[mb]illion)', s, flags=re.IGNORECASE):
            s = re.sub('\$|,','', s)
            value = float(s)
            return value
        else:
            return np.nan
    box_office = box_office.str.replace(r'\$.*[-—–](?![a-z])', '$', regex=True)
    wiki_movies_df['box_office'] = box_office.str.extract(f'({form_one}|{form_two})', flags=re.IGNORECASE)[0].apply(parse_dollars)
    
    #clean budget 
    budget = wiki_movies_df['Budget'].dropna()
    budget = budget.map(lambda x: ' '.join(x) if type(x) == list else x)
    budget = budget.str.replace(r'\$.*[-—–](?![a-z])', '$', regex=True)
    budget = budget.str.replace(r'\[\d+\]\s*', '')
    wiki_movies_df['budget'] = budget.str.extract(f'({form_one}|{form_two})', flags=re.IGNORECASE)[0].apply(parse_dollars)

    #clean release date
    release_date = wiki_movies_df['Release date'].dropna().apply(lambda x: ' '.join(x) if type(x) == list else x)
    date_form_one = r'(?:January|February|March|April|May|June|July|August|September|October|November|December)\s[123]?\d,\s\d{4}'
    date_form_two = r'\d{4}.[01]\d.[0123]\d'
    date_form_three = r'(?:January|February|March|April|May|June|July|August|September|October|November|December)\s\d{4}'
    date_form_four = r'\d{4}'
    release_date.str.extract(f'({date_form_one}|{date_form_two}|{date_form_three}|{date_form_four})', flags=re.IGNORECASE)
    wiki_movies_df['release_date'] = pd.to_datetime(release_date.str.extract(f'({date_form_one}|{date_form_two}|{date_form_three}|{date_form_four})')[0], infer_datetime_format=True)
    
    #clean running time
    running_time = wiki_movies_df['Running time'].dropna().apply(lambda x: ' '.join(x) if type(x) == list else x)
    running_time_extract = running_time.str.extract(r'(\d+)\s*ho?u?r?s?\s*(\d*)|(\d+)\s*m')
    running_time_extract = running_time_extract.apply(lambda col: pd.to_numeric(col, errors='coerce')).fillna(0)
    wiki_movies_df['running_time'] = running_time_extract.apply(lambda row: row[0]*60 + row[1] if row[2] == 0 else row[2], axis=1)
    
# 2. Clean the Kaggle metadata.
    kaggle_metadata = kaggle_metadata[kaggle_metadata['adult'] == 'False'].drop('adult',axis='columns')
    kaggle_metadata['video'] = kaggle_metadata['video'] == 'True'
    kaggle_metadata['budget'] = kaggle_metadata['budget'].astype(int)
    kaggle_metadata['id'] = pd.to_numeric(kaggle_metadata['id'], errors='raise')
    kaggle_metadata['popularity'] = pd.to_numeric(kaggle_metadata['popularity'], errors='raise')
    kaggle_metadata['release_date'] = pd.to_datetime(kaggle_metadata['release_date'])
    ratings['timestamp'] = pd.to_datetime(ratings['timestamp'], unit='s')

# 3. Merge the two DataFrames into the movies DataFrame.
    movies_df = pd.merge(wiki_movies_df, kaggle_metadata, on='imdb_id', suffixes=['_wiki','_kaggle'])

# 4. Drop unnecessary columns from the merged DataFrame.
    movies_df.drop(columns=['title_wiki','release_date_wiki','Language','Production company(s)'], inplace=True)

# 5. Add in the function to fill in the missing Kaggle data.
    def fill_missing_kaggle_data(df, kaggle_column, wiki_column):
        df[kaggle_column] = df.apply(
            lambda row: row[wiki_column] if row[kaggle_column] == 0 else row[kaggle_column]
            , axis=1)
        df.drop(columns=wiki_column, inplace=True)

# 6. Call the function in Step 5 with the DataFrame and columns as the arguments.
    fill_missing_kaggle_data(movies_df, 'runtime', 'running_time')
    fill_missing_kaggle_data(movies_df, 'budget_kaggle', 'budget_wiki')
    fill_missing_kaggle_data(movies_df, 'revenue', 'box_office')

# 7. Filter the movies DataFrame for specific columns.
    movies_df = movies_df.loc[:, ['imdb_id',
                                  'id',
                                  'title_kaggle',
                                  'original_title',
                                  'tagline',
                                  'belongs_to_collection',
                                  'url',
                                  'imdb_link',
                                  'runtime',
                                  'budget_kaggle',
                                  'revenue',
                                  'release_date_kaggle',
                                  'popularity',
                                  'vote_average',
                                  'vote_count',
                                  'genres',
                                  'original_language',
                                  'overview',
                                  'spoken_languages',
                                  'Country',
                                  'production_companies',
                                  'production_countries',
                                  'Distributor',
                                  'Producer(s)',
                                  'Director',
                                  'Starring',
                                  'Cinematography',
                                  'Editor(s)',
                                  'Writer(s)',
                                  'Composer(s)',
                                  'Based on'
                          ]]

# 8. Rename the columns in the movies DataFrame.
    movies_df.rename({'id':'kaggle_id',
                  'title_kaggle':'title',
                  'url':'wikipedia_url',
                  'budget_kaggle':'budget',
                  'release_date_kaggle':'release_date',
                  'Country':'country',
                  'Distributor':'distributor',
                  'Producer(s)':'producers',
                  'Director':'director',
                  'Starring':'starring',
                  'Cinematography':'cinematography',
                  'Editor(s)':'editors',
                  'Writer(s)':'writers',
                  'Composer(s)':'composers',
                  'Based on':'based_on'
                 }, axis='columns', inplace=True)


# 9. Transform and merge the ratings DataFrame.
    rating_counts = ratings.groupby(['movieId','rating'], as_index=False).count() \
                    .rename({'userId':'count'}, axis=1) \
                    .pivot(index='movieId',columns='rating', values='count')
    
    rating_counts.columns = ['rating_' + str(col) for col in rating_counts.columns]
    movies_with_ratings_df = pd.merge(movies_df, rating_counts, left_on='kaggle_id', right_index=True, how='left')
    movies_with_ratings_df[rating_counts.columns] = movies_with_ratings_df[rating_counts.columns].fillna(0)
 
    print(f'{time.time() - start_time} total seconds elapsed')
    return wiki_movies_df, movies_with_ratings_df, movies_df
   ```
   
   ```
# 10. Create the path to your file directory and variables for the three files.
file_dir = 'C://Users/melet/um/hw/Movies-ETL/Resources/'
wiki_file = f'{file_dir}/wikipedia_movies.json'
kaggle_file = f'{file_dir}/movies_metadata.csv'
ratings_file = f'{file_dir}/ratings.csv'

# 11. Set the three variables equal to the function created in D1.
# 12. Set the DataFrames from the return statement equal to the file names in Step 11. 
wiki_movies_df, movies_with_ratings_df, movies_df = extract_transform_return(wiki_file, kaggle_file, ratings_file)

# 13. Check the wiki_movies_df DataFrame. 
wiki_movies_df.head()

# 14. Check the movies_with_ratings_df DataFrame.
movies_with_ratings_df.head()

# 15. Check the movies_df DataFrame. 
movies_df.head()
```


### Deliverable 4
#### This code will load the final dataframe: movies_df to a SQL database

```
    # Transform and merge the ratings DataFrame.
    db_string = f"postgresql://postgres:{db_pw}@127.0.0.1:5432/movie_data" 
    engine = create_engine(db_string)
    movies_df.to_sql(name='movies', con=engine, if_exists='replace')
    rows_imported = 0
    for data in pd.read_csv(f'{file_dir}ratings.csv', chunksize=1000000):
        print(f'importing rows {rows_imported} to {rows_imported + len(data)}...', end='')
        data.to_sql(name='ratings', con=engine, if_exists='append')
        rows_imported += len(data)

    print(f'Done. {time.time() - start_time} total seconds elapsed')
```

```
def extract_transform_load():
    start_time = time.time()
    kaggle_metadata = pd.read_csv(f'{file_dir}movies_metadata.csv', low_memory=False)
    ratings = pd.read_csv(f'{file_dir}ratings.csv', low_memory=False)
    
    # Extract data: wiki_movies
    with open(f'{file_dir}wikipedia-movies.json', mode='r') as file:
        wiki_movies_raw = json.load(file)
        wiki_movies = [movie for movie in wiki_movies_raw
            if ('Director' in movie or 'Directed by' in movie) 
            and 'imdb_link' in movie
            and 'No. of episodes' not in movie]
        
        clean_movies = [clean_movie(movie) for movie in wiki_movies]
        wiki_movies_df = pd.DataFrame(clean_movies)         
    try:
        wiki_movies_df['imdb_id'] = wiki_movies_df['imdb_link'].str.extract(r'(tt\d{7})')
        wiki_movies_df.drop_duplicates(subset='imdb_id', inplace=True)
    except:
        print(f'ERROR:{Exception} - imdb_id cannot be extracted from {imdb_link}')

    wiki_columns_to_keep = [column for column in wiki_movies_df.columns if wiki_movies_df[column].isnull().sum() < len(wiki_movies_df) * 0.9]
    wiki_movies_df = wiki_movies_df[wiki_columns_to_keep]

    #clean box office data
    box_office = wiki_movies_df['Box office'].dropna()
    box_office.apply(lambda x: ' '.join(x) if type(x) == list else x)
    form_one = r'\$\s*\d+\.?\d*\s*[mb]illi?on'
    form_two = r'\$\s*\d{1,3}(?:[,\.]\d{3})+(?!\s[mb]illion)'
    def parse_dollars(s):
        if type(s) != str:
            return np.nan
        if re.match(r'\$\s*\d+\.?\d*\s*milli?on', s, flags=re.IGNORECASE):
            s = re.sub('\$|\s|[a-zA-Z]','', s)
            value = float(s) * 10**6
            return value
        elif re.match(r'\$\s*\d+\.?\d*\s*billi?on', s, flags=re.IGNORECASE):
            s = re.sub('\$|\s|[a-zA-Z]','', s)
            value = float(s) * 10**9
            return value
        elif re.match(r'\$\s*\d{1,3}(?:[,\.]\d{3})+(?!\s[mb]illion)', s, flags=re.IGNORECASE):
            s = re.sub('\$|,','', s)
            value = float(s)
            return value
        else:
            return np.nan
    box_office = box_office.str.replace(r'\$.*[-—–](?![a-z])', '$', regex=True)
    wiki_movies_df['box_office'] = box_office.str.extract(f'({form_one}|{form_two})', flags=re.IGNORECASE)[0].apply(parse_dollars)
    wiki_movies_df.drop('Box office', axis=1, inplace=True)
    
    #clean budget 
    budget = wiki_movies_df['Budget'].dropna()
    budget = budget.map(lambda x: ' '.join(x) if type(x) == list else x)
    budget = budget.str.replace(r'\$.*[-—–](?![a-z])', '$', regex=True)
    budget = budget.str.replace(r'\[\d+\]\s*', '', regex=True)
    wiki_movies_df['budget'] = budget.str.extract(f'({form_one}|{form_two})', flags=re.IGNORECASE)[0].apply(parse_dollars)
    wiki_movies_df.drop('Budget', axis=1, inplace=True)    
    
    #clean release date
    release_date = wiki_movies_df['Release date'].dropna().apply(lambda x: ' '.join(x) if type(x) == list else x)
    date_form_one = r'(?:January|February|March|April|May|June|July|August|September|October|November|December)\s[123]?\d,\s\d{4}'
    date_form_two = r'\d{4}.[01]\d.[0123]\d'
    date_form_three = r'(?:January|February|March|April|May|June|July|August|September|October|November|December)\s\d{4}'
    date_form_four = r'\d{4}'
    release_date.str.extract(f'({date_form_one}|{date_form_two}|{date_form_three}|{date_form_four})', flags=re.IGNORECASE)
    wiki_movies_df['release_date'] = pd.to_datetime(release_date.str.extract(f'({date_form_one}|{date_form_two}|{date_form_three}|{date_form_four})')[0], infer_datetime_format=True)
    
    #clean running time
    running_time = wiki_movies_df['Running time'].dropna().apply(lambda x: ' '.join(x) if type(x) == list else x)
    running_time_extract = running_time.str.extract(r'(\d+)\s*ho?u?r?s?\s*(\d*)|(\d+)\s*m')
    running_time_extract = running_time_extract.apply(lambda col: pd.to_numeric(col, errors='coerce')).fillna(0)
    wiki_movies_df['running_time'] = running_time_extract.apply(lambda row: row[0]*60 + row[1] if row[2] == 0 else row[2], axis=1)
    wiki_movies_df.drop('Running time', axis=1, inplace=True)
    
    # clean kaggle metadata
    kaggle_metadata = kaggle_metadata[kaggle_metadata['adult'] == 'False'].drop('adult',axis='columns')
    kaggle_metadata['video'] = kaggle_metadata['video'] == 'True'
    kaggle_metadata['budget'] = kaggle_metadata['budget'].astype(int)
    kaggle_metadata['id'] = pd.to_numeric(kaggle_metadata['id'], errors='raise')
    kaggle_metadata['popularity'] = pd.to_numeric(kaggle_metadata['popularity'], errors='raise')
    kaggle_metadata['release_date'] = pd.to_datetime(kaggle_metadata['release_date'])
    ratings['timestamp'] = pd.to_datetime(ratings['timestamp'], unit='s')

    #combine dataframes
    movies_df = pd.merge(wiki_movies_df, kaggle_metadata, on='imdb_id', suffixes=['_wiki','_kaggle'])
    movies_df.drop(columns=['title_wiki','release_date_wiki','Language','Production company(s)'], inplace=True)

    #fill missing data
    def fill_missing_kaggle_data(df, kaggle_column, wiki_column):
        df[kaggle_column] = df.apply(
            lambda row: row[wiki_column] if row[kaggle_column] == 0 else row[kaggle_column]
            , axis=1)
        df.drop(columns=wiki_column, inplace=True)
    fill_missing_kaggle_data(movies_df, 'runtime', 'running_time')
    fill_missing_kaggle_data(movies_df, 'budget_kaggle', 'budget_wiki')
    fill_missing_kaggle_data(movies_df, 'revenue', 'box_office')

    #find and rename columns
    movies_df = movies_df.loc[:, ['imdb_id',
                                  'id',
                                  'title_kaggle',
                                  'original_title',
                                  'tagline',
                                  'belongs_to_collection',
                                  'url',
                                  'imdb_link',
                                  'runtime',
                                  'budget_kaggle',
                                  'revenue',
                                  'release_date_kaggle',
                                  'popularity',
                                  'vote_average',
                                  'vote_count',
                                  'genres',
                                  'original_language',
                                  'overview',
                                  'spoken_languages',
                                  'Country',
                                  'production_companies',
                                  'production_countries',
                                  'Distributor',
                                  'Producer(s)',
                                  'Director',
                                  'Starring',
                                  'Cinematography',
                                  'Editor(s)',
                                  'Writer(s)',
                                  'Composer(s)',
                                  'Based on'
                          ]]
    movies_df.rename({'id':'kaggle_id',
                  'title_kaggle':'title',
                  'url':'wikipedia_url',
                  'budget_kaggle':'budget',
                  'release_date_kaggle':'release_date',
                  'Country':'country',
                  'Distributor':'distributor',
                  'Producer(s)':'producers',
                  'Director':'director',
                  'Starring':'starring',
                  'Cinematography':'cinematography',
                  'Editor(s)':'editors',
                  'Writer(s)':'writers',
                  'Composer(s)':'composers',
                  'Based on':'based_on'
                 }, axis='columns', inplace=True)

    # Transform and merge the ratings DataFrame.
    db_string = f"postgresql://postgres:{db_pw}@127.0.0.1:5432/movie_data" 
    engine = create_engine(db_string)
    movies_df.to_sql(name='movies', con=engine, if_exists='replace')
    rows_imported = 0
    for data in pd.read_csv(f'{file_dir}ratings.csv', chunksize=1000000):
        print(f'importing rows {rows_imported} to {rows_imported + len(data)}...', end='')
        data.to_sql(name='ratings', con=engine, if_exists='append')
        rows_imported += len(data)

    print(f'Done. {time.time() - start_time} total seconds elapsed')
    ```
