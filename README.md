# Carnegie Fabrics Window Products
I chose to analyze Carnegie Fabrics because they are a direct competitor for window products and also utilize internal manufacturing. See the results of my HTML data mining and a process explanation below.

<b>Recommendation:</b> create an web scraping pipeline to automate the collection of competitor web data monthly for more impactful benchmarking projects.

<ins><b>Product types:</b></ins>
* <b>Semi-transparent (39 products)</b>
  * Price range: $51 - $231 per yd.
  * Harmonic mean price: $124.81 per yd.
  * Common width: 118" (14 products)
  * Weight range: 10.9 oz - 17.7 oz per yd.
* <b>Sheer (34 products)</b>
  * Price range: $30 - $243 per yd.
  * Harmonic mean price: $102.54 per yd.
  * Common width: 118" (22 products)
  * Weight range: 1.61 oz - 12.95 oz per yd.
* <b>Opaque (33 products)</b>
  * Price range: $34 - $835 per yd.
  * Harmonic mean price: $89.84 per yd.
  * Common widths: 118" (10 products) and 72" (9 products)
  * Weight range: 10 oz - 12.5 oz per yd.

<ins><b>Pricing data scraped from Internet Archive:</b></ins>
* <b>2020:</b>
  * Price range: $23 - $450 per yd. 
* 2021 and 2022 data not available via archive
* <b>2023:</b>
  * Price range: $30 - $739 per yd.
* <b>2024:</b>
  * Price range: $34 - $1105 per yd.

<ins><b>Common warranty periods:</b></ins>
* 3 years (94 products)
* 10 years (8 products)

<ins><b>Common certifications:</b></ins>
* LEED - sustainability
* ISO 9001/14001 - product quality
* Mindful Materials - supply chain
* HHI - product quality

<ins><b>Material:</b></ins>
* Polyester and recycled polyester

<ins><b>Common cleaning codes:</b></ins>
* WS - Water / Solvent (95 products)
* WS & BC - Water / Solvent & Bleach Cleanable (12 products)

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

The dataset could have been fully cleaned with Python or Excel, but I prefer R for this type of one-off analysis. I would use Python for this stage if I was trying to build an automated web data collection pipeline.



8.   Launched an R notebook (view .rmd to see analysis)

9.   Visited https://web.archive.org/https://carnegiefabrics.com/windows to find screenshots of the Carnegie Fabrics site from an earlier date

10.   Repeated the same process for scraping prices from Carnegie Fabrics Internet Archive links

```
    import requests
from bs4 import BeautifulSoup
import csv

#avoid bot blockers
headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36'}

#request content
url='http://web.archive.org/web/20201208151437/https://carnegiefabrics.com/landing/privacy/'
response = requests.get(url, headers=headers)

if response.status_code == 200:
    page_content = response.content
else:
    print(f"Failed to retrieve webpage")

#parse the page
if page_content:
    soup = BeautifulSoup(page_content, 'html.parser')

product_names = soup.find_all('h4', class_='product-name')
prices = soup.find_all('span', class_='price')

#extract text
product_names_text = [product.get_text(strip=True) for product in product_names]
product_price_text = [price.get_text(strip=True) for price in prices]

#print extracted
for name, price in zip(product_names_text, product_price_text): print(f"Product Name: {name}, Price: {price}")


#create .csv
csv_file_path = '2020Dec8Carnegie5.csv'
with open(csv_file_path, mode='w', newline='', encoding='utf-8') as file:
    writer = csv.writer(file)
    writer.writerow(['Product Name', 'Price'])
    for name, price in zip(product_names_text, product_price_text): writer.writerow([name, price])
    print(f"Data has been exported to {csv_file_path}")
```
 
 12. Pulled the data into R for analysis (also in .rmd)

The limitation of relying on Internet Archive is that it crawls pages irregularly, so a lot of information is lost. Collecting this data regularly is best practice for executing meaningful benchmarking projects.


