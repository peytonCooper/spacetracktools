import requests
import csv
from urllib.parse import quote
from datetime import datetime, timedelta

# Space-Track credentials (use with caution)
USERNAME = 'your_email'
PASSWORD = 'your_password

# Parameters
NORAD_CAT_ID = 25544  # Example: ISS
START_DATE = '2024-01-01T00:00:00Z'
END_DATE = '2024-01-30T00:00:00Z'

LOGIN_URL = 'https://www.space-track.org/ajaxauth/login'
TLE_URL = (
    f'https://www.space-track.org/basicspacedata/query/class/tle/'
    f'NORAD_CAT_ID/{NORAD_CAT_ID}/'
    #f'EPOCH/{quote(">" + START_DATE)},{quote("<" + END_DATE)}/'  
    f'orderby/EPOCH%20asc/format/tle'
)

# Output files
OUTPUT_CSV = 'tle_parsed.csv'
OUTPUT_RAW = 'tle_raw.txt'


def tle_epoch_to_iso(epoch_str):
    """
    Convert TLE epoch from format NNNNN.NNNNNNNN (YYDDD.FFFFFF)
    to ISO 8601 datetime string with milliseconds and 'Z' suffix.
    """
    # Parse year (2 digits), julian day (3 digits), fractional day
    year_part = int(epoch_str[0:2])
    julian_day = int(epoch_str[2:5])
    fractional_day = float('0' + epoch_str[5:]) if len(epoch_str) > 5 else 0.0

    year_full = 2000 + year_part  # assuming 21st century
    date_start = datetime(year_full, 1, 1)

    # Add days and fractional days to Jan 1
    full_date = date_start + timedelta(days=julian_day - 1 + fractional_day)

    # Format as ISO 8601 with milliseconds + 'Z' (UTC)
    iso_str = full_date.strftime('%Y-%m-%dT%H:%M:%S.%f')[:-3] + 'Z'
    return iso_str


def parse_tle_lines(line1, line2):
    return {
        'Satellite Name': line1[2:7].strip(),  # fallback name

        # Line 1
        'Line Number 1': line1[0],
        'Satellite Catalog Number': line1[2:7].strip(),
        'Elset Classification': line1[7],
        'International Designator': line1[9:17].strip(),
        'Element Set Epoch (UTC)': line1[18:32].strip(),
        '1st Derivative of Mean Motion': line1[33:43].strip(),
        '2nd Derivative of Mean Motion': line1[44:52].strip(),
        'B* Drag Term': line1[53:61].strip(),
        'Element Set Type': line1[62],
        'Element Number': line1[64:68].strip(),
        'Checksum 1': line1[68],

        # Line 2
        'Line Number 2': line2[0],
        'Satellite Catalog Number 2': line2[2:7].strip(),
        'Inclination (deg)': line2[8:16].strip(),
        'RA of Ascending Node (deg)': line2[17:25].strip(),
        'Eccentricity': line2[26:33].strip(),
        'Argument of Perigee (deg)': line2[34:42].strip(),
        'Mean Anomaly (deg)': line2[43:51].strip(),
        'Mean Motion (rev/day)': line2[52:63].strip(),
        'Revolution Number at Epoch': line2[63:68].strip(),
        'Checksum 2': line2[68]
    }


def save_parsed_tles_to_csv(tle_text, filename):
    lines = [line.strip() for line in tle_text.strip().split('\n') if line.strip()]

    if len(lines) % 2 != 0:
        print("⚠️ Warning: TLE data does not contain a multiple of 2 lines!")

    # Add ISO Date column as first column
    fieldnames = [
        'ISO Date',  # New column 1
        'Satellite Name',
        'Line Number 1',
        'Satellite Catalog Number',
        'Elset Classification',
        'International Designator',
        'Element Set Epoch (UTC)',
        '1st Derivative of Mean Motion',
        '2nd Derivative of Mean Motion',
        'B* Drag Term',
        'Element Set Type',
        'Element Number',
        'Checksum 1',
        'Line Number 2',
        'Satellite Catalog Number 2',
        'Inclination (deg)',
        'RA of Ascending Node (deg)',
        'Eccentricity',
        'Argument of Perigee (deg)',
        'Mean Anomaly (deg)',
        'Mean Motion (rev/day)',
        'Revolution Number at Epoch',
        'Checksum 2'
    ]

    with open(filename, mode='w', newline='') as csvfile:
        writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
        writer.writeheader()

        for i in range(0, len(lines), 2):
            try:
                line1 = lines[i]
                line2 = lines[i + 1]

                if not line1.startswith("1 ") or not line2.startswith("2 "):
                    print(f"⚠️ Skipping invalid TLE pair at lines {i}-{i+1}")
                    continue

                parsed = parse_tle_lines(line1, line2)
                # Add ISO Date converted from Element Set Epoch (UTC)
                parsed['ISO Date'] = tle_epoch_to_iso(parsed['Element Set Epoch (UTC)'])

                writer.writerow(parsed)

            except IndexError:
                print(f"Incomplete TLE at lines {i}-{i+1}, skipping...")
                continue


def main():
    with requests.Session() as session:
        login_payload = {
            'identity': USERNAME,
            'password': PASSWORD
        }
        login_resp = session.post(LOGIN_URL, data=login_payload)
        if login_resp.status_code != 200 or 'Login failed' in login_resp.text:
            print("Login failed: Check your username/password.")
            return

        print("✅ Login successful!")

        print("📡 Fetching TLE data from:", TLE_URL)
        tle_resp = session.get(TLE_URL)
        if tle_resp.status_code == 200:
            raw_data = tle_resp.text
            print("📄 Saving raw TLE data...")
            with open(OUTPUT_RAW, 'w') as raw_file:
                raw_file.write(raw_data)
            print(f"✅ Raw TLE data saved to '{OUTPUT_RAW}'")

            print("🧾 Parsing TLE data and saving to CSV...")
            save_parsed_tles_to_csv(raw_data, OUTPUT_CSV)
            print(f"✅ Parsed TLE data saved to '{OUTPUT_CSV}'")
        else:
            print(f"❌ Failed to fetch TLEs, status code: {tle_resp.status_code}")
            print(tle_resp.text)


if __name__ == '__main__':
    main()
