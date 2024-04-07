# Automate Cleaning Up Your Movie Collection with Python

Hey everyone!
I've been working on a Python script that automatically deletes lower-resolution movie files in your collection, moves the deleted files to the Recycle Bin, and logs the amount of disk space freed up. It's perfect for those with large movie collections stored in a structured folder system, say, for PLEX. It currently finds (mkv|mp4|m4v|avi|mov|wmv|flv|mpg|mpeg|rm|rmvb|vob|asf|3gp|3g2|webm) file extensions. so if you have an MP4 file at a lower resolution than an MKV file of the same movie, it will delete it. Here's how you can set it up and run it yourself:
I use [FileBot](https://github.com/filebot) (an awesome program!) to rename my movies and TV shows.  The filename scheme I use is this:

```D:/{plex} - [{Resolution}, {vc} {bitdepth} bit {vbr}, {ac} {af} {abr}]```


***Step 1: Install Python***

1. Download the latest version of Python from the [official Python website](https://www.python.org/downloads/). Choose the installer for your operating system.
2. Run the installer. Important: Make sure to check the box that says "Add Python to PATH" during the installation process. This step makes it easier to run Python from the command line.

***Step 2: Install send2trash Library***

The script uses the send2trash library to safely move files to the Recycle Bin instead of permanently deleting them. To install this library, follow these steps:

1. Open Command Prompt (Windows) or Terminal (macOS/Linux).
2. Type the following command and press Enter:

```
pip install send2trash
```

***Step 3: Save the Python Script***

1. Open a text editor of your choice (e.g., Notepad, TextEdit, or Visual Studio Code).
2. Copy the Python script below and paste it into your text editor.

```python
import os
import re
import time
from send2trash import send2trash

start_time = time.time()

base_directory = 'D:\\TV Shows\\'
log_file = 'deleted_movies_log.txt'
pattern = re.compile(r"^(.*?) - \[(\d+)x\d+, .*\]\.(mkv|mp4|m4v|avi|mov|wmv|flv|mpg|mpeg|rm|rmvb|vob|asf|3gp|3g2|webm)$")

total_freed_space = 0  # In bytes

# Function to delete lower resolution files in a single directory and calculate freed space
def process_directory(directory):
    local_freed_space = 0  # Initialize local freed space for the current directory
    movies = {}
    for file in os.listdir(directory):
        full_path = os.path.join(directory, file)
        if os.path.isdir(full_path) or not pattern.match(file):
            continue  # Skip directories and non-matching files

        match = pattern.match(file)
        name, resolution, ext = match.groups()
        resolution = int(resolution)

        if name in movies:
            to_delete = movies[name]['filename'] if resolution > movies[name]['resolution'] else file
            delete_path = os.path.join(directory, to_delete)
            file_size = os.path.getsize(delete_path)
            send2trash(delete_path)
            local_freed_space += file_size
            with open(log_file, 'a') as log:
                log.write(f"Deleted {to_delete}\n")
            if resolution > movies[name]['resolution']:
                movies[name] = {'resolution': resolution, 'filename': file}
        else:
            movies[name] = {'resolution': resolution, 'filename': file}

    return local_freed_space

# Traverse subfolders and process each one
for root, dirs, files in os.walk(base_directory):
    for dir in dirs:
        dir_path = os.path.join(root, dir)
        total_freed_space += process_directory(dir_path)  # Accumulate freed space correctly

end_time = time.time()
elapsed_time = end_time - start_time

# Correct handling of large numbers for space conversion
freed_space_gb = total_freed_space / (1024 ** 3)  # Convert bytes to gigabytes

# Final logging
with open(log_file, 'a') as log:
    log.write(f"\nTotal space freed: {freed_space_gb:.2f} GB\n")
    log.write(f"Operation took: {elapsed_time:.2f} seconds\n")

print("Cleanup completed. Check 'deleted_movies_log.txt' for details.")
```

3. Save the file with a .py extension, for example, "cleanup_movies.py".

***Step 4: Run the Script***

1. Open Command Prompt or Terminal.

Navigate to the directory where you saved cleanup_movies.py. You can do this with the 'cd'command. For example, if you saved the script on your desktop, you might use 'cd Desktop' if it's under, say, D:\Movies, it would be 'cd D:\Movies'. If it's under folders with spaces in the name, such as D:\Movies 2010 - 2019, it would be 'cd "D:\Movies 2010 - 2019"'.

2. Run the script by typing the following command and pressing Enter:
```
python cleanup_movies.py 
```
The script will process your movie files, delete the lower-resolution versions based on the filename, move them to the Recycle Bin, and log the amount of disk space freed up in the text file along with how much time the operation took.
Important Notes:

- Backup Your Files: It's a good idea to backup your movie collection before running this script, just in case.
- Customize the Script: You may need to modify the base_directory variable in the script to match the location of your movie collection.

```pythonbase_directory = 'D:\\Movies\\'  # Path to your movies directory```

- Running the Script:

If you encounter any issues running the script, make sure Python is installed correctly and that you've installed the send2trashlibrary as described above.
Feel free to use, modify, and share this script. I hope it helps you manage your movie collection more effectively!


# NOTE: IF YOU DO NOT WANT TO SEND TO THE RECYCLE BIN OR SEE THE GB SAVED AND THE TIME TAKEN, HERE IS THAT SCRIPT:

```python
import os
import re

base_directory = 'D:\\Movies\\'  # Path to your movies directory
log_file = 'deleted_movies_log.txt'
pattern = re.compile(r"^(.*?) - \[(\d+)x\d+, .*\]\.(mkv|mp4|m4v|avi|mov|wmv|flv|mpg|mpeg|rm|rmvb|vob|asf|3gp|3g2|webm)$")

# Function to delete lower resolution files in a single directory
def process_directory(directory):
    movies = {}
    for file in os.listdir(directory):
        full_path = os.path.join(directory, file)
        if os.path.isdir(full_path):
            continue  # Skip directories

        match = pattern.match(file)
        if not match:
            continue

        name, resolution, ext = match.groups()
        resolution = int(resolution)

        if name in movies:
            if resolution > movies[name]['resolution']:
                # Delete the lower resolution file
                os.remove(os.path.join(directory, movies[name]['filename']))
                with open(log_file, 'a') as log:
                    log.write(f"Deleted {movies[name]['filename']}\n")
                movies[name] = {'resolution': resolution, 'filename': file}
            else:
                # Delete the current file as it's of lower resolution
                os.remove(full_path)
                with open(log_file, 'a') as log:
                    log.write(f"Deleted {file}\n")
        else:
            movies[name] = {'resolution': resolution, 'filename': file}

# Traverse subfolders and process each one
for root, dirs, files in os.walk(base_directory):
    for dir in dirs:
        process_directory(os.path.join(root, dir))

print("Cleanup completed. See 'deleted_movies_log.txt' for details.")
```

1. MKV, Matroska Video - MKV stands for Matroska Video, and is a free, open format for storing multimedia content in a container file. MKV files can contain an unlimited number of video, audio, picture, or subtitle tracks in one file. They are similar to MOV and AVI files, but offer more storage flexibility. MKV files are typically large because they don't compress data.
2. MP4, MPEG-4 Part 14 - MP4 is a digital multimedia container format most commonly used to store video and audio, but can also be used to store other data such as subtitles and still images. Like most modern container formats, it allows streaming over the Internet. MP4 files are compressed, which makes them more manageable in size compared to MKV files.
3. M4V, MPEG-4 Video - M4V is a video container format developed by Apple and is very similar to the MP4 format, with the primary difference being the optional Apple Digital Rights Management (DRM) protection. M4V files are often used for video content from the iTunes Store.
4. AVI, Audio Video Interleave - AVI is a multimedia container format introduced by Microsoft as part of its Video for Windows software in 1992. AVI files can contain both audio and video data in a file container that allows synchronous audio-with-video playback.
5. MOV, QuickTime File Format - MOV is a multimedia container format developed by Apple and is the file format for the QuickTime multimedia player. MOV files can contain multiple tracks that store different types of media data and are often used for saving movies and other video files.
6. WMV, Windows Media Video - WMV is a series of video codecs and their corresponding video coding formats developed by Microsoft. It is part of the Windows Media framework and commonly used for streaming and downloading content on the Internet.
7. FLV, Flash Video - FLV is a container file format used to deliver digital video content over the Internet using Adobe Flash Player. Despite the declining use of Flash technology, FLV files are still prevalent for streaming video on the web.
8. MPG/MPEG, Moving Picture Experts Group - MPG files are part of the MPEG family of multimedia coding and distribution standards set by the Moving Picture Experts Group. These files are compressed to allow for faster transmission over computer networks.
9. RM/RMVB, RealMedia/RealMedia Variable Bitrate - RM and RMVB are multimedia container formats created by RealNetworks. RM is typically used for streaming content over the Internet, while RMVB is used for multiplexing audio, video, and data.
10. VOB, Video Object - VOB is a container format in DVD-Video media. VOB files can contain digital video, digital audio, subtitles, DVD menus, and navigation contents multiplexed together into a stream form.
11. ASF, Advanced Systems Format - ASF is Microsoft's proprietary digital audio/digital video container format, especially meant for streaming media. ASF is part of the Windows Media framework.
12. 3GP/3G2, 3rd Generation Partnership Project/3rd Generation Partnership Project 2 - 3GP and 3G2 are container formats designed for video on mobile phones. 3GP is a simplified version of the MP4 format and was designed to make file sizes smaller so mobile phones could support video.
13. WEBM, WebM Video - WebM is a video file format intended for use with HTML5 video. It offers high-quality video compression and is designed for use on the web, with support across many browsers.

# This Python script automates the process of cleaning up a movie collection by deleting lower-resolution files within a specified directory structure, moving the deleted files to the Recycle Bin, and logging the operation's details. Here's a step-by-step explanation of how the script works:

***1. Setting Up***

It imports necessary modules: os for interacting with the operating system, re for regular expressions, time to track the operation's duration, and send2trash to safely move files to the Recycle Bin.
Initializes variables for the start time, base directory path (D:\\Movies\\), log file name, and a compiled regular expression pattern to match filenames.

***2. Compiled Regular Expression Pattern***

The pattern is designed to match file names that follow a specific format: a name followed by details enclosed in brackets (including resolution and file extension), such as "Show Name - [Resolution, Codec, Audio].ext".
This pattern captures the name of the show, the resolution (the number before 'x' in dimensions like 1920x1080), and the file extension.

***3. Function: process_directory***

Iterates over each file in a given directory, skipping directories and files that don't match the pattern.
For files that match, it extracts the movie's name, resolution, and extension.
It compares the current file's resolution with any previously encountered file with the same name but different resolutions stored in a dictionary named movies.
If a file with the same name but a higher resolution is found, the script marks the lower resolution file for deletion.
If the current file is of a higher resolution than a previously noted file, it updates the record to keep the current file and marks the other for deletion.
Deletes the marked file using send2trash, ensuring it goes to the Recycle Bin, and logs the deletion in deleted_movies_log.txt.
Accumulates the size of deleted files to calculate the total freed space.

***4. Traversing Subfolders***

The script walks through the base_directory and its subdirectories using os.walk, processing each directory by calling process_directory.
This approach allows the script to recursively clean up the entire folder structure under the specified base directory.

***5. Calculating and Logging Results***

After processing all directories, the script calculates the total space freed by deleted files and the operation's total duration.
It logs these details, along with a list of deleted files, in deleted_movies_log.txt.
Finally, it prints a completion message, signaling the end of the cleanup process.

Summary
The script intelligently identifies and deletes lower-resolution movie files based on filename patterns, prioritizing the retention of higher-resolution versions. This cleanup not only helps in optimizing storage space but also ensures the collection maintains the best possible video quality.







