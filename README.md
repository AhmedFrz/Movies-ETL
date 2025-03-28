# Movie Data ETL Pipeline

<p align="center">
  <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/pandas-%23150458.svg?style=for-the-badge&logo=pandas&logoColor=white" />
  <img src="https://img.shields.io/badge/numpy-%23013243.svg?style=for-the-badge&logo=numpy&logoColor=white" />
  <img src="https://img.shields.io/badge/Jupyter-F37626.svg?&style=for-the-badge&logo=Jupyter&logoColor=white" />
  <img src="https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white" />
  <img src="https://img.shields.io/badge/SQLAlchemy-FCA121?style=for-the-badge&logo=SQLAlchemy&logoColor=white" />
  <img src="https://img.shields.io/badge/regex-%23F29111.svg?style=for-the-badge&logo=regex&logoColor=white" />
</p>

## Overview

This project implements a robust Extract, Transform, Load (ETL) pipeline that processes raw movie data from Wikipedia and Kaggle, cleans and merges the data, and loads it into a PostgreSQL database for analysis. The pipeline handles multiple data formats, resolves inconsistencies, and creates a unified dataset suitable for business intelligence and data science applications.

## Purpose

### Research Questions
- How can we efficiently consolidate disparate movie data sources into a single, reliable database?
- What techniques can effectively clean and normalize inconsistent data formats from different sources?
- How can we implement an automated pipeline that processes large datasets with minimal manual intervention?

### Project Rationale
The entertainment industry deals with vast amounts of data from various sources, often in inconsistent formats. This project addresses the need for:
- A unified, reliable data source for movie analytics
- Automated data cleaning and normalization processes
- Efficient handling of large datasets (26M+ ratings)

### Expected Learning Outcomes
- Designing and implementing complete ETL pipelines
- Applying data cleaning techniques to real-world, messy datasets
- Developing regular expression skills for text parsing and normalization
- Building proficiency in database integration with pandas

## Tools and Technologies

### Core Technologies
| Technology | Use in Project |
|------------|----------------|
| Python 3.x | Primary programming language |
| Pandas | Data manipulation and transformation |
| NumPy | Numerical operations and handling NaN values |
| Jupyter Notebooks | Development environment |
| PostgreSQL | Relational database for data storage |
| SQLAlchemy | Database connection and ORM |
| Regular Expressions | Pattern matching for data extraction and cleaning |
| JSON | Processing Wikipedia data |
| CSV | Processing Kaggle metadata and ratings |

### Alternative Technologies
- **Apache Spark**: For processing larger datasets
- **Airflow**: For scheduling and monitoring workflows
- **NoSQL databases**: For handling more varied data structures
- **Docker**: For containerizing the ETL process

### Skills Demonstrated
- Data extraction from multiple file formats
- Data cleaning and normalization
- Error handling with try/except blocks
- Function development for reusable ETL operations
- Database connection and data loading
- Processing large datasets in chunks

## Project Structure

The ETL pipeline is divided into four deliverables:

### 1. Function Development (ETL_function_test_Del_1.ipynb)
- Create the initial ETL function structure
- Set up data extraction from source files (JSON, CSV)
- Implement basic dataframe creation

### 2. Data Extraction & Cleaning (ETL_clean_wiki_movies_Del_2.ipynb)
- Extract and filter Wikipedia movie data
- Parse IMDb IDs using regular expressions
- Clean box office, budget, release date, and running time data
- Transform text to numeric values

### 3. Data Transformation (ETL_clean_kaggle_data_Del_3.ipynb)
- Clean and transform Kaggle metadata
- Merge Wikipedia and Kaggle datasets
- Resolve data conflicts between sources
- Create final movies dataframe

### 4. Database Loading (ETL_create_database_Del_4.ipynb)
- Configure PostgreSQL connection
- Create database tables
- Load processed data into SQL
- Implement chunking for large ratings dataset

## Methodology

### ETL Workflow

