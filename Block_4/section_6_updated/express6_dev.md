# Database interactions in Express

Completing the express local library app to include create, retrieve update and delete.  This continues to follow the [tutorial](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/forms)

## Duplicate repository

Create a new local folder express--six and copy the folders, docker and myapp across together with mongo-init.js from express5 ready to start the next stage.

Start by creating a new public github repository express---6 with a node .gitignore file.

Create the local bare clone copy working in powershell;

> git clone --bare https://github.com/derekTurner/express--5.git

Move to the now local copy and Mirror push the thew repository

> cd express--5.git

> git push --mirror https://github.com/derekTurner/express--six.git

Then remove the local repository

> cd..

> rm -rf express--5.git

If necessary, manually remove the local copy of express--5.git which is probably in your users directory.

In order to make all the container names unique there are a few updates needed whilst editing in github.

**compose.yaml**
```yaml
# Use root/example as user/password credentials
version: "3.8"

services:
  server:
    entrypoint:
    - sleep
    - infinity
    image: node:lts-bullseye 
    init: true
    volumes:
    - type: bind
      source: /var/run/docker.sock
      target: /var/run/docker.sock
    # build:
    #  context: myapp
    restart: always
    
    container_name: server6
    ports:
    - 3000:3000
    environment:
      MONGODB_CONNSTRING: mongodb://root:example@mongodb:27017
    depends_on:
      - mongodb
    networks: 
      - mongo1_network

  mongodb:
    image: mongo:7.0-rc-jammy
    restart: always
    container_name: mongodb6
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
      MONGO_INITDB_DATABASE: local_library
    volumes:
    - ./mongo-init.js:/docker-entrypoint-initdb.d/mongo-init.js:ro
    networks: 
      - mongo1_network
    ports: 
      - 27017:27017

  mongo-express:
    image: mongo-express:latest
    restart: always
    container_name: mongo-express6
    ports:
      - 8081:8081
    environment:
      ME_CONFIG_BASICAUTH_USERNAME: root          #| ''              | mongo-express web username
      ME_CONFIG_BASICAUTH_PASSWORD: example       #| ''              | mongo-express web password
      ME_CONFIG_MONGODB_ENABLE_ADMIN: true        #| 'true'          | Enable admin access to all databases. Send strings: `"true"` or `"false"`
      ME_CONFIG_MONGODB_ADMINUSERNAME: root       #| ''              | MongoDB admin username
      ME_CONFIG_MONGODB_ADMINPASSWORD: example    #| ''              | MongoDB admin password
      ME_CONFIG_MONGODB_PORT: 27017               #| 27017           | MongoDB port
      ME_CONFIG_MONGODB_SERVER: mongodb           #| 'mongo'         | MongoDB container name. Use comma delimited list of host names for replica sets.
      ME_CONFIG_OPTIONS_EDITORTHEME: default      #| 'default'       | mongo-express editor color theme, [more here](http://codemirror.net/demo/theme.html)
      ME_CONFIG_REQUEST_SIZE: 100kb               #| '100kb'         | Maximum payload size. CRUD operations above this size will fail in [body-parser](https://www.npmjs.com/package/body-parser).
      ME_CONFIG_SITE_BASEURL: /                   #| '/'             | Set the baseUrl to ease mounting at a subdirectory. Remember to include a leading and trailing slash.
      ME_CONFIG_SITE_COOKIESECRET: cookiesecret   #| 'cookiesecret'  | String used by [cookie-parser middleware](https://www.npmjs.com/package/cookie-parser) to sign cookies.
      ME_CONFIG_SITE_SESSIONSECRET: sessionsecret #| 'sessionsecret' | String used to sign the session ID cookie by [express-session middleware](https://www.npmjs.com/package/express-session).
      ME_CONFIG_SITE_SSL_ENABLED: false           #| 'false'         | Enable SSL.
      ME_CONFIG_SITE_SSL_CRT_PATH:                #| ''              | SSL certificate file.
      ME_CONFIG_SITE_SSL_KEY_PATH:                #| ''              | SSL key file.

      MONGODB_CONNSTRING: mongodb://root:example@mongodb:27017/

    networks: 
      - mongo1_network 
  
    depends_on:
      - mongodb

networks:
  mongo1_network:
    driver: bridge
```

Create a new environment.

![create express--6](createExpress6.png)

![express--6](runningexpress--6.png)

## [Genre details](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/Displaying_data/Genre_detail_page)

Open server6 in VSC and set password to 'node'

>passwd

```code
New password: 
Retype new password: 
passwd: password updated successfully
```

>cd myapp

>su node

>npm install

```code
added 193 packages, and audited 194 packages in 6s

21 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
npm notice 
npm notice New patch version of npm available! 10.2.3 -> 10.2.4
npm notice Changelog: https://github.com/npm/cli/releases/tag/v10.2.4
npm notice Run npm install -g npm@10.2.4 to update!
```

Slip back to root and update:

>su root

>npm  npm install -g npm@10.2.4

```code
removed 10 packages, and changed 30 packages in 10s

28 packages are looking for funding
  run `npm fund` for details
```

>su node

>npm run start

```code
 nodemon ./bin/www

[nodemon] 2.0.20
[nodemon] reading config ./package.json
[nodemon] to restart at any time, enter `rs`
[nodemon] or send SIGHUP to 3812 to restart
[nodemon] watching path(s): *.*
[nodemon] watching extensions: js,mjs,json
[nodemon] starting `node ./bin/www`
[nodemon] forking
[nodemon] child pid: 3825
[nodemon] watching 24 files
Running on http://127.0.0.1:3000
```

Check operation matches endpoint of express--5

In the browser:

>http://127.0.0.1:3000/catalog

![catalog check](catalogcheck.png)

## Search by id

So far documents have been retrieved for full collections, now the query must be refined to allow searching for one item by id.

The next step should use the route in **routes/catalog.js**:

```javascript
// GET request for one Genre.
router.get('/genre/:id', genre_controller.genre_detail);
```

This should call the controller genre_detail to check that the genre id is correct and to return a list of all books with that genre id.

Open **controllers/genreController.js** and add the code to import the Book and asyncHandler modules at the top.

