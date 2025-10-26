# INTRODUCTION TO FLASK AND PROGRESSIVE WEB APPS TUTORIAL

This guided tutorial will introduce HSC Software Engineering to the basics of developing websites with the [Python Flask framework](https://flask.palletsprojects.com/en/3.0.x/). The tutorial has been specifically designed for requirements in the [NESA Software Engineering Syllabus](https://curriculum.nsw.edu.au/learning-areas/tas/software-engineering-11-12-2022/content/n12/fa6aab137e) and for students in NSW Department of Education schools using eT4L computers.

A [list of popular PWA's](https://business.adobe.com/blog/basics/progressive-web-app-examples) (including Ube, Spotify, Facebook and Google Maps)

## Overview of Progressive Web Apps

A [Progressive Web Apps (PWAs)](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps?ref=arctype.com) is an app that is built using web platform technologies, but that provides a user experience like that of a platform-specific app. Like a website, a PWA can run on multiple platforms and devices from a single codebase. Like a platform-specific app, it can be installed on the device, can operate while offline and in the background, and can integrate with the device and with other installed apps.

### Technical features of PWAs

Because PWAs are websites, they have the same basic features as any other website: at least one HTML page, which loads CSS and JavaScript. Javascript is the language of the web and is exclusively used for the client-side front end; python, in the web context, can only be used in the back end. Like a normal website, the JavaScript loaded by the page has a global Window object and can access all the Web APIs that are available through that object. The PWA standard as defined by [W3C Standards](https://www.w3.org/standards/) has some specific features additional to a website:

| Feature             | Purpose                                                                                                                                                                                                                                                                                                                                                       |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| manifest.json       | An app manifest file, which, at a minimum, provides information that the operating system needs to install the PWA, such as the app name, screen orientation and icon set for different-sized views.                                                                                                                                                          |
| serviceworker.js    | A service worker, which, at a minimum, manages the caching that enables an online and offline experience whilst also interfacing with API's such as the [notification web API](https://developer.mozilla.org/en-US/docs/Web/API/Notification). It's important to understand that this JS file cannot control the DOM of the application for security reasons. |
| Icons & screenshots | A set of icons and screenshots that are used when uploading to an app store and when installing it as a native application. It is these icons that will be used in the desktop or app launcher when installed.                                                                                                                                                |
| Installable         | Because of the information contained in the manifest.json all PWA's can be installed like a native app. They can also be packaged and uploaded to the Google, Microsoft & Apple app stores.                                                                                                                                                                   |
| Cached locally      | Because the service worker details all apps and pages to be cached (all pages must have a \*.html name), the app and its resources can be cached locally for quick load times.                                                                                                                                                                                |

_Note backend apps where the web server serves all pages from the DNS root do not meet the PWA specification._

The below image illustrates how the servicework manages online and offline behaviour.

![A highlevel illustration of the service worker](/docs/README_resources/Progressive-Web-Apps-Architecture.png "The service worker handles the initial requests and sets the behaviour depending on if the app is on or offline.")

## Your end product

This screen capture shows how the final PWA will be rendered to the user.

![Screen capture of the finished PWA](/docs/README_resources/final_app.png "This is what your application will look like")

## Requirements

1. [VSCode](https://code.visualstudio.com/download)
2. [Python 3.x](https://www.python.org/downloads/)
3. [GIT 2.x.x +](https://git-scm.com/downloads)

> [!Important]
> MacOS and Linux users may have a `pip3` soft link instead of `pip`, run the below commands to see what path your system is configured with and use that command through the project. If neither command returns a version, then likely [Python 3.x](https://www.python.org/downloads/) needs to be installed.
>
> ```bash
> pip show pip
> pip3 show pip
> ```

## Prior learning

1. Bash basics
2. SQL
3. HTML Basics
4. CSS Basics
5. Python

## STEPS TO BUILDING YOUR FIRST PWA

### Setup your environment

![Screen recording of setting up VSCode](/docs/README_resources/get_vscode_started.gif "Follow these steps to setup VSCode")

> [!Note]
> Helpful VSCode settings are configured in [.vscode/settings.json](/.vscode/settings.json) which will automatically apply if you are not using a custom profile. If you are using a custom profile, it is suggested you manually apply those settings to your profile, especially the \*.md file association, so the README.md default opens in preview mode and setting _bash_ as your default terminal.

1. Install the necessary extensions for this tutorial.

| Required Extensions                                                                                              | Suggested Python Extensions                                                                                  |
| ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| [medo64.render-crlf](https://marketplace.visualstudio.com/items?itemName=medo64.render-crlf)                     | [ms-python.flake8](https://marketplace.visualstudio.com/items?itemName=ms-python.flake8)                     |
| [McCarter.start-git-bash](https://marketplace.visualstudio.com/items?itemName=McCarter.start-git-bash)           | [ms-python.black-formatter](https://marketplace.visualstudio.com/items?itemName=ms-python.black-formatter)\* |
| [yy0931.vscode-sqlite3-editor](https://marketplace.visualstudio.com/items?itemName=yy0931.vscode-sqlite3-editor) | [ms-python.python](https://marketplace.visualstudio.com/items?itemName=ms-python.python)                     |
|                                                                                                                  | [oderwat.indent-rainbow](https://marketplace.visualstudio.com/items?itemName=oderwat.indent-rainbow)         |
|                                                                                                                  | [esbenp.prettier-vscode](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)\*       |

_\*You will need to configure your formatters, it is recommended esbenp.prettier-vscode is your default formatter and ms-python.black-formatter is the Python language formatter_

> [!Important]
> From now on, you should aim to run all commands from the CLI. You are discouraged from left/right clicking the GUI. You will find it feels slow at first, but through disciplined use, you will become much quicker and more accurate with CLI commands than GUI controls.

Make sure you open a new terminal with the keys <kbd>Ctrl</kbd> + <kbd>`</kbd> and choose Git Bash from the menu option in the top right of the terminal shell.

![Screen capture of the menu options for terminals](/docs/README_resources/git_bash_shell.png "Choose Git Bash from the list")

1. Get the working files, which include this README.md
   - Open a new window in VSCode
   - Choose your working directory

```bash
git clone https://github.com/TempeHS/Flask_PWA_Programming_For_The_Web_Task_Template.git
cd Flask_PWA_Programming_For_The_Web_Task_Template
```

> [!TIP]
> Alternatively, you can fork the [template repository](https://github.com/TempeHS/Flask_PWA_Programming_For_The_Web_Task_Template) to your own GitHub account and open it in a Codespace in which all dependencies and extensions will be automatically installed.

4. Install necessary dependencies.

```bash
pip install flask
```

#### ✅ Checkpoint: Environment Setup Complete

```bash
# Verify your setup:
python --version    # Should show Python 3.x
pip show flask      # Should show Flask is installed
code --version      # Should show VSCode is working
```

> **Having issues?** See [🔧 Troubleshooting - Environment Setup Issues](#-troubleshooting-common-issues)

---

### Create files and folders for your Flask Project

1. Make a folder for all your working documents like photoshop \*.psd files, developer documentation etc.

```bash
mkdir working_documents
```

2. Create a license file.

```bash
touch LICENSE
code LICENSE
```

Copy the [GNU GPL license](https://www.gnu.org/licenses/gpl-3.0.txt) text into the file. GNU GPL is a free software license, or copyleft license, that guarantees end users the freedom to run, study, share, and modify the software. 3. Create your directory structure and some base files using BASH scripts reading text files.

```text
├── database
├── working_documents
├── static
│   ├── css
│   ├── icons
│   ├── images
│   ├── js
├── templates
├── LICENSE
├── main.py
└── database_manager.py
```

3. Populate a text file with a list of folders (see above directory structure) you need at the root of your project.

```bash
touch folders.txt
code folders.txt
```

4. Run a BASH script to read the text file and create the folders listed in it.

```bash
while read -r line; do
echo $line
mkdir -p $line
done < folders.txt
```

5. Populate the file with a list of files you need at the root of your project (see above directory structure).

```bash
touch files.txt
code files.txt
```

6. Run a BASH script to read the text file and create the files listed in it.

```bash
while read -r line; do
echo $line
touch -p $line
done < files.txt
```

> [!Important]
>
> - The last list item needs a line ending, so make sure there is a blank last line in the file.
> - You will find that all file and folder names have an unwanted `space` character at the end. This is because you are using a BASH emulator on the Windows operating system. Bash is a Unix language that uses [LF Unicode character 000A while Windows uses CRLF Unicode characters 000D + 000A](https://learn.microsoft.com/en-us/visualstudio/ide/encodings-and-line-breaks?view=vs-2022). Because you have installed the [medo64.render-crlf](https://marketplace.visualstudio.com/items?itemName=medo64.render-crlf) extension, click on `CRLF` in the bottom bar of VSCode and choose `LF` to change the line ending before running your BASH script.

#### ✅ Checkpoint: Project Structure Created

```bash
# Verify your project structure:
ls -la
# Should show: LICENSE, main.py, database_manager.py, static/, templates/, database/, working_documents/
tree -L 2
# Should show the complete folder structure
```

---

### Setup your SQLite3 Database

```bash
cd database
touch data_source.db
touch my_queries.sql
code my_queries.sql
```

> [!Note]
> The following SQL queries are provided as an example only. Students are encouraged to select their content and design a database schema for it; ideas include:
>
> - Favourite bands
> - Favourite movies
> - Favourite games
> - Favourite books
> - etc

1. To run SQLite3 SQL queries in VSCode
   Open the DB file, then choose "Query Editor" from the top menu.

```bash
code data_source.db
```

![Screen capture of query editor](/docs/README_resources/query_editor.png "Choose Query Editor from the top menu")

```sql
CREATE TABLE extension(extID INTEGER NOT NULL PRIMARY KEY,name TEXT NOT NULL, hyperlink TEXT NOT NULL,about TEXT NOT NULL,image TEXT NOT NULL,language TEXT NOT NULL);
```

**Understanding the Database Schema:**
This creates a table called `extension` with 6 columns:

- `extID` - Unique identifier number (Primary Key, auto-increments)
- `name` - The extension's display name (e.g., "Live Server")
- `hyperlink` - URL to the extension's marketplace page
- `about` - Description of what the extension does
- `image` - URL to the extension's icon/screenshot
- `language` - Programming language the extension supports

2. After running each query put `--` infront of the query to turn it into a comment so it doesn't run again and error.
3. Run SQL queries to populate your table.

**Understanding INSERT Statements:**
Each INSERT adds one row of data. The VALUES must match the column order exactly:
`(extID, name, hyperlink, about, image, language)`

```sql
INSERT INTO extension(extID,name,hyperlink,about,image,language) VALUES (1,"Live Server","https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer","Launch a development local Server with live reload feature for static & dynamic pages","https://ritwickdey.gallerycdn.vsassets.io/extensions/ritwickdey/liveserver/5.7.9/1736542717282/Microsoft.VisualStudio.Services.Icons.Default","HTML CSS JS");
```

```sql
INSERT INTO extension(extID,name,hyperlink,about,image,language) VALUES (2,"Render CR LF","https://marketplace.visualstudio.com/items?itemName=medo64.render-crlf","Displays the line ending symbol and optionally extra whitespace when 'Render whitespace' is turned on.","https://medo64.gallerycdn.vsassets.io/extensions/medo64/render-crlf/1.7.1/1689315206970/Microsoft.VisualStudio.Services.Icons.Default","#BASH");
```

```sql
INSERT INTO extension(extID,name,hyperlink,about,image,language) VALUES (3,"Start GIT BASH","https://marketplace.visualstudio.com/items?itemName=McCarter.start-git-bash","Adds a bash command to VSCode that allows you to start git-bash in the current workspace's root folder.","https://mccarter.gallerycdn.vsassets.io/extensions/mccarter/start-git-bash/1.2.1/1499505567572/Microsoft.VisualStudio.Services.Icons.Default","#BASH");
```

```sql
INSERT INTO extension(extID,name,hyperlink,about,image,language) VALUES (4,"SQLite3 Editor","https://marketplace.visualstudio.com/items?itemName=yy0931.vscode-sqlite3-editor","Edit SQLite3 files like you would in spreadsheet applications.","https://yy0931.gallerycdn.vsassets.io/extensions/yy0931/vscode-sqlite3-editor/1.0.85/1690893830873/Microsoft.VisualStudio.Services.Icons.Default","SQL");
```

4. Run some SQL queries to test your database.

**Understanding SELECT Statements:**

- `SELECT *` means "show all columns"
- `FROM extension` specifies which table to query
- `WHERE` adds filtering conditions

```sql
SELECT * FROM extension;
-- This shows ALL rows and ALL columns from the extension table

SELECT * FROM extension WHERE language LIKE '#BASH';
-- This shows only extensions where the language column contains '#BASH'
-- LIKE is used for pattern matching (similar to "contains")
```

#### ✅ Checkpoint: Database Setup Complete

```bash
# Verify your database:
ls database/
# Should show: data_source.db, my_queries.sql
sqlite3 database/data_source.db ".tables"
# Should show: extension
```

> **Database issues?** See [🔧 Troubleshooting - Database Issues](#-troubleshooting-common-issues)

---

### Make your graphic assets

> [!Note]
> Graphic design is not the focus of this course. It is suggested that you do not spend excessive time designing logos and icons.

1. Use Photoshop or [Canva](https://www.canva.com/en_au/signup/?signupRedirect=%2Fedu-signup&loginRedirect=%2Fedu-signup&brandingVariant=edu) to design a simple square logo 1080px X 1080px named logo.png. Save all working files (\*.psd, pre-optimised originals, etc) into the working_documents directory.
2. Design a simplified app icon 512px X 512px named favicon.png.
3. Web optimise the images using [TinyPNG](https://tinypng.com/).
4. Save the files into the static/images folder.
5. Rename the 512x512 icon to icon-512x512.png, then resize and rename it as follows:
   - icon-128x128.png
   - icon-192x192.png
   - icon-384x384.png
   - icon-512x512.png
6. Web optimise the images using [TinyPNG](https://tinypng.com/)
7. Save the optimised icons to static/icons
8. Save the optimised logo and favicon to static/images

#### ✅ Checkpoint: Graphics Assets Ready

```bash
# Verify your assets:
ls static/icons/
# Should show: icon-128x128.png, icon-192x192.png, icon-384x384.png, icon-512x512.png
ls static/images/
# Should show: logo.png, favicon.png
```

---

### Setup your index.html using the Jinja2 template engine

**Understanding Template Engines:**
Jinja2 is Flask's template engine that lets you embed Python-like code in HTML:

- `{{ variable }}` - Display data from Python
- `{% statement %}` - Control logic (loops, conditions)
- `{% extends 'file.html' %}` - Inherit from another template
- `{% block name %}` - Define reusable sections

> [!Note]
> Adjust titles, headings and content to match your concept.

```bash
cd ../templates
touch layout.html
code layout.html
```

1. Insert the basic HTML structure in your templates/layout.html file.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <meta http-equiv="Content-Security-Policy" content="script-src 'self';" />
    <link rel="stylesheet" href="static/css/style.css" />
    <title>VSCode Extension Catalogue</title>
    <link rel="manifest" href="static/manifest.json" />
    <link rel="icon" type="image/x-icon" href="static/images/favicon.png" />
  </head>

  <body>
    <main>
      {% include "partials/menu.html" %} {% block content %}{% endblock %}
    </main>
    <script src="static/js/app.js"></script>
  </body>
</html>
```

**Template Engine Syntax Explained:**

- `{% include "partials/menu.html" %}` - Inserts content from another file
- `{% block content %}{% endblock %}` - Creates a placeholder for child templates to fill

6. Insert the block content into index.html, you will add more later.

```bash
touch index.html
code index.html
```

**Template Inheritance:**
This index.html inherits from layout.html and fills the content block:

```html
{% extends 'layout.html' %} {% block content %}
<div class="container"></div>
{% endblock %}
```

#### ✅ Checkpoint: Templates Created

```bash
# Verify your templates:
ls templates/
# Should show: layout.html, index.html
ls templates/partials/
# Should show: (folder exists for next section)
```

---

### Style the HTML core

```bash
cd ../static/css
touch style.css
code style.css
```

1. Insert the css code into static/css/style.css.

```css
@import url("https://fonts.googleapis.com/css?family=Nunito:400,700&display=swap");

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  background: #fdfdfd;
  font-family: "Nunito", sans-serif;
  font-size: 1rem;
}

main {
  max-width: 900px;
  margin: auto;
  padding: 0.5rem;
  text-align: center;
}
```

---

### Make and style the menu

```bash
cd ../../templates
mkdir partials
cd partials
touch menu.html
code menu.html
```

1. Insert the menu HTML into menu.html.

```html
<nav>
  <img src="static\images\logo.png" alt="VSCode Extensions site logo." />
  <h1>VSCode Extensions</h1>
  <ul class="topnav">
    <li><a href="#">Home</a></li>
    <li><a href="add.html">Add me</a></li>
    <li><a href="about.html">About</a></li>
  </ul>
</nav>
```

```bash
`cd ../../static/css`
`code style.css`
```

2. Style the menu by inserting this below your existing CSS in static/css/style.css.

```css
nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

nav img {
  height: 100px;
}

nav ul {
  list-style: none;
  display: flex;
}

nav li {
  margin-right: 1rem;
}

nav ul li a {
  text-decoration-line: none;
  text-transform: uppercase;
  color: #393b45;
}

nav ul li a:hover {
  color: #14e6dd;
}

nav h1 {
  color: #106d69;
  margin-bottom: 0.5rem;
}
```

<HR

### Render your website

```bash
cd ../..
code main.py
```

1. Insert the Flask python to the backend script.

**Understanding HTTP Methods in Flask Routes:**

- **GET**: Used when requesting/viewing a page (default method)
- **POST**: Used when submitting form data to the server
- **methods=['GET']**: Only accepts GET requests (viewing the page)
- **methods=['POST', 'GET']**: Accepts both GET (viewing) and POST (form submission)

```python
from flask import Flask
from flask import render_template
from flask import request
import database_manager as dbHandler

app = Flask(__name__)

@app.route('/index.html', methods=['GET'])     # Only GET - just viewing
@app.route('/', methods=['POST', 'GET'])       # Both GET and POST - can handle forms
def index():
  return render_template('/index.html')

if __name__ == '__main__':
  app.run(debug=True, host='0.0.0.0', port=5000)
```

2. Run the builtin webserver.

```bash
python main.py
```

3. Visit your website and look at the source in developer tools to see how the page has been rendered.

**Testing HTTP Methods:**

- **Browser address bar**: Always sends GET requests
- **Form submission**: Sends POST requests (when method="POST")
- **DevTools → Network tab**: Shows all HTTP requests and their methods

> [!Note]
> To explain how the Jinja2 template engine works in this example when index.html is called, the render will start with layout.html with the code from partials/menu.html inserted where `{% include "partials/menu.html" %}` is and the index.html content that is between the `{% block content %}` and `{% endblock %}` will be inserted in the same tags in the layout.html.

#### ✅ Checkpoint: Flask App Running

```bash
# Verify Flask is working:
python main.py
# Should show: * Running on all addresses (0.0.0.0)
# Open browser to http://localhost:5000
# Should see your website with menu and basic layout
```

> **Flask not starting?** See [🔧 Troubleshooting - Flask Application Issues](#-troubleshooting-common-issues)

---

### Query your SQL database and migrate the content to the frontend as HTML

```bash
code database_manager.py
```

1. Query the database and store the data in a variable.

```python
import sqlite3 as sql

def listExtension():
  con = sql.connect("database/data_source.db")
  cur = con.cursor()
  data = cur.execute('SELECT * FROM extension').fetchall()
  con.close()
  return data
```

```bash
code main.py
```

2. Pass the data to the front end by modifying the existing `app.route`.

```python
def index():
   data = dbHandler.listExtension()
   return render_template('/index.html', content=data)
```

```bash
cd templates
code index.html
```

3. Use the Jinja2 template engine to pass the data (which is a [tuple](https://www.w3schools.com/python/python_tuples.asp)) to front end content. Insert the HTML inside the `<div class="container">` of the index.html.

**Template Engine Loop Syntax:**

- `{% for row in content %}` - Start a loop through the data
- `{{ row[1] }}` - Display data from the row
- `{% endfor %}` - End the loop

**Understanding Database-to-HTML Mapping:**
The `row` variable contains data in the same order as your database columns:

- `row[0]` = extID (not used in display)
- `row[1]` = name (extension title)
- `row[2]` = hyperlink (link destination)
- `row[3]` = about (description text)
- `row[4]` = image (icon URL)
- `row[5]` = language (not used in this template)

```html
{% for row in content %}
<div class="card">
  <img
    class="card-image"
    src="{{ row[4] }}"
    alt="Product image for the {{ row[1] }} VSCode extension."
  />
  <h1 class="card-name">{{ row[1] }}</h1>
  <p class="card-about">{{ row[3] }}</p>
  <a class="card-link" href="{{ row[2] }}"
    ><button class="btn">Read More</button></a
  >
</div>
{% endfor %}
```

```bash
cd ../static/css
code style.css
```

4. Style the cards by inserting this below your existing CSS in static/css/style.css.

```css
.container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(15rem, 1fr));
  grid-gap: 1rem;
  justify-content: center;
  align-items: center;
  margin: auto;
  padding: 1rem 0;
}

.card {
  display: flex;
  align-items: center;
  flex-direction: column;
  width: 17rem;
  background: #fff;
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.19), 0 6px 6px rgba(0, 0, 0, 0.23);
  border-radius: 10px;
  margin: auto;
  overflow: hidden;
}

.card-image {
  width: 100%;
  height: 15rem;
  object-fit: cover;
}

.card-name {
  color: #222;
  font-weight: 700;
  text-transform: capitalize;
  font-size: 1.1rem;
  margin-top: 0.5rem;
}

.card-about {
  text-overflow: ellipsis;
  width: 15rem;
  white-space: nowrap;
  overflow: hidden;
  margin-bottom: 1rem;
}

.btn {
  border: none;
  background: none;
  border-radius: 5px;
  box-shadow: 1px 1px 2px rgba(21, 21, 21, 0.1);
  cursor: pointer;
  font-size: 1.25rem;
  margin: 0 1rem;
  padding: 0.25rem 2rem;
  transition: all 0.25s ease-in-out;
  background: hsl(110, 21%, 93%);
  color: hsl(141, 100%, 22%);
  margin-bottom: 1rem;
}

.btn:focus,
.btn:hover {
  box-shadow: 1px 1px 2px rgba(21, 21, 21, 0.2);
  background: hsl(111, 21%, 86%);
}

.about-container {
  font-size: 1.25rem;
  margin-top: 2rem;
  text-align: justify;
  text-justify: inter-word;
}
```

#### ✅ Checkpoint: Database Integration Complete

```bash
# Verify database integration:
python main.py
# Open browser to http://localhost:5000
# Should see cards displaying data from your database
# Check browser console for any errors
```

---

### Finish the PWA code so it is compliant to W3 web standards

1. Take a screen shot of the website. Then size the image to 1080px X 1920px, web optimise the images using [TinyPNG](https://tinypng.com/) and save it to static/icons.

```bash
cd ..
code manifest.json
```

2. Configure the manifest.json to the PWA standard by inserting the JSON below and validating the JSON with [jsonlint](https://jsonlint.com/). The manifest.json sets the configuration for the installation and caching of the PWA.

```json
{
  "name": "VSCode Extension Catalogue",
  "short_name": "vscodeextcat",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#fdfdfd",
  "theme_color": "#14E6DD",
  "orientation": "landscape-primary",
  "icons": [
    {
      "src": "icons/icon-128x128.png",
      "type": "image/png",
      "sizes": "128x128",
      "purpose": "maskable"
    },
    {
      "src": "icons/icon-128x128.png",
      "type": "image/png",
      "sizes": "128x128",
      "purpose": "any"
    },
    {
      "src": "icons/icon-192x192.png",
      "type": "image/png",
      "sizes": "192x192",
      "purpose": "maskable"
    },
    {
      "src": "icons/icon-192x192.png",
      "type": "image/png",
      "sizes": "192x192",
      "purpose": "any"
    },
    {
      "src": "icons/icon-384x384.png",
      "type": "image/png",
      "sizes": "384x384",
      "purpose": "maskable"
    },
    {
      "src": "icons/icon-384x384.png",
      "type": "image/png",
      "sizes": "384x384",
      "purpose": "any"
    },
    {
      "src": "icons/icon-512x512.png",
      "type": "image/png",
      "sizes": "512x512",
      "purpose": "maskable"
    },
    {
      "src": "icons/icon-512x512.png",
      "type": "image/png",
      "sizes": "512x512",
      "purpose": "any"
    }
  ],
  "screenshots": [
    {
      "src": "icons/desktop_screenshot.png",
      "sizes": "1920x1080",
      "type": "image/png",
      "label": ""
    },
    {
      "src": "icons/mobile_screenshot.png",
      "sizes": "1080x1920",
      "type": "image/png",
      "form_factor": "wide",
      "label": ""
    }
  ]
}
```

```bash
cd js
code app.js
```

2. Configure the app.js to initiate the servicework.js by inserting the JS. This ensures that when the window (app) loads the serviceworker.js is called to memory.

```js
if ("serviceworker" in navigator) {
  window.addEventListener("load", function () {
    navigator.serviceworker
      .register("static/js/serviceworker.js")
      .then((res) => console.log("service worker registered"))
      .catch((err) => console.log("service worker not registered", err));
  });
}
```

```bash
cd js
code serviceworker.js
```

1. Configure the serviceworker.js by inserting the JS. The serviceworker.js, as the name suggests, is the file that does all the work in a PWA, including caching and API integration for the [WEB APIs](https://developer.mozilla.org/en-US/docs/Web/API).

```js
const assets = [
  "/",
  "static/css/style.css",
  "static/js/app.js",
  "static/images/logo.png",
  "static/images/favicon.png",
  "static/icons/icon-128x128.png",
  "static/icons/icon-192x192.png",
  "static/icons/icon-384x384.png",
  "static/icons/icon-512x512.png",
  "static/icons/desktop_screenshot.png",
  "static/icons/mobile_screenshot.png",
];

const CATALOGUE_ASSETS = "catalogue-assets";

self.addEventListener("install", (installEvt) => {
  installEvt.waitUntil(
    caches
      .open(CATALOGUE_ASSETS)
      .then((cache) => {
        console.log(cache);
        cache.addAll(assets);
      })
      .then(self.skipWaiting())
      .catch((e) => {
        console.log(e);
      })
  );
});

self.addEventListener("activate", function (evt) {
  evt.waitUntil(
    caches
      .keys()
      .then((keyList) => {
        return Promise.all(
          keyList.map((key) => {
            if (key === CATALOGUE_ASSETS) {
              console.log("Removed old cache from", key);
              return caches.delete(key);
            }
          })
        );
      })
      .then(() => self.clients.claim())
  );
});

self.addEventListener("fetch", function (evt) {
  evt.respondWith(
    fetch(evt.request).catch(() => {
      return caches.open(CATALOGUE_ASSETS).then((cache) => {
        return cache.match(evt.request);
      });
    })
  );
});
```

#### ✅ Checkpoint: PWA Features Complete

```bash
# Verify PWA implementation:
ls static/
# Should show: manifest.json, css/, js/, icons/, images/
ls static/js/
# Should show: app.js, serviceworker.js
# Open browser DevTools → Application → Service Workers
# Should show registered service worker
```

> **PWA not working?** See [🔧 Troubleshooting - PWA and Service Worker Issues](#-troubleshooting-common-issues)

---

### Validate your PWA

Validation is important to ensure the app is compliant with [W3 web standards](https://www.w3.org/standards/).

1. Open your website in Chrome, open developer tools (F12), and run a Lighthouse report.

![Screen cpature of Chrome Lighthouse report](/docs/README_resources/Chrome_Lighthouse_Report.png "Click F12 and choose Lighthouse on the top menu of your developer tools")

. 2. Open your website in Edge, open developer tools (F12), and look at the application report.

![Screen cpature of Chrome Lighthouse report](/docs/README_resources/Edge_Application_Report.png "Click F12 and choose Lighthouse on the top menu of your developer tools")

.

#### ✅ Checkpoint: PWA Validation Complete

```bash
# Final verification:
# 1. Open Chrome → DevTools → Lighthouse → Run audit
# 2. PWA score should be 90+ (green)
# 3. Performance should be acceptable
# 4. Test offline: DevTools → Network → Offline → Reload page
```

> **Validation failing?** See [🔧 Troubleshooting - PWA and Service Worker Issues](#-troubleshooting-common-issues)

---

### Take your app further

The following code snippets will help you create a simple form on the add.html page. This form allows people to add their details to an email database for updates on your catalogue. Less explicit instructions have been provided; students are encouraged to practice their BASH, SQL Flask, and HTML to bring it all together. The screenshot below shows what the page should look like, and when users submit, the database is updated.

![Screen capture of the finished PWA](/docs/README_resources/form_example.png "This is what your application will look like")

.

1. Page specifications:
   - Simple form where the user inserts their name and email address
   - When they click submit, the database is updated
   - The input form must be styled to be consistent with the rest of the website
   - A message confirming submission is returned to the user
2. SQL schema specifications:

   - A new table called contact_list
   - 3 columns

     - `id` - Primary key that increments automatically (SQLite handles this)
     - `email` - User's email address (must be unique to prevent duplicates)
     - `name` - User's display name

**Understanding the Contact Form SQL:**

```sql
-- You'll need to create this table first:
CREATE TABLE contact_list (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT UNIQUE NOT NULL,
    name TEXT NOT NULL
);
```

```python
def insertContact(email,name):
  con = sql.connect("database/data_source.db")
  cur = con.cursor()
  # The ? placeholders prevent SQL injection attacks
  # Values are safely inserted in the order: (email, name)
  cur.execute("INSERT INTO contact_list (email,name) VALUES (?,?)", (email,name))
  con.commit()
  con.close()
```

> [!Note]
> You will need to catch the expectation of a duplicate email

**Understanding Form Handling with HTTP Methods:**
This route handles both GET and POST requests differently:

- **GET request**: User visits `/add.html` → Show the empty form
- **POST request**: User submits form → Process data and show confirmation

```python
@app.route('/add.html', methods=['POST', 'GET'])
def add():
   if request.method=='POST':              # Form was submitted
      email = request.form['email']        # Get data from form fields
      name = request.form['name']
      dbHandler.insertContact(email,name)  # Save to database
      return render_template('/add.html', is_done=True)  # Show success message
   else:                                   # GET request - just viewing
      return render_template('/add.html')  # Show empty form
```

**Template Engine Conditional Syntax:**

- `{% if condition %}` - Show content if condition is true
- `{% else %}` - Alternative content
- `{% endif %}` - End the condition block

```html
{% if is_done %} <--DO THIS--> {% else %} <--DO THIS--> {% endif %}
```

**Understanding HTML Form Methods:**
The `method="POST"` attribute tells the browser to send form data using POST request:

- **POST**: Secure for sending form data (email, passwords, personal info)
- **GET**: Data appears in URL - only use for searches, not sensitive data
- **action="/add.html"**: Where to send the form data when submitted

```html
<form action="/add.html" method="POST" class="box">
  <div>
    <label class="form-label">Email address</label>
    <input
      name="email"
      type="email"
      class="form-control"
      id="email"
      pattern="[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2, 4}$"
      placeholder="name@example.com"
    />
  </div>
  <div>
    <label class="form-label">Name</label>
    <textarea class="form-control" name="name" id="name" rows="1"></textarea>
  </div>
  <br />
  <div>
    <button type="submit" class="btn">Submit</button>
  </div>
</form>
```

```css
.form-control {
}
```

> **Need help with forms or advanced features?** See [🔧 Troubleshooting Common Issues](#-troubleshooting-common-issues) below.

---

## 🔧 Troubleshooting Common Issues

### Environment Setup Issues

#### **Problem: Flask not found**

**Error**: `ModuleNotFoundError: No module named 'flask'`
**Solution**:

```bash
pip install flask
# or try:
pip3 install flask
```

#### **Problem: Python command not found**

**Error**: `python: command not found`
**Solution**:

```bash
# Try these alternatives:
python3 main.py
py main.py
```

#### **Problem: Permission denied errors**

**Solution**:

```bash
# Check file permissions:
ls -la
# Make files executable if needed:
chmod +x main.py
```

### Flask Application Issues

#### **Problem: Flask app won't start**

**Error**: Various import or syntax errors
**Solution**:

1. Check you're in the correct directory: `pwd`
2. Verify files exist: `ls -la`
3. Check Python syntax in main.py
4. Ensure all imports are correct

#### **Problem: "Address already in use"**

**Error**: `OSError: [Errno 48] Address already in use`
**Solution**:

```bash
# Kill processes using port 5000:
lsof -ti:5000 | xargs kill -9
# Or use a different port in main.py:
app.run(debug=True, host='0.0.0.0', port=5001)
```

#### **Problem: Template not found**

**Error**: `jinja2.exceptions.TemplateNotFound: index.html`
**Solution**:

1. Check templates folder exists and contains files
2. Verify file names match exactly (case-sensitive)
3. Check Flask knows where templates are located

### Database Issues

#### **Problem: Database file not found**

**Error**: `sqlite3.OperationalError: no such table: extension`
**Solution**:

1. Check database file exists: `ls database/`
2. Recreate table with CREATE TABLE command
3. Verify database path in database_manager.py is correct

#### **Problem: SQL syntax errors**

**Error**: `sqlite3.OperationalError: near "...": syntax error`
**Solution**:

1. Check for typos in SQL commands
2. Ensure proper quotes around text values
3. Verify column names match exactly
4. Check for missing commas or parentheses

#### **Problem: Duplicate entry errors**

**Error**: `sqlite3.IntegrityError: UNIQUE constraint failed`
**Solution**:

```python
# Add error handling:
try:
    dbHandler.insertContact(email, name)
except sqlite3.IntegrityError:
    return render_template('/add.html', error="Email already exists")
```

### PWA and Service Worker Issues

#### **Problem: Service worker not registering**

**Error**: Service worker not showing in DevTools
**Solution**:

1. Check file exists: `ls static/js/serviceworker.js`
2. Verify correct path in app.js registration
3. Use HTTPS or localhost (required for service workers)
4. Check browser console for errors

#### **Problem: App not installable**

**Solution**:

1. Verify manifest.json is valid JSON
2. Check all required manifest fields are present
3. Ensure icons exist in specified paths
4. Test on HTTPS or localhost
5. Check Lighthouse PWA audit for issues

#### **Problem: Offline functionality not working**

**Solution**:

1. Verify service worker is active in DevTools
2. Check cache storage contains expected files
3. Test offline mode in DevTools → Network → Offline
4. Ensure all URLs in serviceworker.js are correct

### Browser and Testing Issues

#### **Problem: CSS/JS changes not appearing**

**Solution**:

1. Hard refresh: `Ctrl+F5` (Windows) or `Cmd+Shift+R` (Mac)
2. Clear browser cache
3. Check browser console for 404 errors
4. Verify file paths are correct

#### **Problem: Form data not submitting**

**Solution**:

1. Check form method is POST
2. Verify input names match Flask route
3. Check browser console for JavaScript errors
4. Ensure Flask route accepts POST method

#### **Problem: Images not loading**

**Solution**:

1. Check file paths are correct
2. Verify images exist in static/images/
3. Check file permissions
4. Ensure image files are not corrupted

### Development Workflow Issues

#### **Problem: Git/Version control errors**

**Solution**:

```bash
# Check git status:
git status
# Reset if needed:
git reset --hard HEAD
# Pull latest changes:
git pull origin main
```

#### **Problem: VSCode extensions not working**

**Solution**:

1. Reload VSCode: `Ctrl+Shift+P` → "Developer: Reload Window"
2. Check extensions are installed and enabled
3. Verify workspace settings in .vscode/settings.json

### Quick Diagnostic Commands

When encountering issues, run these commands to gather information:

```bash
# Check Python version
python --version
python3 --version

# Check Flask installation
pip show flask

# Check current directory and files
pwd
ls -la

# Check if Flask app runs
python main.py

# Test if server responds
curl -I http://localhost:5000

# Check database
sqlite3 database/data_source.db ".tables"

# Check for running processes on port 5000
lsof -i:5000
```

### Getting Help

If you're still stuck after trying these solutions:

1. Check the error message carefully - it often tells you exactly what's wrong
2. Use browser DevTools (F12) to check Console and Network tabs
3. Ask a classmate or teacher for help
4. Search for the specific error message online

---

<p xmlns:cc="http://creativecommons.org/ns#" xmlns:dct="http://purl.org/dc/terms/"><a property="dct:title" rel="cc:attributionURL" href="https://github.com/TempeHS/Flask_PWA_Programming_For_The_Web_Task_Source">Flask PWA Programming For The Web Task Source</a> and <a property="dct:title" rel="cc:attributionURL" href="https://github.com/TempeHS/Flask_PWA_Programming_For_The_Web_Task_Template">Flask PWA Programming For The Web Task Template</a> by <a rel="cc:attributionURL dct:creator" property="cc:attributionName" href="https://github.com/benpaddlejones">Ben Jones</a> is licensed under <a href="https://creativecommons.org/licenses/by-nc-sa/4.0/?ref=chooser-v1" target="_blank" rel="license noopener noreferrer" style="display:inline-block; ">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International<img style="height:22px!important; margin-left:3px; vertical-align:text-bottom; " src="https://mirrors.creativecommons.org/presskit/icons/cc.svg?ref=chooser-v1" alt=""><img style="height:22px!important; margin-left:3px; vertical-align:text-bottom; " src="https://mirrors.creativecommons.org/presskit/icons/by.svg?ref=chooser-v1" alt=""><img style="height:22px!important; margin-left:3px; vertical-align:text-bottom; " src="https://mirrors.creativecommons.org/presskit/icons/nc.svg?ref=chooser-v1" alt=""><img style="height:22px!important; margin-left:3px; vertical-align:text-bottom; " src="https://mirrors.creativecommons.org/presskit/icons/sa.svg?ref=chooser-v1" alt=""><img style="height:22px!important; margin-left:3px; vertical-align:text-bottom; " src="https://mirrors.creativecommons.org/presskit/icons/sa.svg?ref=chooser-v1" alt=""></a></p>
