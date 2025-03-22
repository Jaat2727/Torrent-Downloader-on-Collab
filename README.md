# Simple Colab Torrent Downloader (Please ⭐)

This is a simple Python script for downloading torrents using magnet links in Google Colab. It includes progress monitoring and a keep-alive feature to prevent timeouts.

## How to Use (Copy-Paste Ready)

This is a simple Python script for downloading torrents using magnet links in Google Colab. It includes progress monitoring and a keep-alive feature to prevent timeouts.
 

## Proof of Result
![image](https://github.com/user-attachments/assets/6d98e9b5-4338-42fa-87c9-e8c2c07c214b)



**IMPORTANT: Before you begin, change the Colab runtime type to GPU for potentially faster downloads (although it's not strictly required for this script to *function*, network speed is generally better on GPU runtimes).  To do this:**

1.  **Click "Runtime" in the menu bar.**
2.  **Select "Change runtime type".**
3.  **Under "Hardware accelerator", choose "GPU".**
4.  **Click "Save".**  (This will restart the Colab runtime.)
5.  **If you want to skip creating a zip file and download it to your local storage after downloading instead you can mount your google drive using the code below and skip Skip the last step of creating a zip file, and you will find it in your drive directly after downloading.** (Zipping was necessary if your file exceeds 15 GB, which is the limit Google Drive.)
    
```bash
from google.colab import drive
drive.mount('/content/drive')
```

Now, follow these steps:

1.  **Install Dependencies:** Run this code in a Colab cell *before* running the main code:

     ```bash
    !pip install torrentp
    ```

    ```bash
    !pip install torrentp nest_asyncio
    ```

3.  **Copy the main code below** into a *new* Colab code cell.
4.  **Replace the `MAGNET_LINK`** with your actual magnet link.
5.  **Replace the `DOWNLOAD_PATH = "/content/drive/MyDrive/MyDownloads"`** with  **`DOWNLOAD_PATH = "/content"`** if you are not using Google Drive for Downlaoding else its fine no need to chnge if have mounted your Google Drive .
6.  **Run the cell.**

```python
import asyncio
import nest_asyncio
from torrentp import TorrentDownloader
import time
import os
import threading

# --- Configuration ---
MAGNET_LINK = "magnet:?xt=urn:btih:YOUR_MAGNET_LINK_HERE"  # <--- REPLACE THIS!
DOWNLOAD_PATH = "/content/drive/MyDrive/MyDownloads"  # Replace with your desired path
MONITOR_INTERVAL = 5  # Seconds

# Enable nested asyncio (REQUIRED for Colab)
nest_asyncio.apply()

# --- Keep-Alive (Prevent Timeout) ---
def keep_alive():
    """
    Simulates user interaction to prevent Colab from timing out.
    Runs in a separate thread.
    """
    while True:
        print("Keep-alive: Simulating activity...")
        time.sleep(60)  # Simulate activity every 60 seconds

# Start the keep-alive thread ONLY in Colab
if "COLAB_GPU" in os.environ:
    keep_alive_thread = threading.Thread(target=keep_alive, daemon=True)
    keep_alive_thread.start()
# --- End Keep-Alive ---

async def monitor_download(torrent_file, interval=MONITOR_INTERVAL):
    """Monitors download progress and prints statistics."""
    while not torrent_file.is_completed():
        stats = torrent_file.get_stats()
        total_size_gb = stats.total_size / (1024**3)
        downloaded_gb = stats.downloaded / (1024**3)
        progress = (stats.downloaded / stats.total_size) * 100 if stats.total_size else 0
        download_speed_mbps = (stats.download_speed / (1024**2)) * 8
        upload_speed_mbps = (stats.upload_speed / (1024**2)) * 8
        num_peers = stats.num_peers

        print(
            f"Progress: {progress:.2f}% | "
            f"Downloaded: {downloaded_gb:.2f} GB / {total_size_gb:.2f} GB | "
            f"Down Speed: {download_speed_mbps:.2f} Mbps | "
            f"Up Speed: {upload_speed_mbps:.2f} Mbps | "
            f"Peers: {num_peers}"
        )
        await asyncio.sleep(interval)

    print("Download Complete!")
    print(torrent_file.get_downloaded_files())


async def main():
    """Downloads a torrent from a magnet link, with monitoring."""
    torrent_file = None
    try:
        torrent_file = TorrentDownloader(MAGNET_LINK, DOWNLOAD_PATH)
        print("Starting download...")

        # Run download and monitoring concurrently
        download_task = asyncio.create_task(torrent_file.start_download())
        monitor_task = asyncio.create_task(monitor_download(torrent_file))

        # Wait for download to complete (or be interrupted)
        await download_task

        # Cancel the monitoring task
        monitor_task.cancel()
        try:
            await monitor_task  # Ensure cancellation is complete
        except asyncio.CancelledError:
            pass

    except Exception as e:
        print(f"An error occurred: {e}")
    finally:
        if torrent_file:
            print("Stopping download...")
            torrent_file.stop_download()

if __name__ == "__main__":
    loop = asyncio.get_event_loop()
    loop.run_until_complete(main())

`````````````````````````````````

6.  Downloading Files to Your Computer (After Download Completes)
    The downloaded files will be saved in the /content directory within the Colab environment. To download them to your local computer:

Run this code in a new Colab cell after the download is finished on collab :

 ```bash
!zip -r /content/downloaded_files.zip /content/*
from google.colab import files
files.download('/content/downloaded_files.zip')
```

or you can download manually from content folder