```javascript
const Genre = require("../models/genre");
var Book = require('../models/book');
const asyncHandler = require("express-async-handler");
```

Replace

```javascript
// Display detail page for a specific Genre.
exports.genre_detail = function(req, res) {
    res.send('NOT IMPLEMENTED: Genre detail: ' + req.params.id);
};
```

with:

```javascript
// Display detail page for a specific Genre.
exports.genre_detail = asyncHandler(async (req, res, next) => {
  // Get details of genre and all associated books (in parallel)
  const [genre, booksInGenre] = await Promise.all([
    Genre.findById(req.params.id).exec(),
    Book.find({ genre: req.params.id }, "title summary").exec(),
  ]);
  if (genre === null) {
    // No results.
    const err = new Error("Genre not found");
    err.status = 404;
    return next(err);
  }

  res.render("genre_detail", {
    title: "Genre Detail",
    genre: genre,
    genre_books: booksInGenre,
  });
});

```

The controller makes two queries using asyncHandler.  The first looks to find the genre corresponding to the id, thereby checking it exists.  If this does not exist we will report an error status 404 and message 'Genre not found'.

The second query searches for books whose genres matches the genre id.

Only if both queries are succesful will a list of titles be printed.

Create **views/genre_detail.pug** to display this list with content:

```pug
extends layout

block content

  h1 Genre: #{genre.name}

  div(style='margin-left:20px;margin-top:20px')

    h4 Books

    dl
      each book in genre_books
        dt
          a(href=book.url) #{book.title}
        dd #{book.summary}

      else
        p This genre has no books

```

Follow the all genres link from the catalog page or 

> http://localhost:3000/catalog/genres

![Genre List](genreList.png)

click on the Fantasy list or

> http://127.0.0.1:3000/catalog/genre/637c1e219c9262f298e3af81

![Fantasy Books](fantasyBooks.png)

## [Book details](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/Displaying_data/Book_detail_page)

The next query should use the route in **catalog.js**

```javascript
// GET request for one Book.
router.get('/book/:id', book_controller.book_detail);
```

In the file controllers/bookController.js replace:

```javascript
// Display detail page for a specific book.
exports.book_detail = function(req, res) {
    res.send('NOT IMPLEMENTED: Book detail: ' + req.params.id);
};
```
by the code:

```javascript
// Display detail page for a specific book.
exports.book_detail = asyncHandler(async (req, res, next) => {
  // Get details of books, book instances for specific book
  const [book, bookInstances] = await Promise.all([
    Book.findById(req.params.id).populate("author").populate("genre").exec(),
    BookInstance.find({ book: req.params.id }).exec(),
  ]);

  if (book === null) {
    // No results.
    const err = new Error("Book not found");
    err.status = 404;
    return next(err);
  }

  res.render("book_detail", {
    title: book.title,
    book: book,
    book_instances: bookInstances,
  });
});

```

This will asynchronously find a book by id and list its instances.

If the book is not found and error status of 404 is found and the message "Book not found" is raised.

If the book is found and instances exist they are listed.   The details of the book are populated from the author and genre tables.

Creat a new file views/book_detail.pug with the contents:

```pug
extends layout

block content
  h1 Title: #{book.title}

  p #[strong Author:]
    a(href=book.author.url) #{book.author.name}
  p #[strong Summary:] #{book.summary}
  p #[strong ISBN:] #{book.isbn}
  p #[strong Genre:]
    each val, index in book.genre
      a(href=val.url) #{val.name}
      if index < book.genre.length - 1
        |,

  div(style='margin-left:20px;margin-top:20px')
    h4 Copies

    each val in book_instances
      hr
      if val.status=='Available'
        p.text-success #{val.status}
      else if val.status=='Maintenance'
        p.text-danger #{val.status}
      else
        p.text-warning #{val.status}
      p #[strong Imprint:] #{val.imprint}
      if val.status!='Available'
        p #[strong Due back:] #{val.due_back}
      p #[strong Id:]
        a(href=val.url) #{val._id}

    else
      p There are no copies of this book in the library.

```

The lines: 

```javascript
if index < book.genre.length - 1
        |, 
```

add a comma after each instance except the last.

follow the all books link or

> http://localhost:3000/catalog/books

![book list](bookList.png)

Choose Apes and Angels or

> localhost:3000/catalog/book/637c1e219c9262f298e3af87

![book detail](bookDetail.png)

## [Author detail](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/Displaying_data/Author_detail_page)

The next querey allows listing of all the books by an author identified by id.

Open controllers/authorController.js and add the book model and async module.

```javascript
var Author = require('../models/author');
var Book = require('../models/book');
const asyncHandler = require("express-async-handler");
```

Then replace

```javascript
// Display detail page for a specific Author.
exports.author_detail = function(req, res) {
    res.send('NOT IMPLEMENTED: Author detail: ' + req.params.id);
};
```

with code:

```javascript
// Display detail page for a specific Author.
exports.author_detail = asyncHandler(async (req, res, next) => {
  // Get details of author and all their books (in parallel)
  const [author, allBooksByAuthor] = await Promise.all([
    Author.findById(req.params.id).exec(),
    Book.find({ author: req.params.id }, "title summary").exec(),
  ]);

  if (author === null) {
    // No results.
    const err = new Error("Author not found");
    err.status = 404;
    return next(err);
  }

  res.render("author_detail", {
    title: "Author Detail",
    author: author,
    author_books: allBooksByAuthor,
  });
});
```

This checked that the author exists, and sends an error message "Author not found" if it does not.

If authors are foud they are listed by adding a page at **views/author_detail.pug**

```javascript
extends layout

block content

  h1 Author: #{author.name}
  p #{author.date_of_birth} - #{author.date_of_death}

  div(style='margin-left:20px;margin-top:20px')

    h4 Books

    dl
      each book in author_books
        dt
          a(href=book.url) #{book.title}
        dd #{book.summary}

      else
        p This author has no books.


```

Follow the all authors link or

> http://localhost:3000/catalog/authors

![author list](authorList.png)

Following the link for Ben Bova:

> http://localhost:3000/catalog/author/637c1e219c9262f298e3af7d

