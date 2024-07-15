# Carnegie Fabrics Window Products
I chose to analyze Carnegie Fabrics because they are a direct competitor for window products and also utilize internal manufacturing. See the results of my HTML data mining and a process explanation below.

<ins><b>Product types:</b></ins>
* <b>Semi-transparent (39 products)</b>
  * Price range: $51 - $231 per yd.
  * Harmonic mean price: $124.81 per yd.
* <b>Sheer (34 products)</b>
  * Price range: $30 - $243 per yd.
  * Harmonic mean price: $102.54 per yd.
* <b>Opaque (33 products)</b>
  * Price range: $34 - $835 per yd.
  * Harmonic mean price: $89.84 per yd.

<ins><b>Pricing data scraped from Internet Archive:</b></ins>
* 2020:
* 2021 and 2022 data not available via archive
* 2023:
* 2024:

<ins><b>Common warranty periods:</b></ins>
* 3 years (X products)
* 10 years (X products)

<ins><b>Common certifications:</b></ins>
* LEED - sustainability
* ISO 9001/14001 - product quality
* Mindful Materials - supply chain

<ins><b>Common cleaning codes:</b></ins>
* WS - Water / Solvent (X products)

# Data Mining Process
1. Visited https://carnegiefabrics.com/windows and inspected the HTML
2. Imported necessary Python libraries for web scraping
   
```
import requests
from bs4 import BeautifulSoup
import csv
```
<br>


3.   Established a user-agent header to avoid bot blockers and defined the url to maximize information exposure

```
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36'
}

url = 'https://carnegiefabrics.com/windows?product_list_limit=300'
```
<br>


4.   Sent a GET request and parsed the HTML

```
response = requests.get(url, headers=headers)
response.raise_for_status()

soup = BeautifulSoup(response.content, 'html.parser')
```
<br>

5.   Targeted `<span class="product-item-link>` and created a list to store the extracted data

```
product_spans = soup.find_all('span', class_='product-item-link')

data = []
unique_titles = set()
```
<br>


6.   Looped through every product `<span>`, link, and description title and text. Extracted the relevant information and combined titles and text (ensuring uniqueness) before filling the list.

```
for span in product_spans:
    product_url = span.get('href')
    if product_url:
        product_response = requests.get(product_url, headers=headers)
        product_response.raise_for_status()
        product_soup = BeautifulSoup(product_response.content, 'html.parser')
        
        product_title_elements = product_soup.find_all('span', class_='product-description-title')
        product_text_elements = product_soup.find_all('span', class_='product-description-text')
        
        product_titles = [title.get_text(strip=True) for title in product_title_elements]
        product_texts = [text.get_text(strip=True) for text in product_text_elements]
        
        product_description = dict(zip(product_titles, product_texts))
        
        unique_titles.update(product_titles)
        
        product_name = span.get_text(strip=True)
        data.append((product_name, product_description))
```

<br>

7.   Wrote the data to a .csv file for efficient processing and organized the columns and rows to be meaningful

```
csv_file_path = 'carnegie_fabrics_products.csv'
with open(csv_file_path, 'w', newline='', encoding='utf-8') as file:
    writer = csv.writer(file)
    
    header = ['Product Name'] + sorted(unique_titles)
    writer.writerow(header)
    
    for product_name, product_description in data:
        row = [product_name]
        for title in sorted(unique_titles):
            row.append(product_description.get(title, ''))
        writer.writerow(row)
```

Because of structural inconsistencies in the webpages, some easily identifiable errors were present in the dataset and I corrected them before cleaning with R.

The dataset could have been cleaned with Python or Excel, but I prefer R for this type of analysis.



8.   Launched an R notebook (view .rmd to see analysis)

9.   Visited https://web.archive.org/https://carnegiefabrics.com/windows to find screenshots of the Carnegie Fabrics site from an earlier date