```mermaid
graph LR
    A[Raw Data Sources] --> B[Extraction]
    B --> C[Transformation]
    C --> D[Loading]
    
    subgraph Sources
    A1[Wikipedia JSON] --> A
    A2[Kaggle CSVs] --> A
    end
    
    subgraph Extraction
    B1[Data Reading] --> B
    B2[Filtering] --> B
    B3[Parsing] --> B
    end
    
    subgraph Transformation
    C1[Data Cleaning] --> C
    C2[Merging] --> C
    C3[Formatting] --> C
    end
    
    subgraph Loading
    D1[PostgreSQL] --> D
    D2[Table Creation] --> D
    D3[Data Insertion] --> D
    end
```

### Why This Methodology
1. **Clear separation of concerns**: Divides data pipeline into distinct stages
2. **Modularity**: Allows for incremental development and testing
3. **Scalability**: Can handle growing data volumes with chunk processing
4. **Industry standard**: Follows established best practices for data integration

### Key Implementation Decisions
- **Function-based approach**: Creates reusable, modular code
- **Regular expressions**: Provides consistent data parsing
- **Lambda functions**: Enables efficient data transformations
- **SQLAlchemy**: Abstracts database connection and management
- **Chunk processing**: Allows handling large datasets within memory constraints

## Implementation Details

### Key Functions

<details>
<summary>Data Cleaning Function</summary>

```python
def clean_movie(movie):
    """
    Clean and standardize movie data dictionary.
    
    Args:
        movie (dict): Raw movie data
        
    Returns:
        dict: Cleaned movie data with standardized keys
    """
    movie = dict(movie)  # Create a non-destructive copy
    alt_titles = {}
    
    # Consolidate alternate titles
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
        
    # Standardize column names
    def change_column_name(old_name, new_name):
        if old_name in movie:
            movie[new_name] = movie.pop(old_name)
            
    # Apply column name standardization
    change_column_name('Adaptation by', 'Writer(s)')
    change_column_name('Country of origin', 'Country')
    change_column_name('Directed by', 'Director')
    # ... additional column mappings ...
    
    return movie
```
</details>

<details>
<summary>Parse Dollars Function</summary>

```python
def parse_dollars(s):
    """Convert text monetary values to numeric format."""
    # If s is not a string, return NaN
    if type(s) != str:
        return np.nan
        
    # Handle millions format ($123.4 million)
    if re.match(r'\$\s*\d+\.?\d*\s*milli?on', s, flags=re.IGNORECASE):
        s = re.sub('\$|\s|[a-zA-Z]','', s)
        value = float(s) * 10**6
        return value
        
    # Handle billions format ($1.23 billion)
    elif re.match(r'\$\s*\d+\.?\d*\s*billi?on', s, flags=re.IGNORECASE):
        s = re.sub('\$|\s|[a-zA-Z]','', s)
        value = float(s) * 10**9
        return value
        
    # Handle standard format ($123,456,789)
    elif re.match(r'\$\s*\d{1,3}(?:[,\.]\d{3})+(?!\s[mb]illion)', s, flags=re.IGNORECASE):
        s = re.sub('\$|,','', s)
        value = float(s)
        return value
        
    # Return NaN for unrecognized formats
    else:
        return np.nan
```
</details>

<details>
<summary>Main ETL Function</summary>

```python
def extract_transform_load():
    """
    Main ETL function to process movie data from Wikipedia and Kaggle.
    
    Returns:
        Three dataframes: wiki_movies_df, kaggle_metadata, ratings
    """
    # Read data from source files
    kaggle_metadata = pd.read_csv('movies_metadata.csv', low_memory=False)
    ratings = pd.read_csv('ratings.csv')
    with open('wikipedia-movies.json', mode='r') as file:
        wiki_movies_raw = json.load(file)
        
    # Filter TV shows and clean movie data
    wiki_movies = [movie for movie in wiki_movies_raw if ('No. of episodes' not in movie)]
    clean_movies = [clean_movie(movie) for movie in wiki_movies]
    wiki_movies_df = pd.DataFrame(clean_movies)
    
    # Extract IMDb IDs and remove duplicates
    wiki_movies_df['imdb_id'] = wiki_movies_df['imdb_link'].str.extract(r'(tt\d{7})')
    wiki_movies_df.drop_duplicates(subset='imdb_id', inplace=True)
    
    # Keep columns with sufficient data
    wiki_columns_to_keep = [column for column in wiki_movies_df.columns 
                           if wiki_movies_df[column].isnull().sum() < len(wiki_movies_df) * 0.9]
    wiki_movies_df = wiki_movies_df[wiki_columns_to_keep]
    
    # Clean specific data columns
    # ... box office, budget, release date, and running time cleaning ...
    
    # Merge dataframes and handle conflicts
    # ... merging and conflict resolution code ...
    
    return wiki_movies_df, kaggle_metadata, ratings
```
</details>