![author detail](authorDetails.png)


## [Bookinstance detail](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/Displaying_data/BookInstance_detail_page_and_challenge)

Open controllers/bookinstanceController.js and replace:

```javascript
// Display detail page for a specific BookInstance.
exports.bookinstance_detail = function(req, res) {
    res.send('NOT IMPLEMENTED: BookInstance detail: ' + req.params.id);
};
```

with code:

```javascript
// Display detail page for a specific BookInstance.
exports.bookinstance_detail = asyncHandler(async (req, res, next) => {
  const bookInstance = await BookInstance.findById(req.params.id)
    .populate("book")
    .exec();

  if (bookInstance === null) {
    // No results.
    const err = new Error("Book copy not found");
    err.status = 404;
    return next(err);
  }

  res.render("bookinstance_detail", {
    title: "Book:",
    bookinstance: bookInstance,
  });
});

```

This is a straightforward query which does not need asyncHandler.

Create a view for this in **views/bookinstance_detail.pug**

```javascript
extends layout

block content

  h1 ID: #{bookinstance._id}

  p #[strong Title:]
    a(href=bookinstance.book.url) #{bookinstance.book.title}
  p #[strong Imprint:] #{bookinstance.imprint}

  p #[strong Status:]
    if bookinstance.status=='Available'
      span.text-success #{bookinstance.status}
    else if bookinstance.status=='Maintenance'
      span.text-danger #{bookinstance.status}
    else
      span.text-warning #{bookinstance.status}

  if bookinstance.status!='Available'
    p #[strong Due back:] #{bookinstance.due_back_formatted}

```

View the 'All book-instances' link or

> http://localhost:3000/catalog/bookinstances

![bookinstance list](bookinstanceList.png)

Follow the link to 'The wise mans fear' or

> http://localhost:3000/catalog/bookinstance/637c1e219c9262f298e3af8c

![bookinstance detail](bookinstanceDetail.png)

## Tidying up date format

In the Author model file author.js a virtual lifespan property has been defined.  Modify this so that the lifespan will be displayed in years.  If the author is still living then let's say so, the lifespan can't be calculated as there is no data of death.  If the date of birth of death are to be displayed on the auther details then they should be formatted.  In this case author.js should require moment.

 Update **models/author js** 

```javascript
var mongoose = require('mongoose');
const { DateTime } = require ('luxon');

var Schema = mongoose.Schema;

var AuthorSchema = new Schema(
  {
    first_name: {type: String, required: true, max: 100},
    family_name: {type: String, required: true, max: 100},
    date_of_birth: {type: Date},
    date_of_death: {type: Date},
  }
);

// Virtual for author's full name
AuthorSchema
.virtual('name')
.get(function () {
  return this.family_name + ', ' + this.first_name;
});

AuthorSchema
.virtual('date_of_birth_formatted')
.get(function () {
  return this.date_of_birth ? DateTime.fromJSDate(this.date_of_birth).toLocaleString(DateTime.DATE_MED) : '';
});

AuthorSchema
.virtual('date_of_death_formatted')
.get(function () {
 if (this.date_of_death){
  return this.date_of_birth ? DateTime.fromJSDate(this.date_of_death).toLocaleString(DateTime.DATE_MED) : '';
 }else{
    return "living";
  }  
});

// Virtual for author's lifespan
AuthorSchema
.virtual('lifespan')
.get(function () {
  if(this.date_of_death){
  return "lifespan:" + (this.date_of_death.getYear() - this.date_of_birth.getYear()).toString() + " years";
  } else {
    return "living"; 
  }
});


// Virtual for author's URL
AuthorSchema
.virtual('url')
.get(function () {
  return '/catalog/author/' + this._id;
});

//Export model
module.exports = mongoose.model('Author', AuthorSchema);
```
Modify **views/author_list.pug** to display the lifespan.

```pug
extends layout

block content
  h1= title
  
  ul
    each author in author_list
      li 
        a(href=author.url) #{author.name} 
        |  (#{author.lifespan})

    else
      li There are no authors.
```      

![author lifespan](authorlifespan.png)

Modify **author_detail.pug** to display the formatted dates. (If the date of birth is missing in the database it will be displayed as and invalid date).

```pug
extends layout

block content

  h1 Author: #{author.name}
  p #{author.date_of_birth_formatted} - #{author.date_of_death_formatted}
  
  div(style='margin-left:20px;margin-top:20px')

    h4 Books
    
    dl
      each book in author_books
        dt 
          a(href=book.url) #{book.title}
        dd #{book.summary}

      else
        p This author has no books.
```

Recheck the author details in the browser.

![author lifespan](authorDetailsDated.png)



## Modifying data using HTML forms

The browser will request a page containing a form.  The web server will create and return a page with an empty form.

The user populates the form with data.  The app then validates this data.  If the data is invalid a new form is generated with all the correct user generated content.  If the data is valid appropriate actions are carrid out using the valid data (the database is modified), the browser is redefined to a 'success message'.

The blank form will be sent in response to an HTTP GET request and the data will be validated (and processed) in response to an HTTP POST request.

Express needs to use middleware to process POST and GET parameters from forms and to validate values.

**Validation**, checks that values for each field match the required range and format.

**Sanitization**, removes or replaces characters which might represent malicious content.

