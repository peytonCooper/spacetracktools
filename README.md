# spacetracktools
Repository for python tools made for interfacing with Space-Track.org
📡 TLE Downloader and Parser

This Python script logs in to Space-Track.org, retrieves TLE (Two-Line Element) data for a specified satellite (by NORAD Catalog ID), and saves both the raw and parsed TLE data to local files.

🚀 Features
Logs in to Space-Track using your credentials.
Downloads TLE data for a given satellite and time range.
Parses the TLE data into a structured format.
Saves:
Raw TLE data to tle_raw.txt
Parsed TLE data to tle_parsed.csv (with ISO-8601 formatted epoch dates)

🧰 Requirements
Python 3.6+
Required packages (install via pip):
pip install requests

⚙️ Configuration
Edit the following variables in the script before running:
USERNAME = 'your_email'       # Your Space-Track.org username
PASSWORD = 'your_password'    # Your Space-Track.org password

NORAD_CAT_ID = 'the satellite norad id you want to grab'
START_DATE = '2024-01-01T00:00:00Z'
END_DATE = '2024-01-30T00:00:00Z'
uncomment the data pass statement for the dates to work, otherwise it will return default time values


💡 Note: Ensure you have a valid account on Space-Track.org
 and have accepted their user agreement.

📂 Output
tle_raw.txt: Raw two-line element set as returned from the Space-Track API.
tle_parsed.csv: Parsed data with each TLE element in separate columns, including ISO 8601 date.
