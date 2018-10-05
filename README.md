
1001 Tracklists Scraper
=======================
A set of functions to scrape music tracklists from [1001 Tracklists](https://www.1001tracklists.com)

Import a bunch of stuff


```python
from bs4 import BeautifulSoup as bs
import requests 
import pandas as pd
import urllib3
import os
import spotipy
import spotipy.util as util
from pprint import pprint
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
```

Get track data from spotify and return it


```python
client_id='6389b29d73fc4806ba5e812e678854c1'
client_secret='0b4bcd832e694aedad408b5b4a93dd5c'
ccm=util.oauth2.SpotifyClientCredentials(client_id=client_id, client_secret=client_secret)
sp =spotipy.Spotify(client_credentials_manager=ccm)

def get_attrs(artist, track):
    try:
        res = sp.search(q='artist:'+artist+' track:'+track, type="track")
        track_res = res['tracks']['items'][0]
        track_id = track_res['uri']
        deets = sp.audio_features(track_id)
        return pd.Series(deets[0])
    except Exception as e:
        return False
    
#get_attrs(artist='Armin Van Buuren', track="shivers")
```

Get a tracklist from 1001 Tracklists and write it to a CSV with Spotify track info for all songs it can find


```python
def get_tracklist(url, folder='.'):
    os.makedirs(folder, exist_ok=True)
    
    !wget {url} -q
    fname = url.split('/')[-1]
    soup = bs(open(fname), "lxml")
    !rm {fname}
    
    tracklist = pd.DataFrame(columns=['Artist(s)', 'Title', 'Release'])
    set_name = soup.find(id="pageTitle").get_text()
    print(set_name)
    
    for div in soup.select('.trackValue'):
        try:
            text = div.get_text()
            artists_raw = text.split('-')[0].split(' & ')
            artists = " ".join(artists_raw) 
            artists = artists.replace('vs.', ' ').strip()
            artists = artists.replace('ft.', ' ')
            #print("Artists: "+artists)
            
            # title after '-' but before both label ([) and release (() and if a mashup, remove all but first song to make it easier to look up
            title = text.split('-')[1].split('[')[0].split('(')[0].split('vs.')[0]
            #print("title: "+title)
            
            # Releases in braces
            try:
                release = text.split('(')[1].split(')')[0]
            except:
                release = "unknown"
            #print("release: "+release)
            
            # Label in []
            try:
                label = text.split('[')[1].split(']')[0]
            except:
                label = "uknown"

            basic_details = pd.Series([artists, title, release], index=['Artist(s)', 'Title', 'Release'])
            
            #don't search spotify for unknown version, just search base name
            if release == 'unknown':
                release = ''
                
            spotify_details = get_attrs(artist=artists+' '+release, track=title)
            # Try removing version if not found
            if spotify_details is False:
                for a in artists.split(' '):
                    spotify_details = get_attrs(artist=a, track=title)
                    if spotify_details is not False:
                        break
            # Set details to none instead of false if not found, so track won't get excluded
            if spotify_details is False:
                spotify_details = None
                print(f'Not Found: {title} by {artists}')
            row = pd.concat([basic_details, spotify_details])
            tracklist = tracklist.append(row, ignore_index=True)
        except Exception as e:
            print(e)
            pass
    tracklist.to_csv(f'{folder}/{set_name}.csv')
    
#get_tracklist(url='https://www.1001tracklists.com/tracklist/1kjuxf4t/giuseppe-ottaviani-go-on-air-fsoe-stage-tomorrowland-belgium-2018-08-21.html', folder='otaviani_test')
```

Get a whole series of tracklists, and put them in a folder


```python
def get_series_tracklists(series_url, folder='.', recursecall=False):
    !wget {series_url} 
    fname=series_url.split('/')[-1]
    soup = bs(open(fname), "lxml")
    !rm {fname}
    main = soup.find(id='mainContentDiv')
    for mix_link in main.find_all('a', href=True):
        mix_href = mix_link['href']
        if mix_href.startswith('/tracklist/'):
            webpage = '/'.join(series_url.split('/')[0:3])
            print(mix_href)
            #get_tracklist(webpage+mix_link['href'], folder=folder)
        
    if recursecall == False:
        page_div = soup.find('ul', class_='pagination')
        other_pages = page_div.find_all('a', href=True)
        dont_follow = ['Prev', '1', 'Next']
        for page in other_pages:
            if page.get_text() not in dont_follow: #and mix_href != '/info/cookies.html':
                group = series_url[0:rfind('/')]
                print(group)
                get_series_tracklists(group+page['href'], folder=folder, recursecall=True)
                
get_series_tracklists(series_url='https://www.1001tracklists.com/groups/4u1f7w/ultra-music-festival-singapore-2018/index.html', folder='TomorrowLand2018')
```

    --2018-10-05 15:50:48--  https://www.1001tracklists.com/groups/4u1f7w/ultra-music-festival-singapore-2018/index.html
    Resolving www.1001tracklists.com (www.1001tracklists.com)... 158.69.5.7
    Connecting to www.1001tracklists.com (www.1001tracklists.com)|158.69.5.7|:443... connected.
    HTTP request sent, awaiting response... 403 Forbidden
    2018-10-05 15:50:48 ERROR 403: Forbidden.
    



    ---------------------------------------------------------------------------

    FileNotFoundError                         Traceback (most recent call last)

    <ipython-input-27-e775519e4114> in <module>()
         22                 get_series_tracklists(group+page['href'], folder=folder, recursecall=True)
         23 
    ---> 24 get_series_tracklists(series_url='https://www.1001tracklists.com/groups/4u1f7w/ultra-music-festival-singapore-2018/index.html', folder='TomorrowLand2018')
    

    <ipython-input-27-e775519e4114> in get_series_tracklists(series_url, folder, recursecall)
          2     get_ipython().system('wget {series_url} ')
          3     fname=series_url.split('/')[-1]
    ----> 4     soup = bs(open(fname), "lxml")
          5     get_ipython().system('rm {fname}')
          6     main = soup.find(id='mainContentDiv')


    FileNotFoundError: [Errno 2] No such file or directory: 'index.html'