[Express validator](https://www.npmjs.com/package/express-validator) will be used to validate and sanitize the forms.  Add this to package.json.  

> npm install express-validator

**Package.json** becomes:

```json
{
  "name": "myapp",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "start": "nodemon ./bin/www"
  },
  "dependencies": {
    "body-parser": "^1.20.2",
    "cookie-parser": "~1.4.4",
    "core-js": "^3.33.2",
    "debug": "~2.6.9",
    "express": "^4.18.2",
    "express-async-handler": "^1.2.0",
    "express-validator": "^7.0.1",
    "http-errors": "~1.6.3",
    "luxon": "^3.4.4",
    "mongoose": "^8.0.0",
    "morgan": "~1.9.1",
    "nodemon": "^3.0.1",
    "pug": "^3.0.2"
  },
  "nodemonConfig": {
    "delay": "1500",
    "verbose": "true"
  }
}

```


Instructions for [express-validator](https://express-validator.github.io/docs/) are found on github.

A quick summary of relevant validation is included in the [tutorial](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/forms).

To maintain a simpler approach to form design two rules will be applied.

1. You can only create new objects if their component objects (such as author and genre) already exist.

2. You can only delete an object (e.g. Book) if it is not referenced by any other objects (e.g. bookInstances).

Routes will be formed in pairs for identical requests to GET and POST.

## [Create Genre form](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/forms/Create_genre_form)

Require the express validator at the top of the **controllers/genreController.js** file.

```javascript
const Genre = require("../models/genre");
var Book = require('../models/book');
const asyncHandler = require("express-async-handler");
const { body, validationResult } = require("express-validator");
```

In **controllers/genreControllers.js**  replace:

```javascript
// Display Genre create form on GET.
exports.genre_create_get = function(req, res) {
    res.send('NOT IMPLEMENTED: Genre create GET');
};
```

with code:

```javascript
// Display Genre create form on GET.
exports.genre_create_get = (req, res, next) => {
  res.render("genre_form", { title: "Create Genre" });
};
```

This will render the genre_form pug view  passing a variable for the title.

Then, also in **controllers/genreControllers.js** replace the controller section for genre_create_post:

```javascript
// Handle Genre create on POST.
exports.genre_create_post = function(req, res) {
    res.send('NOT IMPLEMENTED: Genre create POST');
};
```

with:

```javascript
// Handle Genre create on POST.
exports.genre_create_post = [
  // Validate and sanitize the name field.
  body("name", "Genre name must contain at least 3 characters")
    .trim()
    .isLength({ min: 3 })
    .escape(),

  // Process request after validation and sanitization.
  asyncHandler(async (req, res, next) => {
    // Extract the validation errors from a request.
    const errors = validationResult(req);

    // Create a genre object with escaped and trimmed data.
    const genre = new Genre({ name: req.body.name });

    if (!errors.isEmpty()) {
      // There are errors. Render the form again with sanitized values/error messages.
      res.render("genre_form", {
        title: "Create Genre",
        genre: genre,
        errors: errors.array(),
      });
      return;
    } else {
      // Data from form is valid.
      // Check if Genre with same name already exists.
      const genreExists = await Genre.findOne({ name: req.body.name }).exec();
      if (genreExists) {
        // Genre exists, redirect to its detail page.
        res.redirect(genreExists.url);
      } else {
        await genre.save();
        // New genre saved. Redirect to genre detail page.
        res.redirect(genre.url);
      }
    }
  }),
];

```

This is not a single function but an array of functions.

Validator checks that the name field is not empty and trims any blank space.

Processing the request uses isEmpty to check for errors from validation and sanitization

If there are errors then render the empty page again.

If no errors then check if genre already exists.  If it does, go to the details page.

If genre does not exist it is saved and then the details page is displayed.

This will be a general pattern.

A single view is created in **views/genre_form.pug** the GET query will lead to a title and and empty form, in the POST query the validated and sanitized data is displayed.

```javascript
extends layout

block content

  h1 #{title}

  form(method='POST' action='')
    div.form-group
      label(for='name') Genre:
      input#name.form-control(type='text', placeholder='Fantasy, Poetry etc.' name='name' required='true' value=(undefined===genre ? '' : genre.name) )
    button.btn.btn-primary(type='submit') Submit

  if errors
    ul
      for error in errors
        li!= error.msg
```

The form has a single field of type text called name.

From the home page follow the Create new genre link or

> http://127.0.0.1:3000/catalog/genre/create

![Create Genre](genreCreate.png)

Try to submit  a blank entry, the genre name required prompt will be added.

![Create Genre](genreCreatePrompt.png)

Try to submit an existing genre, "Fantasy", the list of fantasy books is displayed.

![Create Genre](genreFantasy.png)

Submit a new Genre Biography

![Create Genre](genreBiography.png)

There are no books in the Biography genre at the moment.



## [Create Author](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/forms/Create_author_form)

The pattern of working in creating an author is similar to genre create.

1. Use 'required' to import the functions needed to the controller file.
2. In the controller respond to a GET request by displaying the form using a pug template with a title.
3. In the controller respond to a POST request   
    1. Validate fields
    2. Sanitize fields
    3. Process the request after validation and sanitization   
        a. Extract validation errors from request  
        b. If there are errors render form again with error messages  
        c. If data is valid 
            i. create a json object with trimmed data  
            ii. save object to database
            iii. redirect to page to list of records
4.  Create a pug template which matches the scheme of the database collection targetted.

Add the required validation and sanitization functions to **controllers/authorController.js**

```javascript
var Author = require('../models/author');
var Book = require('../models/book');
const asyncHandler = require("express-async-handler");
const { body, validationResult } = require("express-validator");
```

In **controllers/authorController.js**  replace code to handle author create GET request.

```javascript
// Display Author create form on GET.
exports.author_create_get = function(req, res) {
    res.send('NOT IMPLEMENTED: Author create GET');
};
```
update with code to render the author create form using a pug template passing the title "Create Author":

```javascript
// Display Author create form on GET.
exports.author_create_get = (req, res, next) => {
  res.render("author_form", { title: "Create Author" });
};
```

Also in **controllers/authorController.js**,  replace code to handle author create POST request.

```javascript
// Handle Author create on POST.
exports.author_create_post = function(req, res) {
    res.send('NOT IMPLEMENTED: Author create POST');
};
```
with code to validate sanitize and process results.

```javascript
// Handle Author create on POST.
exports.author_create_post = [
  // Validate and sanitize fields.
  body("first_name")
    .trim()
    .isLength({ min: 1 })
    .escape()
    .withMessage("First name must be specified.")
    .isAlphanumeric()
    .withMessage("First name has non-alphanumeric characters."),
  body("family_name")
    .trim()
    .isLength({ min: 1 })
    .escape()
    .withMessage("Family name must be specified.")
    .isAlphanumeric()
    .withMessage("Family name has non-alphanumeric characters."),
  body("date_of_birth", "Invalid date of birth")
    .optional({ values: "falsy" })
    .isISO8601()
    .toDate(),
  body("date_of_death", "Invalid date of death")
    .optional({ values: "falsy" })
    .isISO8601()
    .toDate(),

  // Process request after validation and sanitization.
  asyncHandler(async (req, res, next) => {
    // Extract the validation errors from a request.
    const errors = validationResult(req);

    // Create Author object with escaped and trimmed data
    const author = new Author({
      first_name: req.body.first_name,
      family_name: req.body.family_name,
      date_of_birth: req.body.date_of_birth,
      date_of_death: req.body.date_of_death,
    });

    if (!errors.isEmpty()) {
      // There are errors. Render form again with sanitized values/errors messages.
      res.render("author_form", {
        title: "Create Author",
        author: author,
        errors: errors.array(),
      });
      return;
    } else {
      // Data from form is valid.

      // Save author.
      await author.save();
      // Redirect to new author record.
      res.redirect(author.url);
    }
  }),
];
```

This code has not attempted to check if the author already exists so it is allowed to have multiple authors with the same name.

Create a pug template for the author create form in **views/author_form.pug**

```pug
extends layout

block content
  h1=title

  form(method='POST' action='')
    div.form-group
      label(for='first_name') First Name:
      input#first_name.form-control(type='text' placeholder='First name' name='first_name' required='true' value=(undefined===author ? '' : author.first_name) )
      label(for='family_name') Family Name:
      input#family_name.form-control(type='text' placeholder='Family name' name='family_name' required='true' value=(undefined===author ? '' : author.family_name))
    div.form-group
      label(for='date_of_birth') Date of birth:
      input#date_of_birth.form-control(type='date' name='date_of_birth' value=(undefined===author ? '' : author.date_of_birth) )
    button.btn.btn-primary(type='submit') Submit
  if errors
    ul
      for error in errors
        li!= error.msg

```


 Follow link to create new author or

 > http://127.0.0.1:3000/catalog/author/create

![Create Author](authorCreate.png)
 
 Submit details

 ![enter details](enterAuthorDetails.png)

The Author entry is created

![Created author](authorCreated.png)



## [Create Book](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/forms/Create_book_form)

The pattern of working in creating a book entry is similar to genre and author create, but will need to read author and genre details from the database to supply the appropriate id fields to the JSON object before the database entry is made.

1. Use 'required' to import the functions needed to the controller file.
2. In the controller respond to a GET request by displaying the form using a pug template with a title.  
    1. Include details of all available authors and genres into the form using async.
3. In the controller respond to a POST request   
    1. Validate fields
    2. Sanitize fields
    3. Process the request after validation and sanitization   
        a. Extract validation errors from request  
        b. If there are errors render form again with error messages  
        c. If data is valid 
            i. create a json object with trimmed data  
            ii. save object to database
            iii. redirect to page to list of records
4.  Create a pug template which matches the scheme of the database collection targetted.

In **controllers/bookController.js** add the functions needed by requiring them.

```javascript
var Book = require("../models/book");
var Author = require("../models/author");
var Genre = require("../models/genre");
var BookInstance = require("../models/bookinstance");

const asyncHandler = require("express-async-handler");
const { body, validationResult } = require("express-validator");
```

In **controllers/bookController.js** replace the code for book_create_get.

```javascript
// Display book create form on GET.
exports.book_create_get = function(req, res) {
    res.send('NOT IMPLEMENTED: Book create GET');
};
```
with code to display the book form through a put template.

```javascript
// Display book create form on GET.
exports.book_create_get = asyncHandler(async (req, res, next) => {
  // Get all authors and genres, which we can use for adding to our book.
  const [allAuthors, allGenres] = await Promise.all([
    Author.find().sort({ family_name: 1 }).exec(),
    Genre.find().sort({ name: 1 }).exec(),
  ]);

  res.render("book_form", {
    title: "Create Book",
    authors: allAuthors,
    genres: allGenres,
  });
});

```
If there are no errors, the details which are passed on to the book_form include full author and genre content.

Also in **controllers/bookController.js**, replace the cotroller entry for a POST query to validate, sanitize and process a completed form.  Replace:

```javascript
// Handle book create on POST.
exports.book_create_post = function(req, res) {
    res.send('NOT IMPLEMENTED: Book create POST');
};
```

with code:

```javascript
// Handle book create on POST.
exports.book_create_post = [
    // Convert the genre to an array.
    (req, res, next) => {
        if(!(req.body.genre instanceof Array)){
            if(typeof req.body.genre==='undefined')
            req.body.genre=[];
            else
            req.body.genre=new Array(req.body.genre);
        }
        next();
    },

    // Validate fields.
    body('title', 'Title must not be empty.').isLength({ min: 1 }).trim(),
    body('author', 'Author must not be empty.').isLength({ min: 1 }).trim(),
    body('summary', 'Summary must not be empty.').isLength({ min: 1 }).trim(),
    body('isbn', 'ISBN must not be empty').isLength({ min: 1 }).trim(),
  
    // Sanitize fields (using wildcard).
    sanitizeBody('*').escape(),

    // Process request after validation and sanitization.
    (req, res, next) => {
        
        // Extract the validation errors from a request.
        const errors = validationResult(req);

        // Create a Book object with escaped and trimmed data.
        var book = new Book(
          { title: req.body.title,
            author: req.body.author,
            summary: req.body.summary,
            isbn: req.body.isbn,
            genre: req.body.genre
           });

        if (!errors.isEmpty()) {
            // There are errors. Render form again with sanitized values/error messages.

            // Get all authors and genres for form.
            async.parallel({
                authors: function(callback) {
                    Author.find(callback);
                },
                genres: function(callback) {
                    Genre.find(callback);
                },
            }, function(err, results) {
                if (err) { return next(err); }

                // Mark our selected genres as checked.
                for (let i = 0; i < results.genres.length; i++) {
                    if (book.genre.indexOf(results.genres[i]._id) > -1) {
                        results.genres[i].checked='true';
                    }
                }
                res.render('book_form', { title: 'Create Book',authors:results.authors, genres:results.genres, book: book, errors: errors.array() });
            });
            return;
        }
        else {
            // Data from form is valid. Save book.
            book.save(function (err) {
                if (err) { return next(err); }
                   //successful - redirect to new book record.
                   res.redirect(book.url);
                });
        }
    }
];
```

The sanitization is carried out for all fields using a wildcard rather than listing them individually.

A book only has one author but it may have one or more genres.  Therefore the information for genre must be converted into a valid array, even if the entry for this field is empty.

The genres which are selected by the user are marked as checked.

Add the following code to the view file **views/book_form.pug**

```pug
extends layout

block content
  h1= title

  form(method='POST' action='')
    div.form-group
      label(for='title') Title:
      input#title.form-control(type='text', placeholder='Name of book' name='title' required='true' value=(undefined===book ? '' : book.title) )
    div.form-group
      label(for='author') Author:
      select#author.form-control(type='select', placeholder='Select author' name='author' required='true' )
        for author in authors
          if book
            option(value=author._id selected=(author._id.toString()===book.author._id.toString() ? 'selected' : false) ) #{author.name}
          else
            option(value=author._id) #{author.name}
    div.form-group
      label(for='summary') Summary:
      textarea#summary.form-control(type='textarea', placeholder='Summary' name='summary' required='true') #{undefined===book ? '' : book.summary}
    div.form-group
      label(for='isbn') ISBN:
      input#isbn.form-control(type='text', placeholder='ISBN13' name='isbn' value=(undefined===book ? '' : book.isbn) required='true')
    div.form-group
      label Genre:
      div
        for genre in genres
          div(style='display: inline; padding-right:10px;')
            input.checkbox-input(type='checkbox', name='genre', id=genre._id, value=genre._id, checked=genre.checked )
            label(for=genre._id) #{genre.name}
    button.btn.btn-primary(type='submit') Submit

  if errors
    ul
      for error in errors
        li!= error.msg
```

When this view is initiatied by the GET create book controler it displays the author list as a drop down select box and the genres as individual checkboxes.

The POST create book controller reads the contents and if there are no errors it redirects to the formed book url to show book details.

 > http://127.0.0.1:3000/catalog/book/create

![Create Genre](bookCreateForm.png)
 
 Submit details

![Create Genre](bookNewDetail.png)

Although a new book has been created there are no instances of it in the local_library.


## [Create BookInstance](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/forms/Create_BookInstance_form)

The pattern of working in creating a bookInstance entry is similar to genre and author create, but will need to book details from the database to supply the appropriate id fields to the JSON object before the database entry is made.

1. Use 'required' to import the functions needed to the controller file.
2. In the controller respond to a GET request by displaying the form using a pug template with a title.  
    1. Include details of all available book titles into the form using find.
3. In the controller respond to a POST request   
    1. Validate fields
    2. Sanitize fields
    3. Process the request after validation and sanitization   
        a. Extract validation errors from request  
        b. If there are errors render form again with error messages  
        c. If data is valid 
            i. create a json object with trimmed data  
            ii. save object to database
            iii. redirect to page to list of records
4.  Create a pug template which matches the scheme of the database collection targetted.

In **controllers/bookinstanceController.js** add the functions needed by requiring them.  Add a requirement for the book model.

```javascript
const BookInstance = require("../models/bookinstance");
const Book = require("../models/book");
const asyncHandler = require("express-async-handler");
const { body, validationResult } = require("express-validator");
```

In **controllers/bookinstanceController.js**, replace the book Instance create get code:

```javascript
// Display BookInstance create form on GET.
exports.bookinstance_create_get = function(req, res) {
    res.send('NOT IMPLEMENTED: BookInstance create GET');
};
```
with code to find the available book titles and pass these to the bookinstance_form view with the form title:

```javascript
// Display BookInstance create form on GET.
exports.bookinstance_create_get = asyncHandler(async (req, res, next) => {
  const allBooks = await Book.find({}, "title").sort({ title: 1 }).exec();

  res.render("bookinstance_form", {
    title: "Create BookInstance",
    book_list: allBooks,
  });
});
```

Also in **controllers/bookinstanceController.js**, replace the code for bookinstance create on POST

```javascript
// Handle BookInstance create on POST.
exports.bookinstance_create_post = function(req, res) {
    res.send('NOT IMPLEMENTED: BookInstance create POST');
};
```

with code to validate, sanitize and process the request.

```javascript
// Handle BookInstance create on POST.
exports.bookinstance_create_post = [
  // Validate and sanitize fields.
  body("book", "Book must be specified").trim().isLength({ min: 1 }).escape(),
  body("imprint", "Imprint must be specified")
    .trim()
    .isLength({ min: 1 })
    .escape(),
  body("status").escape(),
  body("due_back", "Invalid date")
    .optional({ values: "falsy" })
    .isISO8601()
    .toDate(),

  // Process request after validation and sanitization.
  asyncHandler(async (req, res, next) => {
    // Extract the validation errors from a request.
    const errors = validationResult(req);

    // Create a BookInstance object with escaped and trimmed data.
    const bookInstance = new BookInstance({
      book: req.body.book,
      imprint: req.body.imprint,
      status: req.body.status,
      due_back: req.body.due_back,
    });

    if (!errors.isEmpty()) {
      // There are errors.
      // Render form again with sanitized values and error messages.
      const allBooks = await Book.find({}, "title").sort({ title: 1 }).exec();

      res.render("bookinstance_form", {
        title: "Create BookInstance",
        book_list: allBooks,
        selected_book: bookInstance.book._id,
        errors: errors.array(),
        bookinstance: bookInstance,
      });
      return;
    } else {
      // Data from form is valid
      await bookInstance.save();
      res.redirect(bookInstance.url);
    }
  }),
];
```
If the form contains a valid entry the bookinstance is passed to the database and the user is returned to the bookinstance url.

Create the form view in **views/bookinstance_form.pug**

```pug
extends layout

block content
  h1=title

  form(method='POST' action='')
    div.form-group
      label(for='book') Book:
      select#book.form-control(type='select' placeholder='Select book' name='book' required='true')
        for book in book_list
          option(value=book._id, selected=(selected_book==book._id.toString() ? '' : false) ) #{book.title}

    div.form-group
      label(for='imprint') Imprint:
      input#imprint.form-control(type='text' placeholder='Publisher and date information' name='imprint' required='true' value=(undefined===bookinstance ? '' : bookinstance.imprint))
    div.form-group
      label(for='due_back') Date when book available:
      input#due_back.form-control(type='date' name='due_back' value=(undefined===bookinstance ? '' : bookinstance.due_back_yyyy_mm_dd))

    div.form-group
      label(for='status') Status:
      select#status.form-control(type='select' placeholder='Select status' name='status' required='true' )
        option(value='Maintenance' selected=(undefined===bookinstance || bookinstance.status!='Maintenance' ? false:'selected')) Maintenance
        option(value='Available' selected=(undefined===bookinstance || bookinstance.status!='Available' ? false:'selected')) Available
        option(value='Loaned' selected=(undefined===bookinstance || bookinstance.status!='Loaned' ? false:'selected')) Loaned
        option(value='Reserved' selected=(undefined===bookinstance || bookinstance.status!='Reserved' ? false:'selected')) Reserved

    button.btn.btn-primary(type='submit') Submit

  if errors
    ul
      for error in errors
        li!= error.msg
```
Note that the Status Select drop down menu options are hard coded.  This is ok if there are to be a small number of fixed status available, however the example could be developed by adding a collection of bookStates.


 > http://127.0.0.1:3000/catalog/bookinstance/create

![Create Genre](bookinstanceCreateForm.png)
 
 Submit details

![Create Genre](bookinstanceNewDetail.png)

A small detail to note is that an apostrophe is being listed as \&#x27; which could corrected.

That concludes the form entries for creating new content.  Next step is to build forms for Deletion.

## [Delete Author](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/forms/Delete_author_form)

1. At the foot of each author detail page add a link to delete the author which will call author.url+'/delete' with a GET request.
2. In the controller respond to a GET request by displaying the form using a pug template with a title.  
    1. If the author has assosciated books display these in a pug view.  Books must be deleted before author is removed.
    2. If there are no books associated with the author display details and request confirmation of deletion as a POST request.
3. In the controller respond to a POST request   
    1. Request author details and book details async
    2. Process the request 
        a. Recheck and if author has associated books do not allow deletion but return to the author delete form. 
        b. If author has no associated books, delete and redirect to catalog/authors list.
4.  Create a pug template for the authors delete form.

Add a link to delete code to the foot of **views/author_detail.pug**

```pug
extends layout

block content

  h1 Author: #{author.name}
  p #{author.date_of_birth_formatted} - #{author.date_of_death_formatted}
  
  div(style='margin-left:20px;margin-top:20px')

    h4 Books
    
    dl
      each book in author_books
        dt 
          a(href=book.url) #{book.title}
        dd #{book.summary}

      else
        p This author has no books.
  hr
  p
    a(href=author.url+'/delete') Delete author             
```
Within **controllers/AuthorController.js** replace Display Author delete form on GET:

```javascript
// Display Author delete form on GET.
exports.author_delete_get = function(req, res) {
    res.send('NOT IMPLEMENTED: Author delete GET');
};
```

with code:

```javascript
// Display Author delete form on GET.
exports.author_delete_get = asyncHandler(async (req, res, next) => {
  // Get details of author and all their books (in parallel)
  const [author, allBooksByAuthor] = await Promise.all([
    Author.findById(req.params.id).exec(),
    Book.find({ author: req.params.id }, "title summary").exec(),
  ]);

  if (author === null) {
    // No results.
    res.redirect("/catalog/authors");
  }

  res.render("author_delete", {
    title: "Delete Author",
    author: author,
    author_books: allBooksByAuthor,
  });
});
```

Now ammend the entry in **controllers/authorController.js** for the Author delete on POST from:

```javascript
// Handle Author delete on POST.
exports.author_delete_post = asyncHandler(async (req, res, next) => {
  // Get details of author and all their books (in parallel)
  const [author, allBooksByAuthor] = await Promise.all([
    Author.findById(req.params.id).exec(),
    Book.find({ author: req.params.id }, "title summary").exec(),
  ]);

  if (allBooksByAuthor.length > 0) {
    // Author has books. Render in same way as for GET route.
    res.render("author_delete", {
      title: "Delete Author",
      author: author,
      author_books: allBooksByAuthor,
    });
    return;
  } else {
    // Author has no books. Delete object and redirect to the list of authors.
    await Author.findByIdAndDelete(req.body.authorid);
    res.redirect("/catalog/authors");
  }
});
```

Create **views/author_delete.pug** to show the name if an author is found.

```pug
extends layout

block content
  h1 #{title}: #{author.name}
  p= author.lifespan

  if author_books.length

    p #[strong Delete the following books before attempting to delete this author.]

    div(style='margin-left:20px;margin-top:20px')

      h4 Books

      dl
      each book in author_books
        dt
          a(href=book.url) #{book.title}
        dd #{book.summary}

  else
    p Do you really want to delete this Author?

    form(method='POST' action='')
      div.form-group
        input#authorid.form-control(type='hidden',name='authorid', required='true', value=author._id )

      button.btn.btn-primary(type='submit') Delete

```

Add a delete link to **author_detail.pug

```javascript
extends layout

block content

  h1 Author: #{author.name}
  p #{author.date_of_birth_formatted} - #{author.date_of_death_formatted}
  

  div(style='margin-left:20px;margin-top:20px')

    h4 Books

    dl
      each book in author_books
        dt
          a(href=book.url) #{book.title}
        dd #{book.summary}

      else
        p This author has no books.
        
  hr
  p
    a(href=author.url+'/delete') Delete author
```

To test this, create a test author entry and then delete it.

> http://127.0.0.1:3000/catalog/author/create

![Create Author](authorCreateDoe.png)
 
Submit

![Author Detail](authorDetailDoe.png)

Author Delete

![Author Delete](authorDeleteDoe.png)

Note formatted display of author dates.

Delete Author.

![Deleted Author](authorDeletedDoe.png)

## Exercise
It is left as an exercise to develop Delete forms for  for the Book, BookInstance, and Genre models.

## [Update Book Form](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/forms/Update_Book_form)

The create, retrieve and delete processes have been implimented which just leaves update.  

Updating is like creating except that the form must be filled in from the current database entry.



1. In the controller respond to a GET request by displaying the form using a pug template with a title.  
    1. Include details of all available book titles into the form using find.
2. In the controller respond to a POST request   
    1. Validate fields
    2. Sanitize fields
    3. Process the request after validation and sanitization   
        a. Extract validation errors from request  
        b. If there are errors render form again with error messages  
        c. If data is valid 
            i. create a json object with trimmed data  
            ii. save object to database
            iii. redirect to page to list of records
3. create views for forms            


In **controllers/bookController.js**, replace:

```javascript
// Display book update form on GET.
exports.book_update_get = function(req, res) {
    res.send('NOT IMPLEMENTED: Book update GET');
};
```

with code:

```javascript
// Display book update form on GET.
exports.book_update_get = asyncHandler(async (req, res, next) => {
  // Get book, authors and genres for form.
  const [book, allAuthors, allGenres] = await Promise.all([
    Book.findById(req.params.id).populate("author").populate("genre").exec(),
    Author.find().sort({ family_name: 1 }).exec(),
    Genre.find().sort({ name: 1 }).exec(),
  ]);

  if (book === null) {
    // No results.
    const err = new Error("Book not found");
    err.status = 404;
    return next(err);
  }

  // Mark our selected genres as checked.
  for (const genre of allGenres) {
    for (const book_g of book.genre) {
      if (genre._id.toString() === book_g._id.toString()) {
        genre.checked = "true";
      }
    }
  }

  res.render("book_form", {
    title: "Update Book",
    authors: allAuthors,
    genres: allGenres,
    book: book,
  });
});
```

This is called by a GET request passing the id of a particular book as a parameter.  The result, if succesful is data which is passed to the Update Book form.

The completed form will submit to update book on POST. Also in **controllers/bookController.js**, replace :

```javascript
// Handle book update on POST.
exports.book_update_post = function(req, res) {
    res.send('NOT IMPLEMENTED: Book update POST');
};
```

with code:

```javascript
// Handle book update on POST.
exports.book_update_post = [
  // Convert the genre to an array.
  (req, res, next) => {
    if (!Array.isArray(req.body.genre)) {
      req.body.genre =
        typeof req.body.genre === "undefined" ? [] : [req.body.genre];
    }
    next();
  },

  // Validate and sanitize fields.
  body("title", "Title must not be empty.")
    .trim()
    .isLength({ min: 1 })
    .escape(),
  body("author", "Author must not be empty.")
    .trim()
    .isLength({ min: 1 })
    .escape(),
  body("summary", "Summary must not be empty.")
    .trim()
    .isLength({ min: 1 })
    .escape(),
  body("isbn", "ISBN must not be empty").trim().isLength({ min: 1 }).escape(),
  body("genre.*").escape(),

  // Process request after validation and sanitization.
  asyncHandler(async (req, res, next) => {
    // Extract the validation errors from a request.
    const errors = validationResult(req);

    // Create a Book object with escaped/trimmed data and old id.
    const book = new Book({
      title: req.body.title,
      author: req.body.author,
      summary: req.body.summary,
      isbn: req.body.isbn,
      genre: typeof req.body.genre === "undefined" ? [] : req.body.genre,
      _id: req.params.id, // This is required, or a new ID will be assigned!
    });

    if (!errors.isEmpty()) {
      // There are errors. Render form again with sanitized values/error messages.

      // Get all authors and genres for form
      const [allAuthors, allGenres] = await Promise.all([
        Author.find().sort({ family_name: 1 }).exec(),
        Genre.find().sort({ name: 1 }).exec(),
      ]);

      // Mark our selected genres as checked.
      for (const genre of allGenres) {
        if (book.genre.indexOf(genre._id) > -1) {
          genre.checked = "true";
        }
      }
      res.render("book_form", {
        title: "Update Book",
        authors: allAuthors,
        genres: allGenres,
        book: book,
        errors: errors.array(),
      });
      return;
    } else {
      // Data from form is valid. Update the record.
      const updatedBook = await Book.findByIdAndUpdate(req.params.id, book, {});
      // Redirect to book detail page.
      res.redirect(updatedBook.url);
    }
  }),
];
```



Valid data is passed into the database and the book detail is displayed.

Modify the file **views/book_form.pug** so that it can deal with both create and update:

```javascript
 extends layout

block content
  h1 Title: #{book.title}

  p #[strong Author:]
    a(href=book.author.url) #{book.author.name}
  p #[strong Summary:] #{book.summary}
  p #[strong ISBN:] #{book.isbn}
  p #[strong Genre:]
    each val, index in book.genre
      a(href=val.url) #{val.name}
      if index < book.genre.length - 1
        |,

  div(style='margin-left:20px;margin-top:20px')
    h4 Copies

    each val in book_instances
      hr
      if val.status=='Available'
        p.text-success #{val.status}
      else if val.status=='Maintenance'
        p.text-danger #{val.status}
      else
        p.text-warning #{val.status}
      p #[strong Imprint:] #{val.imprint}
      if val.status!='Available'
        p #[strong Due back:] #{val.due_back}
      p #[strong Id:]
        a(href=val.url) #{val._id}

    else
      p There are no copies of this book in the library.

  hr
  p
    a(href=book.url+'/delete') Delete Book
  p
    a(href=book.url+'/update') Update Book

```


Before updating a book with a new author, the author must be created, and if needed, a new genre.

So add the author details for Charles Reade, born 08 June 1814, died 11 April 1884

![create author](createAuthorReade.png)

Now try to update a book.  Show the book list:

> http://127.0.0.1:3000/catalog/books

![Book List](bookListForUpdate.png)

Then navigate to Test book one:

![Test Book One](testBookOne.png)

Follow the update Book link and add new details:

The book is not really fantasy, but lets use that genre for now!.

![Create Book](bookUpdateCloister.png)

The updated detail is displayed:

Submit.

![Details Cloister](bookDetailsCloister.png)

We may as well create an instance of the book!

![Create Instance](createBookInstanceCloister.png)


Submit.

![Cloister instance](newBookInstanceCloister.png)





## Exercise
It is left as an exercise to complete other delete and update pages.