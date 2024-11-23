import time
import random
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.chrome.service import Service
import os

# Giriş bilgileri (Çevresel değişkenlerden alınması önerilir)
INSTAGRAM_USERNAME = os.getenv('INSTAGRAM_USERNAME')
INSTAGRAM_PASSWORD = os.getenv('INSTAGRAM_PASSWORD')

if not INSTAGRAM_USERNAME or not INSTAGRAM_PASSWORD:
    raise Exception("Lütfen INSTAGRAM_USERNAME ve INSTAGRAM_PASSWORD çevresel değişkenlerini ayarlayın.")

# ChromeDriver ayarı
driver_path = "/usr/lib/chromium-browser/chromedriver"
service = Service(driver_path)
options = webdriver.ChromeOptions()
options.add_argument("--headless")  # Tarayıcıyı arka planda çalıştırır
options.add_argument("--disable-gpu")
options.add_argument("--no-sandbox")
driver = webdriver.Chrome(service=service, options=options)

try:
    # Instagram giriş
    driver.get("https://www.instagram.com/accounts/login/")
    time.sleep(3)

    # Kullanıcı adı ve şifre ile giriş
    driver.find_element(By.NAME, "username").send_keys(INSTAGRAM_USERNAME)
    driver.find_element(By.NAME, "password").send_keys(INSTAGRAM_PASSWORD)
    driver.find_element(By.XPATH, '//*[@id="loginForm"]/div/div[3]/button').click()

    time.sleep(5)  # Giriş sonrası bekleme

    # Belirli bir hashtag sayfasına git
    hashtag = "fitness"
    driver.get(f"https://www.instagram.com/explore/tags/{hashtag}/")
    time.sleep(5)

    # Gönderilere tıkla ve profilleri ziyaret et
    post_links = set()
    for _ in range(3):  # Sayfayı birkaç kez kaydır
        driver.find_element(By.TAG_NAME, "body").send_keys(Keys.END)
        time.sleep(random.uniform(2, 5))
        links = driver.find_elements(By.TAG_NAME, "a")
        for link in links:
            href = link.get_attribute("href")
            if "/p/" in href:
                post_links.add(href)

    print(f"{len(post_links)} gönderi bulundu.")

    # Profillerde e-posta topla
    emails = set()
    for post in post_links:
        driver.get(post)
        time.sleep(random.uniform(2, 5))

        try:
            profile_link = driver.find_element(By.XPATH, "//a[contains(@href, '/')]").get_attribute("href")
            driver.get(profile_link)
            time.sleep(random.uniform(2, 5))

            bio = driver.find_element(By.XPATH, "//div[contains(@class, '-vDIg')]").text
            print("Bio:", bio)

            if "@" in bio:
                bio_lines = bio.split("\n")
                for line in bio_lines:
                    if "@" in line and "." in line:
                        emails.add(line)
        except Exception as e:
            print(f"Hata: {e}")
            continue

    print(f"Toplanan e-postalar: {emails}")

finally:
    driver.quit()