<details>
<summary>Database Loading Code</summary>

```python
# Database connection and data loading
db_string = f"postgresql://postgres:{db_password}@127.0.0.1:5432/movies_data"
engine = create_engine(db_string)
movies_df.to_sql(name='movies', con=engine, if_exists='replace')

# Process large ratings file in chunks
rows_imported = 0
start_time = time.time()
for data in pd.read_csv(f'{file_dir}ratings.csv', chunksize=1000000):
    print(f'importing rows {rows_imported} to {rows_imported + len(data)}...', end='')
    data.to_sql(name='ratings', con=engine, if_exists='append')
    rows_imported += len(data)
    print(f'Done. {time.time() - start_time} total seconds elapsed')
```
</details>

## Results

### Pipeline Performance Metrics

| Metric | Value |
|--------|-------|
| Total Ratings Processed | 26,024,289 |
| Processing Time | ~655 seconds |
| Database Size | ~1-2GB |
| Final Movie Count | >5,000 |

### Research Question Outcomes

#### 1. Data Consolidation
Successfully created a unified dataset merging Wikipedia and Kaggle sources:
- Resolved conflicts between data sources
- Prioritized more reliable data when conflicts occurred
- Created consistent format for dates, monetary values, and text fields

#### 2. Data Cleaning Techniques
Implemented effective techniques for normalizing inconsistent data:
- Regular expressions for parsing varied formats
- Custom functions for data type conversion
- Lambda functions for column-specific transformations

#### 3. Automated Pipeline
Created a fully automated pipeline with:
- Reusable ETL function architecture
- Error handling for exceptional cases
- Memory-efficient processing of large datasets
- Database integration for persistent storage

### Alternative Approaches Considered
- **Data Lake Architecture**: Could have used S3/HDFS for raw storage
- **Incremental Loading**: Could have implemented change detection
- **NoSQL Storage**: Could have used document store for schema flexibility

## Business Impact

### Time Savings
- **Elimination of Manual Processing**: Automates what would take weeks of analyst time
- **Reduced Data Preparation**: Analysts can focus on insights rather than cleaning
- **Faster Time-to-Insight**: From raw data to insights in minutes rather than days

### Cost Reduction
- **Technical Debt Reduction**: Standardized data pipeline minimizes maintenance costs
- **Error Prevention**: Clean data prevents costly analytical mistakes
- **Open Source Stack**: Avoids expensive proprietary solutions

### Efficiency Improvements
- **Single Source of Truth**: Unified dataset for all movie analytics
- **Cross-Source Analytics**: Enables queries across previously siloed data
- **Scalable Framework**: Easily accommodate new data sources
- **Data Science Enablement**: Provides clean, reliable data for advanced analytics

### Quantifiable Benefits
| Benefit | Before | After |
|---------|--------|-------|
| Data Preparation Time | Days-Weeks | Minutes |
| Data Quality Issues | Frequent | Minimal |
| Analyst Productivity | Low (data cleaning) | High (analysis) |
| Data Integration | Manual | Automated |


## Future Enhancements

### Potential Improvements
- **Workflow Orchestration**: Implement Apache Airflow for scheduling and monitoring
- **Data Validation**: Add data quality checks and validation rules
- **Visualization Layer**: Create dashboard for monitoring pipeline performance
- **Incremental Processing**: Implement change detection for efficient updates
- **Containerization**: Dockerize the pipeline for easier deployment
- **Cloud Migration**: Adapt for cloud-based execution (AWS, GCP, Azure)

