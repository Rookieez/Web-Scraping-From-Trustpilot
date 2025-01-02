# Web-Scraping-From-Trustpilot
Scrape reviews from Trustpilot


import requests
from bs4 import BeautifulSoup
import csv
import pandas as pd


url = 'https://ca.trustpilot.com/review/www.canadianappliance.ca'


response = requests.get(url)
soup = BeautifulSoup(response.text, 'html.parser')


item = soup.find(
    name='p',
    attrs={'class': 'typography_body-l__KUYFJ typography_appearance-default__AAY17'}
)
N = int(item.contents[0].replace(',', ''))
print(N)


page = 'https://ca.trustpilot.com' + \
soup.find_all(name="a", string='Next page')[0]['href']
print(page)


def scrape_reviews(url):
    """
    Scrapes customer reviews from Canadian Appliance Source URL.
    
    Use HTTP status code 200 to check if the request was successful or not.

    Find all review elements on the URL website.

    Scrape company name, published date, rating value and review texts from the website.

    If customer only give rating no text then use No Review Text be filled.

    Append review details to the list as a dictionary.

    If the page cannot be retrieved, print an error and return an empty list
    """
    response = requests.get(url)

    if response.status_code == 200:
        soup = BeautifulSoup(response.text, 'html.parser')

        reviews = soup.find_all('div', class_='styles_reviewCardInner__EwDq2')

        review_list = []
        for review in reviews:
            company_name = 'Canadian Appliance Source'

            date_published = review.find('time')['datetime']

            rating_value = review.find(
                'div', class_='styles_reviewHeader__iU9Px'
            )['data-service-review-rating']

            review_paragraph = review.find(
                'p',
                class_='typography_body-l__KUYFJ typography_appearance-default__AAY17 typography_color-black__5LYEn'
            )
            review_body = review_paragraph.text if review_paragraph else "No review text"

            review_list.append({
                'companyName': company_name,
                'datePublished': date_published,
                'ratingValue': rating_value,
                'reviewBody': review_body
            })

        return review_list
    else:
        print("Failed to retrieve the page")
        return []


def scrape_all_reviews(base_url, pages=200):
    """
    Scrape pages of reviews from the base URL.

    Set page limit to 200 because there are so many reviews

    Exit if there are no more pages
    """
    all_reviews = []
    current_url = base_url

    for page in range(pages):
        print(f"Scraping page {page + 1}...")
        all_reviews.extend(scrape_reviews(current_url))

        soup = BeautifulSoup(requests.get(current_url).text, 'html.parser')
        next_page = soup.find(name="a", string="Next page")

        if next_page:
            current_url = 'https://ca.trustpilot.com' + next_page['href']
        else:
            break 

    return all_reviews


def save_reviews_to_csv(reviews, file_name='Canadian_Appliance_Source_Reviews.csv'):
    """Scrape all 200 pages and save the list of reviews to a CSV file."""
    headers = ['companyName', 'datePublished', 'ratingValue', 'reviewBody']

    with open(file_name, mode='w', newline='', encoding='utf-8') as file:
        writer = csv.DictWriter(file, fieldnames=headers)
        writer.writeheader()
        writer.writerows(reviews)

base_url = 'https://ca.trustpilot.com/review/www.canadianappliance.ca'

all_reviews = scrape_all_reviews(base_url, pages=200)

save_reviews_to_csv(all_reviews)
