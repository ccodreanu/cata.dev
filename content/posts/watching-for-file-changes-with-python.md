---
title: Watching for file changes with Python
date: 2019-09-01 12:00:01
tags: code
---
Working on [Generatore](https://github.com/picofish/generatore) I wanted to be able to quickly rebuild the website when new changes were introduced to my posts.

If you need to implement something like that, you need [Watchdog](https://pythonhosted.org/watchdog/).

Quoting from my project, you'll need to implement a custom handler inheriting from `FileSystemEventHandler`:

```python
import os

from watchdog.events import FileSystemEventHandler
from watchdog.observers import Observer

class ContentHandler(FileSystemEventHandler):
    def __init__(self, site):
        self.site = site
    def on_modified(self, event):
        print(f'{event.src_path} has been {event.event_type}')
        self.site.build_site()
```

With `self.site.build_site()` being the function that you need to call when files are modified.

The code for the observer will be something like this:

```python
content_handler = ContentHandler(SiteBuilder())
observer = Observer()
observer.schedule(content_handler, path=os.path.join(os.getcwd(), 'content'), recursive=True)
observer.start()

print('Listening for changes...')

try:
    while True:
        time.sleep(1)
except KeyboardInterrupt:
    observer.stop()
observer.join()
```

Having `path` set to `./content/`, the observer will watch for changes in that directory, recursivly, because of `recursive=True`.

That's it!
