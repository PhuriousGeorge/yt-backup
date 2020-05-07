# yt-backup
Youtube Backup made easy

## What is this?
This is a backup utility for youtube videos. Since youtube-dl works very well, yt-backup will use youtube-dl as downloader.
Since youtube started aggressively blocking extensive youtube-dl crawls with HTTP 429 error, I have started fetching all metadata about channels, playlists and videos via google's youtube API and using youtube-dl only for downloading videos itself.
All metadata will be stored in a database. To be flexible at this point, I have chosen sqlalchemy as ORM. So you can use any database which is supported by sqlalchemy.
Databases are fine for storing stuff and getting it again, but not for visualizing things, I have created some grafana dashboards for visualizing the stats of the tool.
Additionally, I have added support for automatic proxy restarts, in case you get a 429 error on your current IP. This option assumes, that after each comman execution, the proxy will have a new IP adress.

## requirements
- python3
- python modules: sqlalchemy ConfigParser google-api-python-client google_auth_oauthlib mysql dateutil
- Any sqlalchemy supported database
- rclone
- youtube-dl
- youtube API key
- credentials.json file for your API key
- working grafana installation

## Installation
1. Clone this repo
2. Create user in your DBMS with write permissions for a schema
3. If not available, configure a rclone remote. If remote points to cloud storage, I strongly recommend to add a crypt remote
4. Edit config.json to match your system paths, database and rclone remote
4.1. git commit your config.json, so it will not be overwritten by new ones in the repo every time you pull
5. Put your client secret json from google into project directory and name it "client_secret.json"
6. Add your database as datasource in grafana. Best name it yt-backup.

## config options
### database
- connection_info: Connection information to your already installed database

### base
- download_dir: Directory where youtube-dl should put your videos before uploading it via rclone
- download_lockfile: Where to put download lockfile. This prevents, that multiple download jobs will run if script is planned via job
- proxy_restart_command: If you have a proxy which can change it's IP adress, add it's restart command here.

### rclone
- binary_path: Where to find your clone binary
- config_path: Where to find your rclone config file
- move_or_copy: Should rclone move or copy videos after download. Strongly recommend move.
- upload_base_path: Where to upload the videos in your rclone remote
- upload_target: The rclone remote to which the videos should be pushed

### youtube-dl
- binary_path: Where to find your youtube-dl binary
- download-archive: Where to find your youtube-dl download archive. Could be an existing file.
- video-format: This will be put to youtube-dl as --format option. Defaults to the best video possible
- min_sleep_interval: How many seconds to sleep between two video downloads minimum
- max_sleep_interval: How many seconds to sleep between two video downloads maximum
- proxy: Which proxy and port youtube-dl should use to download videos. Leave empty for No proxy usage

## Usage
### Get help output
- python3 yt-backup.py --help

### Add a channel
- python3 yt-backup.py add_channel --channel_id <youtube-channel-id>
  
OR

- python3 yt-backup.py add_channel --username <youtube-user-id>

### Get all playlists for all channels
- python3 yt-backup.py get_playlists

### Get all videos from all playlists
- python3 yt-backup.py get_video_infos

### Download all videos which are not downloaded currently
- python3 yt-backup.py download_videos

### Download all videos from one specific playlist ID
- python3 yt-backup.py download_videos --playlist_id

All videos which are in database, but not in the channel's playlist anymore, will be marked as offline.
If you want to know, if they are completly gone or just private now, you should run python3 yt-backup.py verify_offline_videos

### Generate Statistics
- python3 yt-backup.py generate_statistics --statistics <archive_size,videos_monitored,videos_downloaded>

### Get playlists, get video infos, download new videos, check all offline videos against youtube API and generate statistics in one command
- python3 yt-backup.py run

### Enable or disable the download for videos of a channel
- python3 yt-backup.py toggle_channel_download --username <channel_name> --disable
- python3 yt-backup.py toggle_channel_download --username <channel_name> --enable

### Verifiy all marked offline videos if they are really offline or only private
- python3 yt-backup.py verify_offline_videos

All videos which are marked as offline in database will be checked in packages of 50 videos against the youtube API. Each video which is not returned in answer, will be marked as offline. If a video is part of the answer, it will be marked as online again or as unlisted if the API reports this.

## License
Copyright (C) 2020  w0d4
This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <https://www.gnu.org/licenses/>.

## Questions
### What happens if a video is already marked as downloaded in youtube-dl archive, but not in database and download is started
yt-backup will find the video id in youtube-dl download archive and set it's download date to 1972-01-01 23:23:23 in database. Since we don't know when the video was downloaded originally, we have marked it as downloaded anyways.