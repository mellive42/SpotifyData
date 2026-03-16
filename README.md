# My Spotify Data

To complete this project I extracted my data in two ways:
    1. I requested my expanded listening history directly from Spotify, they provided a JSON file containing data from March 2025 to March 2026. 
    2. I extracted the data via Spotify's API and Google Colab using Python, exporting this into a CSV file.
I then merged both queries in Power BI to create the attached report. Please read on below for the process in Google Colab.

**NOTE:** I used a C:\DummyPath\Data.csv as parameters for the source path of the CSV and JSON file used in the report, to be able to view the report, you'll have to manually enter your local C:\ path (where you downloaded the files into) in Power Query, here:
<img width="890" height="237" alt="image" src="https://github.com/user-attachments/assets/1dda2804-81e5-43db-82c4-cd8669fed0a5" />



# Spotity data representing latest played tracks as sample data for a PBI dashboard.
For this project I extracted the 50 latest tracks I've played on my Spotify, and exported the data to a CSV file via Google Colab. This is the code I've used in my notebook, 
if you'd like to use it too just replace the missing fields with the data from your app in your Spotify Developers account. You'll need Client ID, Client Secret and URI:

    import spotipy
    from spotipy.oauth2 import SpotifyOAuth
    
    # Update credentials with values provided in the task description
    SPOTIPY_CLIENT_ID = 'ADD YOUR CLIENT ID'
    SPOTIPY_CLIENT_SECRET = 'ADD YOUR CLIENT SECRET'
    SPOTIPY_REDIRECT_URI = 'ADD YOUR URI'
    SPOTIPY_SCOPE = 'user-read-recently-played'
    
    # Initialize SpotifyOAuth object
    sp_oauth = SpotifyOAuth(
        client_id=SPOTIPY_CLIENT_ID,
        client_secret=SPOTIPY_CLIENT_SECRET,
        redirect_uri=SPOTIPY_REDIRECT_URI,
        scope=SPOTIPY_SCOPE,
        show_dialog=True # This ensures a fresh token if needed
      )

    print("Attempting to get Spotify access token...")
    print("If a browser window does not open automatically, please follow the URL provided below to authorize access:")
    
    access_token = None
    try:
        # Get an access token (this will open a browser window for user authentication if needed)
        access_token = sp_oauth.get_access_token(as_dict=False)
    except Exception as e:
        print(f"Automated authentication failed: {e}")
        print("Falling back to manual authorization.")
        auth_url = sp_oauth.get_authorize_url()
        print(f"Please navigate to this URL in your browser to authorize Spotify: {auth_url}")
        print("After authorization, you will be redirected to a blank page. Copy the complete URL from the browser's address bar and paste it below.")
        redirect_response = input("Paste the full redirect URL here: ")
        code = sp_oauth.parse_response_code(redirect_response)
        access_token = sp_oauth.get_access_token(code=code, as_dict=False)
    
    
    # Check if token was successfully obtained
    if access_token:
        print("Access token obtained successfully!")
        # You can now initialize the Spotify client with the access token
        sp = spotipy.Spotify(auth=access_token)
        print("Spotify client initialized.")

        # --- Extract Recently Played Tracks ---
        print("\nFetching recently played tracks...")
        results = sp.current_user_recently_played(limit=50) # Fetch up to 50 tracks
    
        # --- Process and Display Data ---
        if results and results['items']:
            print("Recently Played Tracks:")
            for i, item in enumerate(results['items']):
                track = item['track']
                print(f"{i+1}. Track: {track['name']}")
            print(f"   Artist(s): {', '.join([artist['name'] for artist in track['artists']])}")
            print(f"   Album: {track['album']['name']}")
            print(f"   Played At: {item['played_at']}")
            print("-" * 30)
    else:
        print("No recently played tracks found or unable to fetch them.")

    else:
        print("Failed to obtain access token. Please ensure your credentials, redirect URI (set to 'REPLACE WITH YOUR URI HERE' in your Spotify app), and the pasted URL are correct and try again.")
        sp = None # Set sp to None if authentication fails



Then to export this data to a CSV file to use in Power BI:

    import pandas as pd
    
    Prepare data
    if sp and results and results['items']:
        track_data = []
        for item in results['items']:
            track = item['track']
            track_name = track['name']
            artist_names = ', '.join([artist['name'] for artist in track['artists']])
            album_name = track['album']['name']
            played_at = item['played_at']
    
            track_data.append({
                'Track Name': track_name,
                'Artist(s)': artist_names,
                'Album': album_name,
                'Played At': played_at
            })
    
        Create Pandas dataframe
        df_recently_played = pd.DataFrame(track_data)
    
        Convert 'Played At' to datetime objects for better analysis in Power BI
        df_recently_played['Played At'] = pd.to_datetime(df_recently_played['Played At'])
    
        Export to CSV
        csv_filename = 'spotify_recently_played.csv'
        df_recently_played.to_csv(csv_filename, index=False, encoding='utf-8')
    
        print(f"Successfully exported {len(df_recently_played)} recently played tracks to {csv_filename}")
        print("First 5 rows of the exported data:")
        display(df_recently_played.head())
    else:
        print("No data to export. Please ensure authentication was successful and tracks were fetched.")


<img width="899" height="693" alt="Screenshot 2026-03-12 133225" src="https://github.com/user-attachments/assets/a25db0d0-64d5-4f21-bde1-bb62daab28ce" />
