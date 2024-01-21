# first-py
from app import app

class Repository:

    def __init__(self, DBUSER, DBPASSWORD, HOST, DATABASE):
        try:
            self.engine = create_engine(f'postgresql+psycopg2://{DBUSER}:{DBPASSWORD}@{HOST}/{DATABASE}')
            self.conn = self.engine.connect()
        except Exception as e:
            app.logger.warning('Could not connect to the database on client.py file.')
            app.logger.warning(f'Verify your credentials for {DBUSER}.')
            app.logger.warning(e)

    def select_all_tracks(self, listed_artists, popular_artists, not_explicit, from_year, to_year):
        query = f'''
@@ -75,12 +84,3 @@ def select_top_artists(self, top):
        limit {top};
        ''')
        return [{"popularity": x["popularity"], "name": x["name"], "img": x["img"], "genres": x["genres"]} for x in result]

    def __init__(self, DBUSER, DBPASSWORD, HOST, DATABASE):
        try:
            self.engine = create_engine(f'postgresql+psycopg2://{DBUSER}:{DBPASSWORD}@{HOST}/{DATABASE}')
            self.conn = self.engine.connect()
        except Exception as e:
            app.logger.warning('Could not connect to the database on client.py file.')
            app.logger.warning(f'Verify your credentials for {DBUSER}.')
            app.logger.warning(e)
  38 changes: 23 additions & 15 deletions38  
app/services.py
@@ -7,29 +7,37 @@
from spotipy.oauth2 import SpotifyClientCredentials
import spotipy

repository = Repository(config.DBUSER, config.DBPASSWORD, config.HOST, config.DATABASE)
sp = spotipy.Spotify(auth_manager=SpotifyClientCredentials(client_id=config.SPOTIFY_CLIENT_ID, client_secret=config.SPOTIFY_CLIENT_SECRET))
def __init__(self):
    self.repository = Repository(config.DBUSER, config.DBPASSWORD, config.HOST, config.DATABASE)
    self.sp = spotipy.Spotify(auth_manager=SpotifyClientCredentials(client_id=config.SPOTIFY_CLIENT_ID, client_secret=config.SPOTIFY_CLIENT_SECRET))

def search(search_string):
def search(self, search_string):
    results = []
    if search_string:
        results = sp.search(q=search_string,type="track")['tracks']['items']
        results = self.sp.search(q=search_string,type="track")['tracks']['items']
    return jsonify(results)

def get_artist_features(artist_id):
    name, features_values = repository.select_artist_features(artist_id)
def get_artist_features(self, artist_id):
    name, features_values = self.repository.select_artist_features(artist_id)
    features_keys = ['acousticness', 'danceability', 'energy', 'instrumentalness', 'liveness', 'speechiness', 'valence']
    data = {features_keys[i]: features_values[i] for i in range(len(features_keys))}
    return {"name": name, "chart": data}
    return jsonify({"name": name, "chart": data})

def get_artists_top(top):
    return jsonify(repository.select_top_artists(top))
def get_artists_top(self, top):
    return jsonify(self.repository.select_top_artists(top))

def get_counts():
    return jsonify(repository.select_counts())
def get_counts(self):
    return jsonify(self.repository.select_counts())

def get_recommendations(from_year, to_year, listed_artists, popular_artists, exclude_explicit, track_ids):
    names, artists, albums, years, imgs, uris, matches = get_recommendation(from_year, to_year, listed_artists, popular_artists, exclude_explicit, sp.audio_features(track_ids))
def get_dataset_stats(self):
    albums_per_year=self.repository.select_albums_by_year()
    artists_per_genre=self.repository.select_artists_by_genres()
    artists_per_popularity=self.repository.select_artists_by_popularity()
    artist_list=self.repository.select_all_artists()
    return albums_per_year, artists_per_genre, artists_per_popularity, artist_list

def get_recommendations(self, from_year, to_year, listed_artists, popular_artists, exclude_explicit, track_ids):
    names, artists, albums, years, imgs, uris, matches = get_recommendation(from_year, to_year, listed_artists, popular_artists, exclude_explicit, self.sp.audio_features(track_ids))
    return jsonify ([{"name": names[i], "artist": artists[i], "album": albums[i], "year": int(years[i]), "img": imgs[i], "uri": "https://open.spotify.com/track/" + uris[i][14:], "match": matches[i]} for i in range(len(names))])


@@ -39,13 +47,13 @@ def create_similarity(song, dataframe):
    similarity_index = pd.DataFrame(similarities, columns=dataframe.index)
    return similarity_index

def get_recommendation(from_year, to_year, listed_artists, popular_artists, not_explicit, features_list):
def get_recommendation(self, from_year, to_year, listed_artists, popular_artists, not_explicit, features_list):
    keys_to_remove = ["duration_ms","key","mode","time_signature","loudness","id","uri","track_href","analysis_url","type"]
    for features in features_list:
        for key in keys_to_remove:
            features.pop(key)

    tracks_list = pd.DataFrame.from_dict(repository.select_all_tracks(listed_artists, popular_artists, not_explicit, from_year, to_year),
    tracks_list = pd.DataFrame.from_dict(self.repository.select_all_tracks(listed_artists, popular_artists, not_explicit, from_year, to_year),
    orient="columns")
    tracks_list.columns=["name", "artist", "album", "year", "uri", "img", "acousticness", "danceability", "energy", "instrumentalness", "liveness", "speechiness", "valence", "tempo"]
    scaler = StandardScaler()
