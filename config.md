## Getting Started

### Prerequisites
- Python 3.x
- PostgreSQL instance
- Required Python packages:
  ```
  pandas
  numpy
  psycopg2
  sqlalchemy
  jupyter
  ```

### Installation
1. Clone this repository
   ```bash
   git clone https://github.com/yourusername/movie-etl-pipeline.git
   cd movie-etl-pipeline
   ```

2. Install dependencies
   ```bash
   pip install -r requirements.txt
   ```

3. Configure database connection
   Create a `config.py` file with:
   ```python
   db_password = 'your_postgres_password'
   ```

4. Run the notebooks in sequence:
   - `ETL_function_test_Del_1.ipynb` 
   - `ETL_clean_wiki_movies_Del_2.ipynb`
   - `ETL_clean_kaggle_data_Del_3.ipynb`
   - `ETL_create_database_Del_4.ipynb`
