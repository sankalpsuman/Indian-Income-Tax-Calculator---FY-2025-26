from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import time

# Path to your ChromeDriver
CHROMEDRIVER_PATH = "/path/to/chromedriver"

# Local or hosted HTML file path
PAGE_URL = "file:///path/to/TaxCalculatorLoginPage.html"

# Setup driver
options = webdriver.ChromeOptions()
options.add_argument("--start-maximized")
driver = webdriver.Chrome(service=Service(CHROMEDRIVER_PATH), options=options)

try:
    driver.get(PAGE_URL)

    wait = WebDriverWait(driver, 10)

    # Fill in salary
    salary_input = wait.until(EC.presence_of_element_located((By.ID, "salary")))
    salary_input.clear()
    salary_input.send_keys("1600000")

    # Fill other allowances/deductions if available
    try:
        hra_input = driver.find_element(By.ID, "hra")
        hra_input.clear()
        hra_input.send_keys("200000")
    except:
        print("HRA field not found, skipping.")

    try:
        deduction_input = driver.find_element(By.ID, "deductions")
        deduction_input.clear()
        deduction_input.send_keys("150000")
    except:
        print("Deductions field not found, skipping.")

    # Click calculate
    calc_btn = driver.find_element(By.ID, "calculateBtn")
    calc_btn.click()

    # Wait for results to show
    result_old = wait.until(EC.presence_of_element_located((By.ID, "oldRegimeTax")))
    result_new = wait.until(EC.presence_of_element_located((By.ID, "newRegimeTax")))

    print("Old Regime Tax:", result_old.text)
    print("New Regime Tax:", result_new.text)

    # Simple assertion
    assert result_old.text != "" and result_new.text != "", "Tax calculation failed!"

    print("✅ Test Passed: Tax calculation values displayed.")

finally:
    time.sleep(3)
    driver.quit()
