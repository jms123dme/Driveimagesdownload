import os
import re
import requests
import streamlit as st
from datetime import datetime
from pathlib import Path  # For cross-platform Downloads folder

def convert_google_drive_link(link):
    """Convert a Google Drive link to a direct download link."""
    if "drive.google.com" in link:
        # Extract the file ID from the link
        if "id=" in link:
            file_id = link.split("id=")[-1].split("&")[0]
        elif "/d/" in link:
            file_id = link.split("/d/")[-1].split("/")[0]
        else:
            return None
        return f"https://drive.google.com/uc?id={file_id}&export=download"
    return link

def sanitize_filename(url):
    """Generate a safe filename from a URL and append .jpeg extension."""
    # Extract a safe portion of the URL for the filename
    filename = re.sub(r"[^\w\-_.]", "_", url.split("/")[-1])  # Replace invalid characters
    filename = filename[:100]  # Limit filename length to 100 characters
    if not filename.endswith(".jpeg"):  # Ensure the file has a .jpeg extension
        filename += ".jpeg"
    return filename

def download_image(url, folder):
    """Download an image from a URL to a specified folder and save it as .jpeg."""
    try:
        # Follow redirects
        response = requests.get(url, stream=True, allow_redirects=True)
        if response.status_code == 200:
            # Sanitize filename and save as .jpeg
            filename = sanitize_filename(url)
            filepath = os.path.join(folder, filename)
            # Save file
            with open(filepath, "wb") as f:
                for chunk in response.iter_content(1024):
                    f.write(chunk)
            return filepath
        else:
            return None
    except Exception as e:
        st.error(f"Error downloading {url}: {e}")
    return None

def main():
    """Streamlit app to download images."""
    st.title("Image Downloader")
    st.write("Paste your image URLs (comma-separated) below to download them as JPEGs.")

    # User input
    links_input = st.text_area("Paste URLs (comma-separated):", height=150)

    # Button to trigger download
    if st.button("Download Images"):
        if not links_input.strip():
            st.error("Please provide at least one URL.")
            return

        # Process links
        links = [link.strip() for link in links_input.split(",") if link.strip()]
        converted_links = [convert_google_drive_link(link) for link in links]
        converted_links = [link for link in converted_links if link]

        if not converted_links:
            st.error("No valid links provided!")
            return

        # Create a folder in the Downloads directory
        downloads_folder = str(Path.home() / "Downloads")
        today_date = datetime.now().strftime("%Y-%m-%d")
        folder_name = f"Cleaning Images {today_date}"
        folder = os.path.join(downloads_folder, folder_name)

        # Ensure the folder exists
        if not os.path.exists(folder):
            os.makedirs(folder)

        # Download images
        st.write("Starting download...")
        success_count = 0
        for link in converted_links:
            if download_image(link, folder):
                success_count += 1

        # Display results
        if success_count:
            st.success(f"Downloaded {success_count} images to the folder: {folder}")
            st.write(f"Folder Path: `{folder}`")
        else:
            st.error("No images were downloaded.")

# Run the app
if __name__ == "__main__":
    main()
