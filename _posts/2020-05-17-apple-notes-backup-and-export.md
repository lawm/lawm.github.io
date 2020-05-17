---
layout: "post"
title: "Apple Notes Backup and Export"
date: "2020-05-17 14:01"
---
## Motivation
Time Machine works to backup Apple Notes on macOS, but I also want:
* a secondary backup that's easier for me to store in other places
* to see the data to be sure it's backed up correctly
* have the data in a format that I can transfer to other apps if needed

## Steps
1. Backup your Notes and computer.
2. Copy your Notes folder to a working directory

    ```
    mkdir notes-test
    cd notes-test

    cp -r /Users/$USER/Library/Group\ Containers/group.com.apple.notes ./
    ```

3. Download the tool made by threeplanetssoftware:

    ```
    git clone https://github.com/threeplanetssoftware/apple_cloud_notes_parser.git

    # Or use my branch to skip table parsing and fix linebreaks/indentation:
    https://github.com/lawm/apple_cloud_notes_parser

    cd apple_cloud_notes_parser
    # Install ruby, bundle, and other dependencies
    sudo apt-get install build-essential libsqlite3-dev zlib1g-dev git ruby-full ruby-bundler

    bundle install
```

4. Run the tool to convert from Apple's Notes format to a more readable and portable format.
    ```
    ruby notes_cloud_ripper.rb -m ../group.com.apple.notes
    ```
The output will include

    * `output/TIMESTAMP/NoteStore.sqlite`
        * plain text sqlite database
        * Read with sqlite3, or other sqlite browsing tool e.g.
            ```
            $ sqlite3 NoteStore.sqlite
            sqlite> .tables
            sqlite> select * from ZICNOTEDATA;
            ```
    * `output/TIMESTAMP/all_notes_1.html`
        * HTML version of all the notes
        * Open with a web browser or text editor.
        * Inspect to make sure formatting and other metadata important to you is preserved.
    * `output/TIMESTAMP/files/`
        * attachments and embedded picture files

5. Now the `output/TIMESTAMP` folder can be zip'd and backed up to the location of your choice.