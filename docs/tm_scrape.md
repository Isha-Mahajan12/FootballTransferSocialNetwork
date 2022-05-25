```python
import requests
from bs4 import BeautifulSoup as bs
import csv
import re
```


```python
def data_to_csv(data):
    keys = data[0].keys()
    with open('data_new_markets_21_full_test_3.csv', 'w', newline='')  as output_file:
        dict_writer = csv.DictWriter(output_file, keys)
        dict_writer.writeheader()
        dict_writer.writerows(data)
```


```python
def format_text(text):
    regex = re.compile(r'[\n\r\t]')
    text = regex.sub('', text)

    return " ".join(text.split())
```


```python
def format_currency(value):
    value = value.replace('€', '')
    value = value.replace('Loan fee:', '')
    
    if value[-1] == 'm':
        value = value.replace('m', '')
        return int(float(value)) * 1000000

    if value[-1] == '.':
        value = value.replace('.', '')
        if value[-2:] == 'Th':
            value = value.replace('Th', '')
            return int(value) * 1000
    
    return value
```


```python
def get_data(pages):
    players_list = []
    for page in range(1, pages+1):
        headers = {"User-Agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.97 Safari/537.36"}

        #url  = f'https://www.transfermarkt.com/transfers/saisontransfers/statistik/top/plus/1/galerie/0?saison_id=2021&transferfenster=alle&land_id=&ausrichtung=&spielerposition_id=&altersklasse=&lesihe='

        
        
        url = f'https://www.transfermarkt.com/transfers/saisontransfers/statistik/top/saison_id/2021/transferfenster/alle/land_id//ausrichtung//spielerposition_id//altersklasse//lesihe//plus/1/galerie/0/page/4'
        print(url)

        html = requests.get(url, headers=headers)
        soup = bs(html.content)

        soup = soup.select('.responsive-table > .grid-view > .items > tbody')[0]

        try:
            for cells in soup.find_all(True, {"class": re.compile("^(even|odd)$")}):
                fee = cells.find_all('td')[16].text
                position = cells.find_all('td')[4].text
                age = cells.find_all('td')[5].text
                market_value = cells.find_all('td')[6].text
                country_from = cells.find_all('td')[11].img['title']
                league_from = cells.find_all('td')[11].a.text if cells.find_all('td')[11].a != None else 'Without League'
                club_from = cells.find_all('td')[9].img['alt']
                country_to = cells.find_all('td')[15].img['alt']
                league_to = cells.find_all('td')[15].a.text if cells.find_all('td')[15].a != None else 'Without League'
                club_to = cells.find_all('td')[13].img['alt']

                player = {
                    'name': cells.find_all('td')[1].select('td > img')[0]['title'],
                    'position': position,
                    'age': age,
                    'market_value': format_currency(market_value),
                    'country_from': country_from,
                    'league_from': format_text(league_from),
                    'club_from': club_from,
                    'country_to': country_to,
                    'league_to': format_text(league_to),
                    'club_to': club_to,
                    'fee': format_currency(fee),
                }

                players_list.append(player)
        except IndexError:
            pass

    return players_list
```


```python
data = get_data(1)
data_to_csv(data)
```

    https://www.transfermarkt.com/transfers/saisontransfers/statistik/top/saison_id/2021/transferfenster/alle/land_id//ausrichtung//spielerposition_id//altersklasse//lesihe//plus/1/galerie/0/page/3



```python

```
