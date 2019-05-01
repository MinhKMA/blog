---
title: "{{ replace .TranslationBaseName "-" " " | title }}"

date: {{ .Date }}

categories:
# - category
# - subcategory

tags:
# - tag1
# - tag2

keywords:
# Define keywords for search engines. you can also define global keywords 
# in Hugo configuration file.
# - tech

# thumbnailImage: //example.com/image.jpg
# Image displayed in index view.

thumbnailImagePosition : left
# Display thumbnail image at the right of title 
# in index pages (right, left or bottom). thumbnailImagePosition 
# overwrite the setting thumbnailImagePosition in the theme configuration file.

coverImage:
# Image displayed in full size at the top of your post in post view. If thumbnail image is not configured, cover image is also used as thumbnail image.

coverSize:
# partial: cover image take a part of the screen height (60%), full: cover image take the entire screen height.

coverCaption:
# Add a caption under the cover image
    
gallery:
# - image 1
# - image 2
comments: true
summary:
# Custom excerpt text to show on the homepage.
---