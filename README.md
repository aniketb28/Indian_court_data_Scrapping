
# Court Data Scraping Project

## Project Overview
This project is designed to extract data points from Indian court websites using Python, BeautifulSoup, and Selenium. The data includes petitioner, respondent, and case dates, and the project collects nearly 1 million records to ensure a robust and comprehensive dataset. The extracted data is cleaned and processed, achieving a high level of accuracy and facilitating detailed case trend analysis.

## Technologies Used
- **Python**: For scripting and data processing.
- **Selenium**: For automating web browsing and interacting with web elements.
- **BeautifulSoup**: For parsing HTML content and extracting data.
- **Pandas**: For data manipulation and analysis.

## Project Structure
- `main.py`: Main script to run the web scraping process.
- `requirements.txt`: List of required Python packages.
- `README.md`: Project documentation.

## Installation
1. **Clone the repository:**
    ```sh
    git clone <repository-url>
    cd <repository-directory>
    ```

2. **Install the required packages:**
    ```sh
    pip install -r requirements.txt
    ```

3. **Download and set up the ChromeDriver:**
    - Download ChromeDriver from [here](https://sites.google.com/a/chromium.org/chromedriver/downloads).
    - Ensure the `chromedriver` executable is in your PATH or place it in the project directory.

## Usage
1. **Update the `url` variable** in `main.py` with the URL of the court website:
    ```python
    url = 'https://main.sci.gov.in/case-status#'  # Replace with the actual URL
    ```

2. **Run the script:**
    ```sh
    python main.py
    ```

## Code Explanation
- **Import necessary libraries:**
    ```python
    from selenium import webdriver
    from selenium.webdriver.common.by import By
    from selenium.webdriver.support.ui import WebDriverWait
    from selenium.webdriver.support import expected_conditions as EC
    from selenium.common.exceptions import TimeoutException
    import time
    import pandas as pd
    from bs4 import BeautifulSoup
    ```

- **Define the CAPTCHA solving function:**
    ```python
    def solve_captcha_automatically(driver):
        html_content = driver.page_source
        soup = BeautifulSoup(html_content, 'html.parser')
        captcha_element = soup.find('p', {'id': 'cap'})
        captcha_value = captcha_element.find('font').text.strip()
        captcha_input = driver.find_element(By.ID, 'ansCaptcha')
        captcha_input.clear()
        captcha_input.send_keys(captcha_value)
    ```

- **Define the state and year selection functions:**
    ```python
    def select_state(driver, state_value):
        state_dropdown = WebDriverWait(driver, 10).until(EC.element_to_be_clickable((By.ID, 'ddl_st_agncy')))
        state_dropdown.click()
        state_option_xpath = f'//select[@id="ddl_st_agncy"]/option[@value="{state_value}"]'
        state_option = driver.find_element(By.XPATH, state_option_xpath)
        state_option.click()

    def select_year(driver, year_value):
        year_dropdown = WebDriverWait(driver, 10).until(EC.element_to_be_clickable((By.ID, 'ddl_ref_caseyr')))
        year_dropdown.click()
        year_option_xpath = f'//select[@id="ddl_ref_caseyr"]/option[@value="{year_value}"]'
        year_option = driver.find_element(By.XPATH, year_option_xpath)
        year_option.click()
    ```

- **Define the function to click the right button:**
    ```python
    def click_right_button(driver):
        try:
            right_button = WebDriverWait(driver, 10).until(EC.element_to_be_clickable((By.ID, 'btn_right_cs')))
            right_button.click()
            time.sleep(10)
            return True
        except TimeoutException:
            return False
    ```

- **Define the function to get table data:**
    ```python
    def get_table_data(driver, year):
        try:
            table = WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.XPATH, '//div[@id="dv_include_cs"]/table')))
            resultant_page_html = driver.page_source
            soup = BeautifulSoup(resultant_page_html, 'html.parser')
            table = soup.find('div', {'id': 'dv_include_cs'}).find('table')

            if table:
                headers = table.find_all('th')
                header_texts = [header.get_text(strip=True) for header in headers]
                data = []
                for row in table.find_all('tr')[1:]:
                    row_data = [td.get_text(strip=True) for td in row.find_all('td')]
                    data.append(row_data)
                df = pd.DataFrame(data, columns=header_texts)
                df['Year'] = year
                return df
            else:
                return None
        except TimeoutException:
            return None
    ```

- **Main script:**
    ```python
    url = 'https://main.sci.gov.in/case-status#'
    driver = webdriver.Chrome()
    driver.get(url)
    court_tribunal_tab = driver.find_element(By.XPATH, '//li[@data-link="tab5"]/a[@class="z-link"]')
    court_tribunal_tab.click()

    all_data = pd.DataFrame()
    for year in range(1950, 2024):
        solve_captcha_automatically(driver)
        select_state(driver, '21945')
        select_year(driver, str(year))
        button = WebDriverWait(driver, 10).until(EC.element_to_be_clickable((By.ID, 'getLowerCourtData1')))
        button.click()
        time.sleep(10)
        current_page_data = get_table_data(driver, year)
        if current_page_data is not None:
            all_data = pd.concat([all_data, current_page_data], ignore_index=True)
        while click_right_button(driver):
            current_page_data = get_table_data(driver, year)
            if current_page_data is not None:
                all_data = pd.concat([all_data, current_page_data], ignore_index=True)
            else:
                break
    ```

## Notes
- Adjust the sleep times and element selectors according to the website's structure and behavior.
- Ensure compliance with the website's terms of service and legal requirements when scraping data.
.
