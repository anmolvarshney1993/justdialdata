# All you need to do is copy the below code and save it as a .py file.
# Run- python yourfilename.py (and see the magic)
import requests
from bs4 import BeautifulSoup
import json
import csv

jd_url = "http://www.justdial.com/Bangalore/Car-Hire-%3Cnear%3E-Shanthinagar"

jd_url = jd_url.split('http://www.justdial.com/')[-1].split('https://www.justdial.com/')[-1]
city, search, cat_id = '', '', ''
split_vals = jd_url.split('/')
if len(split_vals) == 3:
    city, search, cat_id = jd_url.split('/')
    cat_id = cat_id.split('-')[-1]
elif len(split_vals) == 2:
    city, search = jd_url.split('/')
search = search.replace('-', '+')


with open('data.csv', 'w') as f:
    #writer = csv.writer(f, delimiter=',', quoting=csv.QUOTE_ALL, lineterminator='\n')

    page = 1
    while True:
        print 'page', page
        resp = requests.get('http://www.justdial.com'+'/functions/ajxsearch.php?national_search=0&act=pagination&city={0}&search={1}&where=&catid={2}&psearch=&prid=&page={3}'.format(city, search, cat_id, page))
        markup = resp.json()['markup'].replace('\/', '/')
        soup = BeautifulSoup(markup, 'html.parser')


        for thing in soup.find_all('section'):
            csv_list = []
            if thing.get('class')==[u'jcar']:
                # Company name
                for a_tag in thing.find_all('a'):
                    if a_tag.get('onclick')=="_ct('clntnm', 'lspg');":
                        csv_list.append(a_tag.get('title'))

                # Address
                for span_tag in thing.find_all('span'):
                    if span_tag.get('class')==[u'mrehover', u'dn']:
                        csv_list.append(span_tag.get_text().strip())

                # Phone number
                for a_tag in thing.find_all('a'):
                    if a_tag.get('href').startswith('tel:'):
                        csv_list.append(a_tag.get('href').split(':')[-1])


                csv_list = ['"'+item+'"' for item in csv_list]
                writeline = ','.join(csv_list)+'\n'
                f.write(','.join(csv_list)+'\n')
        page+=1
